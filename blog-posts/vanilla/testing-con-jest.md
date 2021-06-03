# Testing con Jest

[TOC]

## Introducción

Cuando hablas con personas del mundo del desarrollo te das cuenta de que muy pocos saben y hacen tests de sus implementaciones. Entonces, les pregunto "¿cómo me demuestras que lo que has hecho está bien?" Siempre me suelen contestar "pues mira abro la aplicación, pincho en el botón y sale lo que tiene que salir", me dicen orgullosos. Entonces es cuando les digo, "y si te pido que esto me lo demuestres cada vez que hagas un cambio en el código junto con las otras historias de usuario que has implementado para saber que esto se puede poner en producción…" entonces es cuando algunos cambian el rictus y otros, los más atrevidos, dicen "bueno pero esto ya lo validará el equipo de QA que para eso está".

Y ese, compañeros y compañeras, es uno de los mayores problemas de las grandes compañías que tienen un equipo de QA, que encima no automatiza las pruebas con lo cual el tiempo desde que una persona de negocio tiene una superidea, que va a reportar millones a su empresa, es directamente proporcional al tiempo que el equipo de QA, con sus pruebas en Excel supercurradas, va a tardar en validar ese desarrollo para su puesta en producción; perdiendo de esta forma la ventana de oportunidad y por tanto la ganancia de la idea.

¿Cómo podemos los desarrolladores minimizar este periodo al máximo? La respuesta es sencilla… estando seguros en todo momento que lo que se va a subir a producción es funcional y técnicamente correcto desde la fase de desarrollo; y esto solo se puede conseguir con tests y procesos automáticos que podamos repetir una y otra vez. De esta forma podríamos hacer subidas a producción con total confianza varias veces al día si fuera necesario.

Además el testing son da una serie de ventajas a tener en cuenta:

* Nos permite identificar los errores que han ocurrido en la fase de desarrollo.
* Tener una buena calidad en el software desarrollado.
* Minimizamos el mantenimiento y los costes asociados cuando el desarrollo está avanzado.
* Garantizamos que el software es fiable.

## Tests unitarios y de integración

No se puede empezar a hablar de testing en JavaScript sin mencionar a Jasmine que fue el pionero y que casi al 100% es compatible con la sintaxis de Jest.

Sus características principales son que no depende de ninguna otra librería JavaScript, tiene una sintaxis obvia y limpia que facilita la escritura de los tests, pudiendo añadir aseveraciones que nos permitan determinar si la ejecución ha sido correcta o no.

Además se puede utilizar para implementar tests unitarios y de integración.

### Test Suite

Para quien no esté familiarizado con el concepto de Test Suite simplemente es una forma de organizar casos de tests que estén relacionados.

La sintaxis comienza con la palabra reservada "describe" seguida de un texto descriptivo y una función anónima que contiene los casos de test que se van a ejecutar.

```js
describe ('texto descriptivo', () => {
   //Aquí van los casos de test relacionados
}
```

### Test Case

Dentro de un test suite puede haber uno o más casos de test, donde va a residir la lógica de verificación de nuestros tests. Se utiliza la palabra reservada "it".

```js
describe ('texto descriptivo', () => {
   it ('texto descriptivo', () => {
      //Caso de test
   }
}
```

### Matchers

Son una serie de funciones definidas que nos permiten añadir lógica de verificación a los casos de tests gracias a la función expect.

Tienes que tener en cuenta que JavaScript es un poco especial con el valor booleano de las variables de forma que cualquier variable que tenga valor: false, o cero, o cadena vacía, o "undefined", o null, o Nan será false, y el resto de casos que se te ocurran serán true.

* **toBe:** devuelve true haciendo la comparación de los elementos con el triple igual.
* **toEqual:** es similar al anterior pero se utiliza más para literales y objetos.
* **toMatch:** compara expresiones regulares.
* **toBeDefined:** comprueba si el elemento no es undefined.
* **toBeUndefined:** comprueba si el elemento devuelve undefined.
* **toBeNull:** comprueba si el elemento es nulo.
* **toBeTruthy:** hace cast y devuelve true si el valor está fuera de los posibles valores antes mencionados.
* **toBeFalsy:** hace cast y devuelve true si el valor  está dentro de los posibles valores antes mencionados.
* **toContain:** para comprobar si un elemento está presente en un array.
* **toBeLessThan:** compara dos elementos con el menor que.
* **toBeGreaterThan:** compara dos elementos con el mayor que.
* **toBeCloseTo:** comprueba la precisión de una operación matemática.
* **toThrow:** comprueba si se ha lanzado una excepción.
* **not.:** anteponiendo not a cualquiera de los anteriores matchers cambiamos el valor booleano esperado.

### beforeEach() y afterEach()

A fin de poder hacer reutilización de código en nuestros tests, tenemos las funciones beforeEach() y afterEach()

El contenido de la función beforeEach() se va ejecutar siempre antes de cada uno de los casos de test y el contenido de la función afterEach() se ejecutará justo después de la ejecución de cada uno de los casos de tests.

En el ejemplo podemos ver como la variable "count" toma el valor 10 antes de la ejecución de añadir uno a la variable, de forma que la aseveración que lo compara con el valor 11 es afirmativa, y justo después el valor de count se establece a cero.

```js
describe('A spec suite', () => { 
   var count
   beforeEach(() => {
     count = 10
   })
   afterEach(() => {
     count = 0 
   })
   it('should add one to count',() => { 
       count += 1
       expect(count).toEqual(11)
    })
})
```

> Fíjate cómo hacemos uso de expect junto con el matcher "toEqual" para verificar que el resultado es el esperado.

En el siguiente ejemplo podemos ver la unión de todos los conceptos para verificar el funcionamiento de la clase Calculadora.

```js
import {Calculadora} from './calculadora'

describe('Calculadora', () => { 
   let calc;
   beforeEach(() => {
     calc = new Calculadora()
   })
   it('should add two numbers',() => { 
        let a = 3
        let b = 2
        expect(calc.add(a, b)).toEqual(5)
    })
})
```
