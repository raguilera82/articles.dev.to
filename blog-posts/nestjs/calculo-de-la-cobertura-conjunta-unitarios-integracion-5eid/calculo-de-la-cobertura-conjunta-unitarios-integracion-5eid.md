---
published: true
title: "Cálculo de la cobertura conjunta (unitarios + integración) en NestJS con Jest"
cover_image: 
description: 
tags: nestjs
series: NestJS
canonical_url:
---

Uno de los cambios en la configuración por defecto de los proyectos de Nest que es interesante abordar es el poder tener una cobertura calculada de forma conjunta con el resultado de los tests unitarios y los test de integración.

Para ello tenemos que seguir los siguientes pasos: primero editamos el fichero test/jest-e2e-json dejándolo de esta forma:

```typescript
{
  "moduleFileExtensions": ["js", "json", "ts"],
  "rootDir": "..",
  "testMatch": ["**/*.e2e-spec.ts"],
  "testPathIgnorePatterns": ["/node_modules/", "/src/"],
  "transform": {
    "^.+\\.(t|j)s$": "ts-jest"
  },
  "coverageReporters": ["json", "lcov", "text", "html"],
  "coverageDirectory": "coverage/e2e",
  "collectCoverageFrom": ["src/**"],
  "coveragePathIgnorePatterns": [
    ".module.ts$",
    ".spec.ts$",
    "src/main.ts"
  ],
  "testEnvironment": "node"
}
```

> Lo más importante es establecer el resultado de la cobertura en el directorio coverage/e2e

Ahora editamos el fichero package.json, concretamente la sección de jest y la sustituimos en su totalidad por este contenido:

```javascript
"jest": {
    "moduleFileExtensions": [
      "js",
      "json",
      "ts"
    ],
    "rootDir": "src",
    "testRegex": ".spec.ts$",
    "transform": {
      "^.+\\.(t|j)s$": "ts-jest"
    },
    "coverageReporters": [
      "json",
      "lcov",
      "text",
      "html"
    ],
    "coverageDirectory": "../coverage/unit",
    "coveragePathIgnorePatterns": [
      ".module.ts$",
      ".spec.ts$",
      "src/main.ts"
    ],
    "testEnvironment": "node"
  }
```

Con la información de cobertura separada en dos carpetas independientes lo que tenemos que hacer ahora es mergear ambos resultados. Para ello en el raíz del proyecto creamos el fichero merge-coverage.ts con el siguiente contenido:

```typescript
import * as fs from 'fs-extra';
import { createReporter } from 'istanbul-api';
import { createCoverageMap } from 'istanbul-lib-coverage';
import * as yargs from 'yargs';

main().catch((err) => {
  console.error(err);
  process.exit(1);
});

async function main() {
  const argv = yargs
    .options({
               report: {
                 type: 'array', // array of string
                 desc: 'Path of json coverage report file',
                 demandOption: true,
               },
               reporters: {
                 type: 'array',
                 default: ['json', 'lcov', 'text'],
               },
             })
    .argv;

  const reportFiles = argv.report as string[];
  const reporters = argv.reporters as string[];

  const map = createCoverageMap({});

  reportFiles.forEach((file) => {
    const r = fs.readJsonSync(file);
    map.merge(r);
  });

  const reporter = createReporter();
  await reporter.addAll(reporters);
  reporter.write(map);
  console.log('Created a merged coverage report in ./coverage');
}
```

Es importante que antes de ejecutarlo instalemos las siguientes dependencias:

```bash
$> npm install --save-dev fs-extra istanbul-api
```

Y, por último, modificamos los scripts del fichero package.json para invocar al script de mergeo y establecer una serie de parámetros útiles. En el script de "verify" (que ejecutamos antes de subir la rama que vayamos a mergear con develop) levantamos la base de datos, ejecutamos todos los tests, calculamos la cobertura y volvemos a bajar la base de datos.

```javascript
...
"test": "jest",
"test:watch": "jest --watch",
"test:cov": "jest --coverage --passWithNoTests",
"test:debug": "node --inspect-brk -r tsconfig-paths/register -r ts-node/register node_modules/.bin/jest --runInBand",
"test:e2e": "jest --config ./test/jest-e2e.json --runInBand --detectOpenHandles --coverage",
"verify": "docker-compose up -d && npm run lint && npm run test:cov && npm run test:e2e && ts-node ./merge-coverage.ts --report coverage/unit/coverage-final.json --report coverage/e2e/coverage-final.json && docker-compose down -v",
...
```
