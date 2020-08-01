---
published: true
title: "Gestión del estado de una SPA solo con RxJS"
cover_image: 
description: 
tags: rxjs
series: rxjs
canonical_url:
---

# Resumen

La gestión del estado es uno de los problemas más comunes que tenemos que enfrentar a la hora de implementar una SPA con el framework o librería que usemos... si estamos en Angular tenemos tenemos que aprender ngrx, si estamos en Vue tenemos que aprender Vuex y si estamos en React tenemos que aprender Redux o... podemos buscar una solución más universal a través de RxJS.

# Entorno

* Slimbook Prox15 32 Gb RAM i7 
* SlimbookOS (Ubuntu 20.04)
* RxJS 6.6.2
* Visual Studio Code

# Introducción

A la hora de crear una SPA es crucial la gestión del estado ya que en mayor o menor medida vamos a querer saber si el usuario está autenticado, o si tenemos que mostrar un loading, o vamos a querer hacer reactiva nuestra interfaz ante cambios en el estado de una determinada entidad y que una parte de la aplicación responda a cambios en el estado desde otra parte de la aplicación.

Quizá el caso más común e ilustrativo sea cuando tienes una lista de elementos en pantalla y desde un formulario añades un nuevo elemento.. refresca la página para cargar todos los elementos o de forma reactiva modificas la interfaz para mostrar el nuevo elemento sin llamar al servidor?

Para poder hacer lo segundo y dependiendo de la tecnología que estés usando tendrás que aprender como funciona la librería de turno: en Angular sería ngrx, en Vue sería Vuex o en React sería Redux, entre otras; pero resulta que todas ellas tienen en común RxJS y aquí vamos a ver como implementar una solución de la forma más sencilla y universal posible.

# Vamos al lío

Como decimos la clave de todo está en RxJS, concretamente en un concepto llamado "BehaviourSubject" que nos permite almacenar en memoria el último valor emitido por el Observable. Gracias a los genéricos podemos implementar la siguiente solución y crear un Store para cualquier tipo de estado que queramos manejar.

```ts
import { BehaviorSubject, Observable } from 'rxjs';

export abstract class Store<T> {

    private state$: BehaviorSubject<T> = new BehaviorSubject<T>(undefined);

    get = (): T => this.state$.getValue();

    get$ = (): Observable<T> => this.state$.asObservable();

    store = (nextState: T) => this.state$.next(nextState);

}
```

Como ves es una clase sumamente sencilla que nos evita tener que aprender cómo funcionan todas las librerías antes mencionadas y nos permite conseguir nuestro objetivo que es la gestión del estado.

Se trata de una clase abstracta, por lo que por si misma no puede ser instanciada tiene que heredar de otra clase que será la que defina el tipo de estado que se va a gestionar, es decir, le da valor a T. 

El estado lo define como un BehaviorSubject al que es obligatorio darle un valor inicial, de ahí que establezcamos inicialmente "undefined".

Además establece un método público get() que va a devolver el valor actual del estado y un método público get$() que es el que nos va a permitir suscribirnos a los cambios del estado, es decir, cuando modifiquemos el valor del estado, cualquier elemento suscrito va a recibir el nuevo valor y podrá actuar en consecuencia.

Por último, tenemos el método público store() que nos permite modificar el valor del estado y notificar a los elementos suscritos.

# Ejemplo de uso sencillo

A partir de aquí podemos utilizar esta clase abstrata para cualquier estado que necesitemos manejar. Quizá el ejemplo más común y la vez más sencillo e ilustrativo sea cuando queremos saber si hay que mostrar un loading en la aplicación. 

Para este caso solo queremos manejar un estado donde a través de un boolean podamos determinar cuando hay que mostrar el componente que carga el loading de la aplicación.

Entonces creamos una clase pura en TypeScript, es importante que no tenga nada que la relacione con la tecnología que estemos utilizando, por ejemplo, si es Angular no podrá anotarse con @Injectable({provideIn: 'root'}) para no casarlo a ninguna tecnología en concreto.

También podemos hacer que el servicio tenga una única instancia con el patrón Singleton, de forma que estaremos compartiendo el estado gestionado en un único sitio para toda la aplicación, lo que en Angular se conseguiría con el ya mencionado @Injectable({provideIn: 'root'})

```ts
import { Store } from './../store';

export class LoadingStore extends Store<boolean> {

    private static instance: LoadingStore;

    private constructor(){
        super();
        this.hide();
    }

    static getInstance(): LoadingStore {
        if (!LoadingStore.instance) {
            LoadingStore.instance = new LoadingStore();
        }
        return LoadingStore.instance;
    }

    show() {
        this.store(true);
    }

    hide() {
        this.store(false);
    }

}
```

Y el test unitario asociado también queda muy sencillo al estar desacoplado del resto de la aplicación.

```ts
import { LoadingStore } from "./loading-store";

describe('Loading Store', () => {

    let loadingStore: LoadingStore = null;

    beforeEach(() => {
        loadingStore = LoadingStore.getInstance();
    })


    it('loading show', () => {
        
        loadingStore.show();

        expect(loadingStore.get()).toBe(true);
    })

    it('loading hide', () => {
        
        loadingStore.hide();

        expect(loadingStore.get()).toBe(false);
    })

})
```

Como ves creamos un método público show() para guardar un valor true en el estado cuando queremos que se muestre el componente de loading y un método público hide() cuando queremos que se oculte; los cuales pueden ser llamados desde cualquier parte de la SPA.

Del otro extremo vamos a tener un componente que se va a suscribir al estado a través del método get$(), por ejemplo, en Angular se haría de esta forma:

```ts
export class LoadingComponent implements OnInit {

    loading: boolean;

    ngOnInit() {
        const loadingStore = LoadingStore.getInstance();
        loadingStore.get$().subscribe(value => this.loading = value)
    }

}
```

Y en el template en función del valor del atributo "loading" vamos a mostrar el loading en la SPA o no. Dado que nuestro store es agnóstico a la tecnología, podemos utilizarlo en Angular, Vue, React o Vanilla sin problema, haciéndolo altamente portable solo tenemos que depender de RxJS.

# Conclusiones

Este es el caso más sencillo donde en el estado solo manejamos un boolean pero es fácil ver que podemos estar manejando estados todo lo complejos que necesitemos, y cubrir desde un conjunto de operaciones CRUD para una determinada entidad hasta un sistema de notificaciones... aunque todo esto lo vamos a cubrir en otros tutoriales.