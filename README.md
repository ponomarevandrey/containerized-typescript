# Dockerized TypeScript app

The project demonstrates how to set up and debug TypeScript app inside Docker container with VS Code. 

We use:
* Docker (Docker Compose)
* Node.js (Express.js) app
* TypeScript Compiler
* VS Code
* `ts-node` and `nodemon` packages 



## Troubleshooting

* **If you encounter ant problems with TypeScript types** (e.g. Express is `any`), here are all workarounds you can try: https://github.com/DefinitelyTyped/DefinitelyTyped/issues/53397

  Currently I resolved the typing problem with Yarn: use Yarn instead of NPM:
  
  1. delete `node_modules` and all lock files 
  2. reinstall them by issuing `yarn install`
  3. copy-paste the dependencies below, I've tested their versions, they're compatible
  4. replace `RUN npm install` with `RUN yarn install` in `Dockerfile`

  ```json
  {
    "name": "typescript-dockerized-project",
    "version": "1.0.0",
    "description": "",
    "main": "src/index.js",
    "scripts": {
      "start": "nodemon",
      "test": "echo \"Error: no test specified\" && exit 1"
    },
    "keywords": [],
    "author": "",
    "license": "ISC",
    "devDependencies": {
      "@types/express": "^4.17.2",
      "@types/express-serve-static-core": "^4.17.21",
      "@types/node": "^20.10.5",
      "nodemon": "^3.0.2",
      "ts-node": "^10.9.2",
      "typescript": "^5.3.3"
    },
    "dependencies": {
      "express": "^4.17.2",
      "express-serve-static-core": "^0.1.1"
    },
    "resolutions": {
      "@types/express-serve-static-core": "4.17.21",
      "@types/express": "4.17.2"
    }
  }
  ```

* **If you experience an error concerned with types on Request object of Express, do this:** https://github.com/DefinitelyTyped/DefinitelyTyped/discussions/62263 . 

  In short, just issue this command: `npm i --save-dev @types/express-serve-static-core@4.17.28` - this fixes types, cause the latest version of `express-serve-static-core` has breaking changes or wrong types, I don't know, but something is wrong with it.


## TypeScript Compiler configuration

You can fine-tune configuration through options in `tsconfig.json` and `start` script in `package.json`.

**`package.json`:**
```json
...
    "start": "nodemon --watch ./src/**/* -e ts,json --exec TS_NODE_PROJECT=tsconfig.json node --inspect=0.0.0.0:9229 -r ts-node/register ./src/index.ts"
...
```

* `--watch ./src/**/*` tells `nodemon` to watch for only `src` folder

* `--e ts,json` tells `nodemon` to watch only for the files with these extensions

* `--exec TS_NODE_PROJECT=tsconfig.json node --inspect=0.0.0.0:9229 -r ts-node/register ./src/index.ts` tells `nodemon` to start `node` and pass it `TS_NODE_PROJECT` environment variable which specifies the location and name of TypeScript Compiler config file (this variable is used by TypeScript Compiler). `-r` flag is a short command-line variant of `require()`, it preloads  the specified module at startup, in our case we preload `ts-node/register`. Finally we pass to `node` process the name of the main app's file - `index.ts`.

* If you need to debug the script from the very first line, change `... node --inspect=0.0.0.0:9229 ...` to `... node --inspect-brk=0.0.0.0:9229 ...`

Note the `volume` option in `dcoker-compose.yml`: we mount `tsconfig.json` (to tell TSC *how* to compile `.ts` files inside container) and `src` folder (containing all source `.ts`-files )



## VS Code Debugger configuration

`.vscode/launch.json` (pay attention to `sourceMapPathOverrides` option and make sure you provided the correct value, otherwise the debugger won't stop on breakpoints)
