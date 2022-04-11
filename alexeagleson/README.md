# How to Create a React Typescript Monorepo with Git Submodules.
[See Original Blog](https://dev.to/alexeagleson/how-to-create-a-node-and-react-monorepo-with-git-submodules-2g83)

### We will be using a tool called Learner to manage the dependencies so projects can share files with similar versions

Leveraging on Yarn workspaces, we'll use Project Structure required for [lerna](https://github.com/lerna/lerna).

- Create a Node Server [backend]

    `mkdir -p packages/simple-node-server`

    `cd packages/simple-node-server`

    `yarn init`

    `touch server.ts`

    ``server.ts

        import express from 'express';
        const app = express();
        const port = 3001;

        // this should be available in the mono-repo to be used by other applications accessing it
        export interface QueryPayload{
            "foo": string;
        }

        app.get("/data", (req, res) =>{
            res.setHeader('Access-Control-Allow-Origin', '*'); //cors

            const data:QueryPayload = {foo:"bar"};
            res.json(data);
        });

        app.listen(port, ()=>{
            console.log(`Example app listening at http://localhost:${port}`);
        });
    ``

    `yarn add -D typescript @types/express`
    `yarn add express`
    `npx tsc --init`



- Create a react app that reads from node server [frontend]
    
        cd ../packages
        yarn create react-app simple-react-app --template typescript
        cd simple-react-app

    ### you'll notice the react typescript version and the node-server typescript versions are different.
    
    remove typescript from react-app and add as devDependency 

        yarn remove typescript
        yarn add -D typescript

    

    ## Hence the need for a monorepo to help us solve this problem
    - Configure Monorepo @ ../typescript-mono-repo


            cd ../typescript-mono-repo
            yarn init

        Install [lerna](https://github.com/lerna/lerna) tool inside the mono-repo root directory

            yarn add -D lerna
            npx lerna init
            
        lerna.json file should be created which is configured to look for all folders within the package directory.

        Configure lerna.json to use yarn and workspaces. Update lerna.json to below

            {
                "packages": [
                    "packages/*"
                ],
                "version": "0.0.0",
                "npmClient": "yarn",
                "useWorkspaces": true             
            }

        Update ./typescript-mono-repo/alexeagleson/package.json to below (reflect workspaces)

            {
                "private": true,
                "name": "typescript-mono-repo",
                "version": "1.0.0",
                "main": "index.js",
                "license": "MIT",
                "workspaces": [
                    "packages/*"
                ],
                "devDependencies": {
                    "lerna": "^4.0.0"
                }
            }

        Run yarn install

            yarn install

        ### you'll notice, the typescript installation in the node-server-project and simple-react-app will have been moved up to the root node_modules. These will have symlink files if anything.


    ## Startup node server

        cd ../typescript-mono-repo/alexeagleson/simple-node-server

    > edit tsconfig.json, uncomment outDir to value for transpiled js files

        "outDir": "./dist"

    > edit package.json, add scripts

        "scripts": {
            "start": "tsc && node dist/server.js
            +"
        },
    
    > start node server

        yarn start

    ## Let us ensure our back-end can communicate with our front-end apps.

        cd ../typescript-mono-repo/alexeagleson/packages/simple-react-app
        yarn start

    ### Update App.tsx in react-app

        import React from 'react';
        import logo from './logo.svg';
        import './App.css';

        import {QueryPayload} from 'simple-node-server/server';

        const App = ()=>{

            <button title="Get Data" onClick={() =>{
                fetch("http://localhost:3001/data",{})
                .then((response)=>response.json())
                .then((data: QueryPayload) => console.log(data.foo));

            }}/>
        }
    
    ## Do stuff in typescript-mono-repo root directory the power of Lerna

        cd typescript-mono-repo
        #add new package to all projects at the sametime
        npx lerna add lodash
        #check node_modules for all projects


    ## Git Submodules: to allow us to add existing repo to a project and still maintain it as a separate repo

        cd ../typescript-mono-repo
        git init
        touch .gitignore
        echo "node_modules" >> .gitignore
        echo "dist" >> .gitignore

    > We want to add a [git repo react-dark-mode](https://github.com/alexeagleson/react-dark-mode) as a third module to the mono-repo
      
        cd ../typescript-mono-repo/alexeagleson/packages/simple-react-app/src
        git submodule add git@github.com:alexeagleson/react-dark-mode.git
        

- Create a project called shared data which is basically data both front end and back end uses.
