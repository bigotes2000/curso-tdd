# Tests de cobertura


## Planteamiento

Si el código no se prueba, no funciona. Los tests de cobertura nos
ayudan a saber qué partes del código no están cubiertas por los tests
unitarios, y nos ayudan a establecer un nivel determinado de
cobertura, así como políticas, para asegurar la calidad del código.

## Al final de esta sesión

Se habrán incluido tests de cobertura en el proyecto y refactorizado
el código en caso necesario.

## Criterio de aceptación

Inclusión del badge de `codecov` con un porcentaje de cobertura aceptable.

## Tests de cobertura

Los tests de cobertura miden qué parte de nuestro código está cubierta por los tests unitarios (que, recordemos, son de caja blanca). Estos tests de cobertura funcionan tanto a nivel de línea, como de función o de paquetes, pero generalmente van a dar un porcentaje de líneas cubiertas por los tests unitarios.

Dependiendo del lenguaje, se hará con unas herramientas u otras. En Go, por ejemplo, es parte de la instrumentación del propio lenguaje.

```
go test -coverprofile=coverage.out
go tool cover -html=coverage.out
```

La primera ejecuta los tests y genera un fichero de salida, y la segunda orden abre un navegador con una página en la que nos muestra nuestro código y la cobertura que tiene, señalando las funciones que no están cubiertas. Sobre la clase [`HitosIV`que ya hemos usado anteriormente](https://github.com/JJ/HitosIV), estos serían los resultados.

![Cobertura de los tests en la clase HitosIV](img/gocover.png)

En este caso, las  líneas no cubiertas lanzan errores en caso de que
se encuentren algún problema. No hay que cubrir el 100% de las
eventualidades (por ejemplo, generar un JSON erróneo a ver si salta la
segunda), pero quizás la primera sí merece la pena que se cubra, así
que se añade un test adicional, pero
debemos
[modificar ligeramente el código](https://stackoverflow.com/a/46841524/891440) para
asegurar que sigue las mejores prácticas del lenguaje: 


![Nueva cobertura de los tests en la clase HitosIV](img/gocover-2.png)

Siempre es mejor en Go devolver un error que enviar a log un error fatal, así que este cambio en el código asegura que se pueda cubrir mejor con los tests.

> Y también demuestra que la calidad en el desarrollo no es siempre
> cuestión de escribir más o menos tests, sino de seguir buenas
> prácticas en el diseño de aplicaciones para que estos tests sean
> posibles y tengan la máxima cobertura. 

En Go se muestran las dos partes de los tests de cobertura: el
generador de datos, y el que crea los informes. En casi todos los
lenguajes va a ocurrir lo mismo, sólo que no va a estar integrado con
el lenguaje, sino que va a ser una o dos utilidades aparte del
compilador.

En `jest`, que hemos usado anteriormente con TypeScript, también está
incluido un sistema de tests de cobertura. Por ejemplo, si lo
aplicamos a la última versión de
nuestro [sistema de hitos](https://github.com/JJ/ts-milestones),
obtendremos un resultado como este:

```
 PASS  src/__tests__/all_test.ts
  ✓ Issue (3ms)
  ✓ Milestone (3ms)

------------|----------|----------|----------|----------|-------------------|
File        |  % Stmts | % Branch |  % Funcs |  % Lines | Uncovered Line #s |
------------|----------|----------|----------|----------|-------------------|
All files   |      100 |    83.33 |      100 |      100 |                   |
 Project.ts |      100 |    83.33 |      100 |      100 |                31 |
------------|----------|----------|----------|----------|-------------------|
Test Suites: 1 passed, 1 total
Tests:       2 passed, 2 total
Snapshots:   0 total
Time:        1.263s
Ran all test suites.
```

En este caso se indica que la cobertura de ramas es del 83%, lo que
puede oscilar entre lo inaceptable y totalmente aceptable. Por eso
tenemos que usar otro sistema que genere una página web en la que se
pueda ver claramente qué ha fallado, y ese sistema está integrado
dentro
de
[`istanbul` y se llama `nyc`](https://www.npmjs.com/package/nyc). Ejecutando

```
nyc report --reporter=html
```

Dentro de la carpeta `coverage` se generarán una serie de ficheros
HTML donde podremos consultar qué es lo que ocurre. Esto se puede
añadir al sistema donde se haga integración continua, como en este
caso
a
[GitHub Actions](https://github.com/JJ/ts-milestones/blob/master/.github/workflows/coverage.yml).

```yaml
name: Pasa tests de cobertura
on: [push]

jobs:
  build:
    name: Cobertura
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v1
        with:
          fetch-depth: 1
      - name: Instalación general
        run: npm install
      - name: Instala tests y cobertura
        run: sudo npm install -g jest nyc codecov
      - name: Ejecuta tests de cobertura
        run: nyc jest --coverage
      - name: Crea los informes
        run: nyc report --reporter=json > coverage/coverage.json
      - name: Sube los tests
        run: codecov
```

Los tres últimos pasos son los que ejecutan los [tests de cobertura](https://github.com/JJ/ts-milestones/commit/599e3f41ed6314f23603862b5da5079358df61c6/checks?check_suite_id=299177238) y
los
suben
[a codecov](https://codecov.io/gh/JJ/ts-milestones/src/master/src/Project.ts). La
cobertura es del 100%, así que no hay mucho que mejorar aquí.

> Se puede configurar GitHub para que en los pull requests sólo acepte
> uno si no empeora la cobertura, lo que es razonable. Si el código
> añadido tiene una parte no testeada, es que no es correcto.

 
## Actividad


Añadir los tests de cobertura al proyecto.

* Darse de alta en codecov o algún sitio similar.
* Añadir tests de cobertura al ejecutor de tareas y al sitio de
  integración continua.
* Añadir API key a Travis (desde la web) y subir automáticamente los
  tests de cobertura a la misma tras cada test.
