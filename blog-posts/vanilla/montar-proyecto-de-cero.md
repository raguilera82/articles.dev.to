# Montar proyecto de cero con VanillaJS

## Introducción

En un mundo frontend dominado actualmente por los tres frameworks: React, Angular y Vue; es importante no perder las bases y ver que nada es magia y que todo se puede hacer con VanillaJS.

En este primer tutorial vamos a ver cómo montar el entorno necesario para comenzar nuestro proyecto sin frameworks, utilizando únicamente alguna librería.

## Creación del proyecto y primeros ficheros

Partimos de que nuestra máquina de desarrollo cuenta con un runtime de NodeJS y vamos a utilizar la terminal y como editor el Visual Studio Code.

Así que lo primero será abrir la terminal y crear el directorio donde vamos a empezar nuestro proyecto de cero.

```bash
$> mkdir /path/project
$> cd /path/project
```

Ahora para inicializar el proyecto hacemos uso del comando init de npm, y con el modificador -y aceptamos todos los valores por defecto:

```bash
$> npm init -y
```

Para ayudar en la edición de los ficheros vamos a abrir el proyecto con Visual Studio Code y el resto de comandos los ejecutaremos desde el terminal del editor:

```bash
$> code .
```

Nos apoyamos en el editor para crear una carpeta llamada "src" que va a contener inicialmente los ficheros "index.html", "main.css" y "main.js".

El fichero index.html va a tener el siguiente contenido donde no hacemos ninguna referencia al CSS ni al JavaScript, esa será labor de webpack.

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Vanilla</title>
  </head>
  <body>
    <h1>Vanilla</h1>
  </body>
</html>
```

En el fichero "main.css" podemos añadir algún estilo:

```css
body {
  background-color: grey;
}
```

E igual podemos hacer con el fichero "main.js" para ver que vamos bien, si vemos en la consola el mensaje y donde importamos el fichero main.css que luego será reconocido por webpack:

```javascript
import './main.css';
console.log('Vanilla');
```

Directamente podemos abrir el fichero "index.html" en un navegador y ver que efectivamente se aplica la regla de CSS y se muestra el mensaje en consola.

## Añadimos webpack al proyecto

A fin de que nuestro proyecto sea una SPA y para realizar todas las operaciones necesarias para su puesta en producción como minificado, refresco automático y otras tareas, vamos a incluir webpack como dependencia de desarrollo del proyecto.

```bash
$> npm install webpack webpack-cli --save-dev
$> npm install webpack-cli --save-dev
$> npm install webpack-dev-server --save-dev
$> npm install webpack-merge --save-dev
```

Instalamos también todos los plugins de webpack que inicialmente vamos a utilizar en nuestro proyecto:

```bash
$> npm install --save-dev html-webpack-plugin
$> npm install --save-dev mini-css-extract-plugin
$> npm install --save-dev css-loader
$> npm install --save-dev style-loader
$> npm install --save-dev html-loader
```

Ahora en el raíz del proyecto vamos a crear un fichero llamado "webpack.common.js" donde vamos a incluir todos los comportamientos que queremos que webpack nos ejecute independiente del entorno sea de desarrollo o de producción.

```javascript
const path = require('path');
const HTMLWebpackPlugin = require('html-webpack-plugin');

module.exports = {
  entry: './src/main.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'bundle.js',
    clean: true
  },
  module: {
    rules: [
      {
        test: /\.css$/i,
        use: ['style-loader', 'css-loader']
      },
      {
        test: /\.html$/i,
        loader: 'html-loader' 
      }
    ]
  },
  plugins: [
    new HTMLWebpackPlugin({
      template: './src/index.html'
    })
  ]
};
```

Y luego vamos a crear uno para el entorno de desarrollo llamado webpack.dev.js

```javascript
const { merge } = require('webpack-merge');

const common = require('./webpack.common.js');

module.exports = merge(common, {
  mode: 'development',
  devtool: 'inline-source-map',
  target: 'web',
  devServer: {
    contentBase: './dist'
  }
});
```

Y otro con la configuración para producción:

```javascript
const { merge } = require('webpack-merge');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');

