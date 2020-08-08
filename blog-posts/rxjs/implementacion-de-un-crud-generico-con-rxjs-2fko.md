---
published: true
title: "Implementación de un CRUD genérico con RxJS"
cover_image: 
description: 
tags: rxjs
series: rxjs
canonical_url:
---

# Resumen

En este tutorial vamos a ver un ejemplo práctico de aplicación de la forma de gestión de estado que vimos en este [tutorial anterior](https://dev.to/raguilera82/gestion-del-estado-de-una-spa-solo-con-rxjs-3bm2) por lo que te recomiendo que lo veas antes de seguir con éste.

# Entorno

* Slimbook Prox15 32 Gb RAM i7 
* SlimbookOS (Ubuntu 20.04)
* RxJS 6.6.2
* Visual Studio Code

# Introducción

No hay cosa más tediosa en las aplicaciones, y que tengamos que repetir tanto, que tener que implementar el típico CRUD de una entidad, que por lo general necesita mantener el estado para hacer que la UI sea reactiva ante los cambios de los usuarios.

Así que vamos a ver una forma de hacer más genérico este desarrollo y que una misma implementación nos valga para cualquier entidad.

# Vamos al lío

Lo primero que tenemos que hacer es fijar una serie de restricciones en nuestras entidades para hacerlas compatibles. En el caso de TypeScript esto lo podemos conseguir con interfaces y herencia; así que vamos a definir una interfaz "Base" que va a contener un campo id de tipo string o number, y que tendrá que ser heredado por todas las entidades que queramos manejar con esta implementación. La interfaz es tan sencilla como:

```typescript
export interface Base {
    id?: string | number;
}
```

Ahora vamos a definir un estado genérico para el manejo de cualquier entidad, lo que vamos a crear es una interfaz que va definir el array de elementos de la entidad y registrar si ha habido algún error. De esta forma:

```typescript
export interface CrudState<T extends Base> {
    isError?: boolean;
    elems?: T[];
}
```

Otro de los puntos clave de esta implementación es crear una interfaz con los métodos habituales del CRUD, de forma que la implementación del store no se acople a la forma de recuperar los datos. De esta forma:

```typescript
import { Observable } from 'rxjs';

export interface CrudRepository<T> {

    getAll(): Observable<T[]>;

    add(elem: T): Observable<T>;

    update(id: string | number, elem: T): Observable<T>;

    delete(id: string | number): Observable<string | number>;

}
```

Con estos métodos podemos: 
* (C) crear nuevos elementos
* (R) leer elementos ya existentes
* (U) actualizar algún elemento existente
* (D) borrar algún elemento existente

Con todos estos elementos, estamos en disposición de hacer la implementación de nuestro CRUD genérico con manejo de estado. Esta sería la implementación más básica:

```typescript
import { catchError, tap } from 'rxjs/operators';
import { Base } from 'src/app/domain/models/base';
import { Store } from '../store';
import { CrudRepository } from './crud-repository';
import { CrudState } from './crud-state';

export class CrudStore<T extends Base> extends Store<CrudState<T>> {

    constructor(public repository: CrudRepository<T>){
        super();
        this.getElems();
    }

    getElems(): Promise<T[]> {
        return this.repository.getAll().pipe(
            tap(elems => {
                const state: CrudState<T> = {
                    isError: false,
                    elems: elems
                }
                this.store(state);
            })
        ).toPromise();
    }

    addElem(elem: T): Promise<T> {
        return this.repository.add(elem).pipe(
            tap(el => {
                const elems = this.get().elems;
                const state: CrudState<T> = {
                    isError: false,
                    elems: [...elems, el]
                };
                this.store(state);
            }),
            catchError(err => {
                this.store({isError: true, elems: []})
                throw err;
            })
        ).toPromise();
    }

    updateElem(id: string | number, elem: T): Promise<T> {
        return this.repository.update(id, elem).pipe(
            tap(() => {
                const elems = this.get().elems;
                const e = Object.assign({}, elem);
                const index = elems.findIndex((el: T) => el.id === id);
                const newElems = [...elems.slice(0, index), e, ...elems.slice(index + 1)];
                const state: CrudState<T> = {
                    isError: false,
                    elems: newElems
                };
                this.store(state);
            })
        ).toPromise();
    }

    deleteElem(id: string | number): Promise<string | number> {
        return this.repository.delete(id).pipe(
            tap(() => {
                const elems = this.get().elems;
                const newElems = elems.filter(elem => elem.id !== id);
                const state: CrudState<T> = {
                    isError: false,
                    elems: newElems
                };
                this.store(state);
            })
        ).toPromise();
    }

}
```

A destacar:

* La clase está parametrizada para recibir una entidad que extienda de Base de forma que nos aseguramos que el campo id, ya sea string o number, va a existir en nuestras operaciones.
* La clase extiende de Store para tener todas las propiedades reactivas que vimos en el [tutorial anterior](https://dev.to/raguilera82/gestion-del-estado-de-una-spa-solo-con-rxjs-3bm2) y está parametrizado con el tipo de estado que queremos manejar.
* A través del constructor inyectamos el repositorio de datos pero a través de su interfaz, de forma que no nos acoplamos a una implementación específica.
* Todos los métodos devuelven el observable en forma de promesa, para aprovechar el async/await en cualquier framework que tengamos por encima y olvidarnos de tener que hacer desuscripción explícita.
* Es importante resaltar que la suscripción que si hay que eliminar de forma explícita es la que se haga al store a través del método público get$()
* Los métodos siempre hacen lo mismo, llaman al método del repositorio correspondiente, si hay un error lo registran en el store, y si no, trabajan con el array en memoria para reflejar el cambio en el store y notificar a los elementos interesados en el cambio.

Creamos un test unitario con una entidad "Example" que hace uso de un repositorio fake en memoria. Este sería el resultado:

```typescript
import { Observable, of } from 'rxjs';
import { Base } from 'src/app/domain/models/base';
import { CrudRepository } from './crud-repository';
import { CrudState } from './crud-state';
import { CrudStore } from "./crud-store";

describe('CRUD Store Test', () => {


    it('get all elems', async() => {
        
        const crudStore = new CrudStore<Example>(new FakeRepository());
        
        await crudStore.getElems();

        const state: CrudState<Example> = crudStore.get();
        expect(state.isError).toBe(false);
        expect(state.elems.length).toBe(2);

    })

    it ('add elem', async() => {

        const crudStore = new CrudStore<Example>(new FakeRepository());
        const addExample = {id:'3', name:'example-3'};

        await crudStore.addElem(addExample);

        const state: CrudState<Example> = crudStore.get();
        expect(state.isError).toBe(false);
        expect(state.elems.length).toBe(3);


    })

    it ('update elem', async() => {

        const exampleName = 'example-test-1'

        const crudStore = new CrudStore<Example>(new FakeRepository());
        const updateElem = {id: '1', name: exampleName};

        await crudStore.updateElem('1', updateElem);

        const state = crudStore.get();
        expect(state.isError).toBe(false);
        expect(state.elems[0].name).toEqual(exampleName);
        
    })

    it ('delete elem', async () => {
        const crudStore = new CrudStore<Example>(new FakeRepository());
        const deleteId = '1';

        await crudStore.deleteElem(deleteId);

        const state: CrudState<Example> = crudStore.get();
        expect(state.isError).toBe(false);
        expect(state.elems.length).toBe(1);

    })

})

class FakeRepository implements CrudRepository<Example> {
    
    getAll(): Observable<Example[]> {
        
        const example1 = {id: '1', name: 'example-1'}
        const example2 = {id: '2', name: 'example-2'}
        const examples = [example1, example2];

        return of(examples);
    }
    add(elem: Example): Observable<Example> {
        return of(elem);
    }
    update(id: string | number, elem: Example): Observable<Example> {
        return of(elem);
    }
    delete(id: string | number): Observable<string | number> {
        return of(id);
    }


}

export interface Example extends Base {
    id: string | number;
    name: string;
}
```

Ahora crear un CRUD de cualquier entidad de nuestro dominio será tan sencillo como crear un estado especifico donde definiríamos otras propiedades en el estado específicas de esta entidad:

```typescript
import { Dog } from '../../../domain/models/dog';
import { CrudState } from '../crud-store/crud-state';

export interface DogState extends CrudState<Dog> {}
```

Crear la interfaz para el repositorio que extienda de la del CRUD genérico de forma que si, por ejemplo, queremos añadir filtros específicos de esta entidad lo hagamos en esta interfaz, de esta forma:

```typescript
import { Observable } from 'rxjs/internal/Observable';
import { Dog } from '../../../domain/models/dog';
import { CrudRepository } from '../crud-store/crud-repository';

export interface DogsRepository extends CrudRepository<Dog> {

    filterByName(name: string): Observable<Dog[]>;

    filterByBreed(breed: string): Observable<Dog[]>;
}
```

Y en el store de la entidad solo tendríamos que incluir la implementación de los métodos que son específicos de dicha entidad:

```typescript
import { tap } from 'rxjs/operators';
import { Dog } from '../../../domain/models/dog';
import { CrudStore } from '../crud-store/crud-store';
import { DogsRepository } from './dogs-repository';
import { DogState } from './dogs-state';

export class DogsStore extends CrudStore<Dog> {

    constructor(public repository: DogsRepository) {
        super(repository);
    }

    filterDogsByName(name: string): Promise<Dog[]> {
        return this.repository.filterByName(name).pipe(
            tap(elems => {
                const state: DogState = {
                    isError: false,
                    elems: elems
                }
                this.store(state);
            })
        ).toPromise()
    }

    filterDogsByBreed(breed: string): Promise<Dog[]> {
        return this.repository.filterByBreed(breed).pipe(
            tap((elems: Dog[]) => {
                const state: DogState = {
                    isError: false,
                    elems: elems
                }
                this.store(state);
            })
        ).toPromise()
    }
    
}
```

Y para el caso de Angular, la implementación del repositorio para esta entidad podría quedar de la siguiente forma:

```typescript
import { HttpClient } from '@angular/common/http';
import { Injectable } from '@angular/core';
import { of } from 'rxjs';
import { Observable } from 'rxjs/internal/Observable';
import { catchError } from 'rxjs/operators';
import { DogsRepository } from 'src/app/application/stores/dogs-store/dogs-repository';
import { Dog } from '../../domain/models/dog';

@Injectable({ providedIn: 'root' })
export class DogsRepositoryProxy implements DogsRepository {

    constructor(private httpClient: HttpClient) { }

    getAll(): Observable<Dog[]> {
        return this.httpClient.get<Dog[]>('http://localhost:3000/dogs');
    }

    add(elem: Dog): Observable<Dog> {
        return this.httpClient.post<Dog>('http://localhost:3000/dogs', elem).pipe(
            catchError(error => {
                console.log(error);
                throw error;
            })
        );
    }

    update(id: string | number, elem: Dog): Observable<Dog> {
        return of(elem);
    }
    delete(id: string | number): Observable<string | number> {
        return of(id);
    }

    filterByName(name: string): Observable<Dog[]> {
        return this.httpClient.get<Dog[]>('http://localhost:3000/dogs/filter/name/' + name);
    }

    filterByBreed(breed: string): Observable<Dog[]> {
        return this.httpClient.get<Dog[]>('http://localhost:3000/dogs/filter/breed/' + breed);
    }

}
```

# Conclusiones

Como ves un caso tan común como el de hacer un CRUD de una entidad se puede resolver de una forma muy práctica ahorrando mucho código repetitivo, pero dejando la posibilidad de extender las capacidades en función de las necesidadades de la entidad en cuestión.