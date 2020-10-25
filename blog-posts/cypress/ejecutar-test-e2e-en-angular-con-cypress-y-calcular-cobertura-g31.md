---
published: false
title: 'Ejecutar test e2e en Angular con Cypress y calcular cobertura'
cover_image:
description:
tags: cypress
series: cypress
canonical_url:
---

# Resumen

En este tutorial vamos a ver cómo ejecutar los tests e2e de nuestras aplicaciones Angular con Cypress en vez de Protractor y cómo configurar el proyecto para obtener la cobertura de los tests de Cypress.

# Entorno

- Slimbook Prox15 32 Gb RAM i7
- SlimbookOS (Ubuntu 20.04)
- Angular 9
- Cypress 4.5.0
- Visual Studio Code

# Introducción

A poco que hayas trabajado o estés trabajando con Angular te habrás dado cuenta que por defecto viene configurado para ejecutar los tests con Protractor, una tecnología que adolece de los problemas de inestabilidad y falsos positivos de Selenium y que por ello ahora se prefiere la tecnología de Cypress para este tipo de tests.

Además otro punto interesante es registrar a modo de informe de cobertura que tanto por ciento de mi código ha sido ejecutado con estos tests... pues todo esto lo vamos a ver en este turorial.

# Vamos al lío

Partimos de que hemos creado o tenemos ya creado un proyecto con Angular CLI, si lo creamos de cero, solo tenemos que ejecutar:

```bash
$> ng new poc
```

Como ya sabrás esto hace la magía de crearnos un proyecto Angular completamente configurado y listo para entrar en producción.

Pero como ya hemos dicho ahora queremos utilizar Cypress en vez de Protractor para lo cual, y gracias a los schematics de Angular, simplemente y sin preguntar mucho tenemos que ejecutar:

```bash
$> ng add @briebug/cypress-schematic
```

Este comando hace su magia y borra todo lo relacionado con Protractor, añadiendo toda la configuración necesaria para trabajar con Cypress, además respetando TypeScript, ¡qué más podemos pedir!

Ahora cuando ejecutamos los tests e2e con el comando:

```bash
$> npm run e2e
```

Lo que nos aparece es la consola de Cypress, lista para ayudarnos a crear nuestros primeros tests.

Si queremos ejecutar los tests en modo CI, solo tenemos que ejecutar:

```bash
$> npm run poc:cypress-run
```

Ahora vamos a por la segunda parte del tutorial que consiste en registar el cálculo de cobertura de los tests de Cypress. Desafortunadamente, todavía no contamos con un schematics para hacerlo, por lo que vamos a verlo paso a paso:

En primer lugar creamos el fichero cypress/coverage.webpack.js con el siguiente contenido:

```js
module.exports = {
    module: {
      rules: [
        {
          test: /\.(js|ts)$/,
          loader: 'istanbul-instrumenter-loader',
          options: { esModules: true },
          enforce: 'post',
          include: require('path').join(__dirname, '..', 'src'),
          exclude: [
            /\.(e2e|spec)\.ts$/,
            /node_modules/,
            /(ngfactory|ngstyle)\.js/
          ]
        }
      ]
    }
  };
```

Luego comprobamos si ya existe en nuestro proyecto el fichero cypress/plugins/cy-ts-preprocessor.ts, si no existe lo creamos con el siguiente contenido:

```js
const webpackPreprocessor = require("@cypress/webpack-preprocessor");

const webpackOptions = {
  resolve: {
    extensions: [".ts", ".js"]
  },
  module: {
    rules: [
      {
        test: /\.ts$/,
        exclude: [/node_modules/],
        use: [
          {
            loader: "ts-loader"
          }
        ]
      }
    ]
  }
};

const options = {
  webpackOptions
};

module.exports = webpackPreprocessor(options);

```

Ahora añadimos un nuevo builder a nuestro proyecto, ejecutando:

```bash
$> npm i -D ngx-build-plus
``` 

Y dentro del fichero angular.json buscamos la propiedad "serve" y la configuramos de esta forma:

```js
"serve": {
    "builder": "ngx-build-plus:dev-server",
    "options": {
        "browserTarget": "poc:build",
        "extraWebpackConfig": "./cypress/coverage.webpack.js"
    },
    "configurations": {
        "production": {
            "browserTarget": "poc:build:production"
        },
        "ci": {
            "progress": false
        }
    }
},
```

> Nota: fíjate que "poc" es el nombre del proyecto, tu seguro que tendrás otro.

Hecho esto vamos a añadir la depedencia de istanbul que nos va a ayudar a sacar el reporte de cobertura.

```bash
$> npm i -D istanbul-instrumenter-loader
```

Y añadimos las depedencias que hacer que istanbul entienda TypeScript.

```bash
$> npm i -D @istanbuljs/nyc-config-typescript source-map-support ts-node
```

Tenemos que editar el fichero angular.json para añadir la configuración de nyc que deja de ser el CLI de Istanbul.

```js
...
"nyc": {
    "extends": "@istanbuljs/nyc-config-typescript",
    "all": true
  }
...
```

Ahora instalamos el plugin de cobertura de Cypress y lo necesario de nyc:

```bash
$> npm install -D @cypress/code-coverage nyc istanbul-lib-coverage
```

Casi por último, editamos el fichero cypress/support/index.ts con el siguiente contenido:

```js
import './commands';

// Import cypress code-coverage collector plugin
import '@cypress/code-coverage/support';
```

> Nota: no importa si el formateador te cambia de orden los imports.

y creamos/editamos el fichero cypress/plugins/index.js, dejándolo con el siguiente contenido:

```js
const cypressTypeScriptPreprocessor = require("./cy-ts-preprocessor");
const registerCodeCoverageTasks = require("@cypress/code-coverage/task");

module.exports = (on, config) => {
  on("file:preprocessor", cypressTypeScriptPreprocessor);

  registerCodeCoverageTasks(on, config);

  return config;

};

```

Ya hemos terminado pero para facilitar la ejecución de los tests en modo CI, vamos a editar el fichero package.json y añadir el siguiente script de npm:

```js
...
"e2e:run": "ng run poc:cypress-run",
...
```

De esta forma cuando ejecutamos:

```bash
$> npm run e2e:run
```

Veremos que Cypress se ejecuta en modo CI y después de la ejecución dejará el reporte de cobertura calculado en dentro de la carpeta "coverage" del proyecto.

De esta forma podemos utilizar estos informes para integrarlos con Sonarqube o cualquier otra herramienta de calidad y lo más importante podemos ver que si hay código que no llega a ejecutarse y puede ser eliminado para favorecer la mantenibilidad del proyecto.