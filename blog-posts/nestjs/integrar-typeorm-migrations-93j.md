---
published: true
title: "Integrar TypeORM migrations en NestJS"
cover_image: 
description: 
tags: nestjs
series: NestJS
canonical_url:
---

# Introducción

Migrations de TypeORM es la tecnología que nos permite llevar un control de los cambios en el modelo de base de datos con el fin de que sean automáticamente aplicados sobre la base de datos real cuándo subamos de versión la aplicación.

En el mundo Java la tecnología de migrations se puede asemejar a Flyway o Liquibase.

Estas tecnologías nos facilitan la obtención de las sentencias SQL necesarias para aplicar los cambios a una base de datos. Siempre guardan una tabla especial que registra los cambios que ya se han aplicado, de forma que estos cambios se aplican de forma incremental; es decir si la base de datos está vacía se aplican todos por orden, si ya tiene algún cambio aplicado, solo se van a aplicar los que falten en orden cronológico.

# Vamos al lío

Nota: para seguir este tutorial se recomienda haber visto este anterior donde hacemos uso de TypeORM para hacer un API Rest con PostgreSQL.

Lo primero que queremos es generar de forma automática el modelo de nuestras entidades para obtener los ficheros de migrations que nos permitirán hacer los cambios en base de datos al arrancar la aplicación.

Para eso TypeORM nos ofrece un CLI con distintas operaciones que configuraremos más adelante como scripts dentro del fichero package.json.

Si has seguido el tutorial anterior se habrás dado cuenta de que hemos configurado el acceso a base de datos en el fichero app.module.ts de esta forma:

```typescript
import { TypeOrmModule } from '@nestjs/typeorm';
import { UsersModule } from './users/users.module';

@Module({
  imports: [
    ConfigModule.forRoot(), 
    TypeOrmModule.forRoot({
      type: 'postgres',
      host: process.env.DB_HOST,
      port: parseInt(process.env.DB_PORT),
      username: process.env.DB_USER,
      password: process.env.DB_PASS,
      database: process.env.DB_NAME,
      autoLoadEntities: true,
      synchronize: !!process.env.DB_SYNC,
      keepConnectionAlive: true
    }),
    UsersModule
  ],
  controllers: [],
  providers: [],
})
export class AppModule {}
```

La primera piedra en el camino es NestJS delega completamente el manejo de migrations a TypeORM, es decir, no tiene una integración directa y por tanto esta configuración no puede ser leída por los scripts de migrations de TypeORM que veremos después.

En todos los tutoriales que he visto te recomiendan que saques esta configuración a un fichero ormconfig.json, que hace complicado poder seguir utilizando las variables de entorno para la definición de la conexión a base de datos.

Por tanto, lo mejor es crearse un fichero src/ormconfig.ts que si nos va a permitir hacer uso de las variables de entorno como ves a continuación:

```typescript
import { ConnectionOptions } from 'typeorm';

const config: ConnectionOptions = {
    type: 'postgres',
    host: process.env.DB_HOST,
    port: parseInt(process.env.DB_PORT),
    username: process.env.DB_USER,
    password: process.env.DB_PASS,
    database: process.env.DB_NAME,
    entities: [__dirname + '/**/*.entity{.ts,.js}'],
    synchronize: false,
    migrationsRun: true,
    logging: true,
    migrations: [__dirname + '/../migrations/**/*{.ts,.js}'],
    cli: {
        migrationsDir: './migrations',
    }
}

export = config;
```

En este fichero tenemos definidas las mismas propiedades, más todas las relativas a migrations, pero hemos perdido las que son propias de NestJS como el autoLoadEntities o el keepConnectionAlive que veremos cómo recuperarlas más adelante.

Detalles de esta configuración, con la propiedad "cli" le estamos indicando donde queremos que se generen los archivos de migrations y con la propiedad migrations de donde tiene que leerlos, además con la propiedad migrationsRun le estamos indicando que queremos aplicar todas los ficheros de migrations en el arranque de la aplicación.

Ahora aplicamos esta configuración a nuestro módulo de TypeOrm en el fichero app.module.ts, de forma que lo vamos a hacer añadiendo las propiedades que decíamos eran propias de NestJS, quedando de esta forma:

```typescript
import { TypeOrmModule } from '@nestjs/typeorm';
import { UsersModule } from './users/users.module';
import * as ormconfig from './ormconfig';

@Module({
  imports: [
    ConfigModule.forRoot(), 
    TypeOrmModule.forRoot({...ormconfig, 
                           keepConnectionAlive: true, 
                           autoLoadEntities: true}),
    UsersModule
  ],
  controllers: [],
  providers: [],
})
export class AppModule {}
```

Hecho esto es el momento de añadir los siguientes scripts al fichero package.json:

```javascript
...
"typeorm": "ts-node -r tsconfig-paths/register ./node_modules/typeorm/cli.js --config ./src/ormconfig.ts",
"typeorm:migrate": "npm run typeorm migration:generate -- -n",
"typeorm:run": "npm run typeorm migration:run"
...
```

De esta forma para generar un fichero de migración con el estado actual del modelo de entidades, tenemos que primero asegurarnos de que la base de datos está levantada y es accesible por la aplicación y después ejecutar:

```bash
$> npm run typeorm:migrate nombre-migracion
```

Si este comando acaba con éxito, se habrá creado la carpeta "migrations" en la raíz de nuestro proyecto y dentro se habrá creado un fichero con el patrón TIMESTAMP-nombre-migracion.ts que contiene dos métodos públicos: uno llamado "up" con las sentencias SQL necesarias para crear la base de datos de cero; y otro llamado "down" con las sentencias SQL necesarias para revertir los cambios de base de datos.

Este comando lo volveremos a ejecutar siempre que haya un cambio en el modelo de entidades de nuestra aplicación y se guardarán en el orden del TIMESTAMP.

Ahora cuando arranquemos la aplicación debemos ver que antes de arrancar se han ejecutado las sentencias SQL necesarias para crear las tablas y los datos que tengamos en los ficheros migrations de TypeORM.