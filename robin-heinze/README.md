[Watch](https://www.youtube.com/watch?v=2wOvhDtqsW8)

# Web and mobile apps using React Native Web with [Robin Heinze](https://github.com/robinheinze)

- Create Monorepo Directory

        mkdir mono-repo
        cd mono-repo

- Create react native app

        npx react-native init MobileDemo --template react-native-template-typescrpt

        cd MobileDemo

        yarn ios && yarn android

- Create web app in root of mono-repo

        cd ..

        cd mono-repo
        
        mkdir web-demo && cd  web-demo

        touch index.html index.tsx tsconfig.json # see alexeagleson

        yarn init #sets up as npm package

        yarn add -D parcel-bundler typescript

        yarn add react react-dom

    - >update index.html
        
            <html>
                <body>
                    <div id="root"></div>
                    <script src="./index.tsx"/>
                </body>
            </html>
        
   
    - >update index.tsx

            import React from 'react';
            import {render} from 'react-native';
            
            const App = () => {
                return (
                    <div>
                        <p>Hello world!</p>
                    </div>
                );
            };

            render(<App/>, document.getElementById("root"));

    - >update tsconfig.json

            {
                "compilerOptions": {
                    "jsx": "react",
                    "lib": ["es6","dom"],
                    "target": "esnext",
                    "allowSyntheticDefaultImports": true,
                    "esModuleInterop": true,
                    "moduleResolution": "node",
                    "noEmit": true,
                    "skipLibCheck": true
                },
                "exclude": ["./node_modules"]
            }
    
    - >Update package.json

            {
                "name": "web-demo",
                "version": "1.0.0",
                "main": "index.js",
                "license": "MIT",
                "devDependencies": {
                    "parcel-bundler": "^1.12.5",
                    "typescript": "^4.3.5"
                },
                "dependencies": {
                    "react": "^17.0.2",
                    "react-dom": "^17.0.2"
                },
                "scripts": {
                    "start": "parcel serve ./index.html",
                    "build": "parcel build ./index.html"
                }
            }

    
    
    - >Run
    
            yarn start
            http://localhost:1234

    - > React Native and React-Web apps should be running

- >Setup Shared Folder within react-native app *MobileApp*: Why is the shared folder in the mobile app and not in some stand alone directory? React-native uses Metro as a bundler and it's preference is to put  stuff inside the app as it's not a fan of symlinks. Thus link stuff out from here to somewhere else. 

    - >create subfolder app/shared

        cd ../mono-repo/MobileDemo
        
        mkdir -p app/shared && cd app && cd shared

        mkdir components && cd components

    - Create symlink from MobileDemo/app/shared to [WebApp]/src/shared

        cd ../mono-repo/web-demo

        mkdir src

        cd src

        ln -s ~/.../mono-repo/MobileDemo/app/shared shared

        #reload vscode, shared directory should appear in src folder of web app

- >Back to Web-Demo

        cd ../mono-repo/web-demo

        yarn add react-native-web @types/react-native
    
    - >edit web-demo package.json, add alias

        {
            ...
            +"alias": {
                "react-native": "react-native-web"
            }
        }
    
- >Back to MobileDemo; add shared files/assets

        cd ../mono-repo/MobileDemo/app/shared/components

        touch counter.tsx

    - >edit counter.tsx file

            import React from 'react';
            import {Pressable, StyleSheet, Text, View, ViewStyle} from 'react-native';

            const centered:ViewStyle = {
                justifyContent: 'center',
                alignItems: 'center'
            };

            const SIZE = 100;
            const FONT_SIZE = SIZE/2;

            const styles = StyleSheet.create({
                container: {
                    ...centered,
                    flex: 1,
                    flexDirection: 'row',
                    backgroundColor: 'red'
                },
                counter: {
                    ...centered,
                    height: SIZE,
                    width: SIZE,
                    borderRadius: 5,
                },
                numberText: {
                    color: 'green',
                    fontWeight: 'bold',
                    fontSize; FONT_SIZE,
                },
                buttonText: {
                    color: 'white',

                }
            });

            export const Counter = () => {
                const [count, setCount] = React.useState(0);
                const increment = () => {
                    setCount(count + 1);
                };
                const decrement = () => {
                    count > 0 && setCount( count - 1);
                };
                return (
                    <View style={styles.container}>
                        <Pressable style={styles.button} onPress={increment}>
                            <Text>+</Text>
                        </Pressable>
                        <View>
                            <Text>{count}</Text>
                        </View>
                        <Pressable style={styles.button} onPress={decrement}>
                            <Text>-</Text>
                        </Pressable>
                    </View>
                );
            };

    


    

        




- >Back to web-demo; add Counter component to index.tsx

    const SIZE = Platform.select({web: 200, default: 100});
    const App = () => {
        return (
            <View>
                <Counter />
            </View>
        );
    }