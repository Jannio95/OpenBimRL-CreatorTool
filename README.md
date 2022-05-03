# OpenBimRL Creator-Tool

A web-based application to create and edit OpenBimRL files. 

## Getting started

To run this project, following applications must be installed:

- node.js
- npm
- Visual Studio Code (Recommended) + Vetur plugin for Vue


The following dependencies have to be installed using npm:


```shell
npm install express
npm install cors
npm install laravel-mix
npm install vue
```

For more details on dependencies view the package.json file.

# Compiling and Running

New client side implementations can be added by writing code in: /webapp/src/
New server side implementations can be added by writing code in: express-server-run.js

In order to bundle Vue components into a single app.js file, the code has to be compiled using laravel-mix and watch. The files will be stored in */webapp/dist/*. These files are used by the index.html file. To generate the compiled files, simply run following command:

```shell
npm run watch
```

**NOTE:** Executing watch will install additional packages. You may need to run it again afterwards. If errors accure run *npm install* to fix them. The watch-mode supports hot-reload, in order to edit and view files live in a browser. 

Additionally, the server can be used to execute the environment and serving the webapp. Simply run:

```shell
node express-server-run.js
```

Exiting a running server or watch-mode: **STRG + C**