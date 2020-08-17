---
published: false
title: "Creación de un Custom Pipe para validar contra un servicio en NestJS"
cover_image: 
description: 
tags: nestjs
series: NestJS
canonical_url:
---

# Resumen

Vamos a ver cómo implementar una validación contra base de datos a través de la creación de un custom pipe donde se va a inyectar el servicio de comunicación con la base de datos.

# Entorno

* Slimbook Prox15 32 Gb RAM i7 
* SlimbookOS (Ubuntu 20.04)
* NestJS 7.0.0
* Visual Studio Code

# Introducción

Si una cosa tiene buena NestJS es su documentación, [pero en el caso de los Pipes](https://docs.nestjs.com/pipes) no encuentras un caso de uso muy común, como es el de validar un dato de entrada contra la base de datos y los ejemplos que vienen pueden llevarnos a equivocarnos o pensar que no se puede, así que vamos a ver que si se puede.

# Vamos al lío

Para ilustrar el ejemplo vamos a implementar un custom pipe que dado el valor de un id compruebe si se encuentra en base de datos y si no se encuentra lanzaremos una excepción de "Bad Request" al cliente.

Lo primero que tenemos que hacer es crear nuestro custom pipe, el cual tiene que extender de PipeTransform, lo que nos obliga a implementar nuestra lógica en el método transform que recibe el valor asociado al pipe y el metada de la información. Esta sería la implementación por defecto de cualquier Pipe.

```ts
import { PipeTransform, Injectable, ArgumentMetadata } from '@nestjs/common';

@Injectable()
export class ValIdPipe implements PipeTransform {
  transform(value: any, metadata: ArgumentMetadata) {
    return value;
  }
}
```

Como queremos hacer una consulta a la base de datos con el valor que llega en el argumento "value" tenemos que injectar el servicio a través del constructor, y hacer uso de él, en la lógica del método transform. El servicio nos tendrá que devolver un error cuando el id no se encuentre en base de datos, que capturaremos para lanzar la excepción apropiada. El código podría quedar de esta forma:

```ts
import { Dog } from "@entities/dog.entity";
import { ArgumentMetadata, BadRequestException, Injectable, Logger, PipeTransform } from "@nestjs/common";
import { InjectRepository } from "@nestjs/typeorm";
import { Repository } from "typeorm";

@Injectable()
export class ValIdDogPipe implements PipeTransform {

    constructor(@InjectRepository(Dog)
    public dogsRepository: Repository<Dog>) {}
    
    async transform(value: any, metadata: ArgumentMetadata) {
        try {
            await this.dogsRepository.findOneOrFail(value);
        }catch(err) {
            throw new BadRequestException("Id no existe");
        }
        
        return value;
    }

}
```

> En el ejemplo estoy inyectando directamente el repository para dejar claro que hace una consulta a base de datos, pero lo normal es que la llamada a base de datos se haga a través de un servicio de negocio o por QueryBus si se está utilizando el módulo de CQRS.

Como ves la implementación es muy sencilla. Ahora viene la segunda parte y donde digo que la documentación de NestJS puede llevarnos a equivoco: el momento de hacer uso del custom pipe en el controlador.

Para hacer uso del custom pipe tenemos que anotar el método del controlador con @UsePipes y declarar la clase que acabamos de implementar, el caso es que en la documentación tenemos este ejemplo:

```
@Post()
@UsePipes(new JoiValidationPipe(createCatSchema))
async create(@Body() createCatDto: CreateCatDto) {
  this.catsService.create(createCatDto);
}
```

Donde puede parecer que tenemos que hacer un new de la clase para poder utilizarlo, y como nosotros estamos inyectando el servicio, se nos complica la forma de poder pasarle el argumento que requiere al estar en una anotación.

Pero la realidad es que si hacemos uso del tipo y no de la instancia, es decir, quitamos el new y los paréntesis, el framework infiere la inyección y todo funciona a la perfección. Con lo cual para hacer uso del custom pipe solo tenemos que hacer:

```ts
@Get(/dogs/:id)
@UsePipes(ValIdDogPipe)
getDogById(@Param('id') id: string) {
  ...
}
```

Con lo que si el id no existe, el cliente lo que recibirá será una respuesta indicando:

```ts
{
    "statusCode": 400,
    "message": "Id no existe",
    "error": "Bad Request"
}
```

Otro detalle es que haciéndolo a nivel de método, si tenemos más de un parámetro en la URL, el pipe se va a ejecutar tantas veces como parámetros estén definidos, así que una mejor forma, que evita este problema, es declarar el pipe a nivel de parámetro de esta forma:

```ts
@Get(/dogs/:id)
getDogById(@Param('id', ValIdDogPipe) id: string) {
  ...
}
``