const common = require('./webpack.common.js');

module.exports = merge(common, {
  mode: 'production',
  devtool: 'source-map',
  plugins: [new MiniCssExtractPlugin()],
  module: {
    rules: [
      {
        test: /\.css$/i,
        use: [MiniCssExtractPlugin.loader, 'css-loader']
      }
    ]
  }
});
```

Añadimos los siguientes scripts al fichero package.json:

```javascript
"scripts": {
    "start": "webpack serve --config=webpack.dev.js",
    "build": "webpack --config=webpack.prod.js"
},
```

De forma que cuando estamos desarrollando utilizamos la configuración de desarrollo y cuando construimos el bundle lo hacemos con la configuración para producción.

## Configuración de pruebas automáticas

A fin de poder ejecutar pruebas automáticas en el proyecto, añadimos la dependencia de Jest:

```bash
$> npm install --save-dev jest
```

y el siguiente script al fichero package.json, quedando de esta forma: 

```javascript
"scripts": {
    "start": "webpack serve --config=webpack.dev.js",
    "build": "webpack --config=webpack.prod.js",
    "test": "jest --coverage"
},
```

Para poder realizar pruebas automáticas del DOM vamos a integrar la siguiente librería:

```bash
$> npm install --save-dev @testing-library/dom
```

De forma que podemos ejecutar `npm run test` para lanzar todas las pruebas del proyecto y recoger el informe de cobertura gracias al modificador --coverage.

## Configuración de linters

### Linter de código

A fin de que todo el equipo de desarrollo haga uso de una misma guía de estilo, vamos a incluir la dependencia:

```bash
$> npm install --save-dev semistandard
```

y el siguiente script en el fichero package.json, también vamos a añadir configuración para que no marque errores en los tests, quedando de esta forma:

```javascript
"scripts": {
    "start": "webpack serve --config=webpack.dev.js",
    "build": "webpack --config=webpack.prod.js",
    "test": "jest",
    "lint": "semistandard --fix"
},
"semistandard": {
    "globals": [
      "describe",
      "context",
      "before",
      "beforeEach",
      "after",
      "afterEach",
      "it",
      "expect"
    ]
  },
```

Pudiendo ejecutar `npm run lint` para descubrir las violaciones en las reglas de estilo y con el modificador --fix solucionarlas de forma automática.

>  Nota: no se comprobarán los ficheros que estén dentro del fichero .gitignore.

### Linter de mensajes de commit

Vamos a añadir a nuestro proyecto la dependencia de husky, la cual nos permitirá establecer una serie de hooks de Git que nos permitan verificar que los mensajes de commit cumplen con el standard de Conventional Commits 

```bash
$> npm install --save-dev husky
$> npm install --save-dev @commitlint/cli
$> npm install --save-dev @commitlint/config-conventional
```

Creamos en la raíz del proyecto el directorio .husky y dentro creamos el fichero "commit-msg" con el siguiente contenido para comprobar que el mensaje cumple con el formato de Conventional Commits:

```shell
#!/bin/sh
. "$(dirname "$0")/_/husky.sh"

npx --no-install commitlint --edit $1
```

Y otro llamado "pre-commit" para ejecutar los test antes de hacer un commit:

```shell
#!/bin/sh
. "$(dirname "$0")/_/husky.sh"

npm test
```

Y en el fichero package.json incluyo el siguiente script para instalar husky al hacer un prepare del proyecto y añado la siguiente configuración:

```javascript
"scripts": {
    "start": "webpack serve --config=webpack.dev.js",
    "build": "webpack --config=webpack.prod.js",
    "test": "jest",
    "lint": "semistandard",
    "prepare": "husky install"
},
"commitlint": {
    "extends": "@commitlint/config-conventional"
},
```

Para probarlo tenemos que inicializar git en el proyecto:

```bash
$> git init
```

Ejecutar un install de npm:

```bash
$> npm install
```

>  Al tratar de hace un commit tenemos que ver que ejecuta los tests y nos saca un mensaje de error si no se cumple con el formato marcado.



## Conclusiones

Con estos pasos ya tenemos preparado nuestro proyecto para poder implementar la solución que necesitemos.

