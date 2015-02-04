# Empezando

Echemos un vistazo a la arquitectura de Codeception. Entendemos que ya ha sido instalado [instalado](http://codeception.com/install), y que instalaste las primeras pruebas. Codeception ha generado tres de ellas: unitaria, funcional, y de aceptación. En el capítulo anterior se describen. Dentro de la carpeta __/tests__ hay tres ficheros de configuración y tres directorios cuyos nombres corresponden al tipo de pruebas. Cada grupo es independiente con un propósito común.

## Actores

Una de los principales conceptos de Codeception es la representación de las pruebas como acciones realizadas por una persona. Tambien existe un testeador Unitario, que ejcecuta funciones y prueba el código. También hay un testeador Funcional, un probador cualificado, que pone a prueba la aplicación en su conjunto, que conoce el funcionamiento interno de la misma. Y tambien tenemos un testeador Aceptación, un usuario final que trabaja con la aplicaicón con el interface que le proporcionamos.

Cada uno de estos actores son clases PHP, junto con las acciones que les permite hacer.Como se puede apreciar, cada uno de los actores tiene capacidades diferentes. Ellos nos son constantes, se pueden ampliar. Un actor pertenece a un grupo de pruebas.

Las clases actores, no están escritas, se generan automáticamente  a partir de la configuración del tipo de test. Cuando cambia la configuración, las clases actores se vuelven a **generar automaticamente**.

Si las clases no se regenerean o actualizan como esperabam se pueden hacer de forma manual con el comando `build`:

```bash
$ php codecept.phar build
```


## Escenario de ejemplo

Por defecto, las pruebas se esciben como escenarios narrados. Para crear un fichero PHP válido como escenarios el nombre tiene que tener el sufijo `Cept`. 

Vamos a crear un fichero de pruebas `tests/acceptance/SigninCept.php`

Lo hacemos ejecutando el siguiente comando:

```bash
$ php codecept.phar generate:cept acceptance Signin
```

Un escenario siempre comienza por la inicialización de la clase Actor. Después se escriben líneas que comienzan por `$I->` seguidas de la acción adecuada de la lista de autocompletar.

```php
<?php
$I = new AcceptanceTester($scenario);
?>
```

Vamos a probar nuestro sitio web. Supongamos que tengamos una página de 'login'donde nos identificamos con un usuario y una password. Cuando lo hemos introducido se nos redirige a una página de usuario, donde podemos ver el texto `Hola, %username%`. Veamos como está hecho este escenario en Codeception.

```php
<?php
$I = new AcceptanceTester($scenario);
$I->wantTo('log in as regular user');   //yo quiero
$I->amOnPage('/login');                 // yo estoy en la página
$I->fillField('Username','davert');     // yo relleno el campo usuario con 'davert'
$I->fillField('Password','qwerty');     // yo relleno el campo password con 'qwerty'
$I->click('Login');                     // yo pulso el botón 'Login'
$I->see('Hello, davert');               // yo veo una página que pone 'Hola, davert'
?>
```

Antes de ejecutar la prueba, debemos de asegurarnos de que el sitio web está en el servidor local. Para ello abrimos el fichero `tests/acceptance.suite.yml` y ponemos la URL correcta de nuestra aplicación Web:

``` yaml
config:
    PhpBrowser:
        url: 'http://myappurl.local'
```

Después de configurarlo, ejecutamos la prueba con el siguiente comando `run`:

``` bash
$ php codecept.phar run
```

Esta es la salida que deberíamos de ver:

``` bash
Acceptance Tests (1) -------------------------------
Trying log in as regular user (SigninCept.php)   Ok
----------------------------------------------------

Functional Tests (0) -------------------------------
----------------------------------------------------

Unit Tests (0) -------------------------------------
----------------------------------------------------

Time: 1 second, Memory: 21.00Mb

OK (1 test, 1 assertions)
```

Si queremos una información mas detallada:

```bash
$ php codecept.phar run acceptance --steps
```

Entonces veríamos un informe paso a paso de las diferentes acciones realizadas.

```bash
Acceptance Tests (1) -------------------------------
Trying to log in as regular user (SigninCept.php)
Scenario:
* I am on page "/login"
* I fill field "Username" "davert"
* I fill field "Password" "qwerty"
* I click "Login"
* I see "Hola, davert"
  OK
----------------------------------------------------  

Time: 0 seconds, Memory: 21.00Mb

OK (1 test, 1 assertions)
```

Esta ha sido una prueba muy simple que se puede repetir en su propio sitio web.
Al emulas las acciones del usuario, se puede probar todos los sitios web de la misma manera.
Inténtalo !!

## Modulos y Asistentes

Las acciones de las clases actor se toman a partir de módulos. Las clases de actor generadas emulan la herencia múltiple. Los módulos estan diseñados ppara tener una acción con un método. De acuerdo con el [principio DRY](http://es.wikipedia.org/wiki/No_te_repitas), si utilizas los mismos componentes de escenarios en diferentes módulos, puedes combinarlos y moverlos a un módulo personalizado. Por defecto, cada grupo tiene un módulo vacio, que se puede utilizar para extender las clases de actor. Se almacenan en los directorios __support__.

## Arranque

Cada grupo tiene su propio fichero de arranque. Está en el directorio del grupo con el nombre de `_bootstrap.php`. Este fichero se ejecutará antes que el grupo de pruebas. También hay un archivo de arranque general que esta en el directorio `tests`. Se utiliza para incluir ficheros adicionales.

## Formatos de pruebas

Codeception soporta tres formatos de pruebas. Junto con el formato basado en escenarios, descritos anteriormente CEPT, también se pueden ejecutar [formatos de fichero de pruebas PHPUnit para pruebas unitarias](http://codeception.com/docs/06-UnitTests), y formato de pruebas [clases basadas en  Cest](http://codeception.com/docs/07-AdvancedUsage#Cest-Classes). Estas se ecplicaran en próximos capítulos. No hay diferencia en como se realizan los test en cada uno de los formatos cuando se ejecutan en el grupo.

## Configuración

Hay un fichero de configuración en Codeception `codeception.yml` y uno de configuración en cada grupo. También soporta laos ficheros de configuración. También soporta ficheros de configuración `.dist`. Si hay varios desarrolladores en un proyecto, en el fichero `codeception.dist.yml` se pone los ajustes compartidos y los ajustes personales en `codeception.yml`. Lo mismo ocurre con las configuraciones de grupo. Por ejemplo, el fichero `unit.suite.yml` se fusionará con `unit.suite.dist.yml`. 


## Ejecutando las pruebas

Las pruebas se ejecutan con el comando `run`.

```bash
$ php codecept.phar run
```

Con un primer argumento podemos ejecutar pruebas de un grupo.

```bash
$ php codecept.phar run acceptance
```

Si queremos efectuar una prueba determinada, añadimos un segundo argumento. Se le dice la ruta local al fichero, desde el directorio del grupo.

```bash
$ php codecept.phar run acceptance SigninCept.php
```

Alternativamente se le pude decir la ruta completa al fichero de prueba:

```bash
$ php codecept.phar run tests/acceptance/SigninCept.php
```

Se puede ejecutar una prueba desde una clase de pruebas (para Cest o formatos de pruebas)

```bash
$ php codecept.phar run tests/acceptance/SignInCest.php:anonymousLogin
```

Sele puede decir una la ruta a un directorio:

```bash
$ php codecept.phar run tests/acceptance/backend
```

Esto ejecutará todos las pruebas del directorio backend.

Para ejecutar un grupo de pruebas que nos están en el mismo directorio , se puede hacer organizandolo así [groups](http://codeception.com/docs/07-AdvancedUsage#Groups).

### Informes

Para generar una salida XML JUnit, tenemos que ponerle la opcion `--xml` , y `--html` para informes en HTML.

```bash
$ php codecept.phar run --steps --xml --html
```

Este comando ejecutará todos las pruebas de todos los grupos, mostrando los pasos y creando los informes HTML y XML. Los informes se guardan en le directorio `tests/_output/`.

Para saber todas las opciones posibles, ejecutar el siguiente comando:

```bash
$ php codecept.phar help run
```

## Trazar

Para recibir una salida detallada, las pruebas s epueden ejecutar con la opción `--debug`.
Se puede imprimir cualquier información dentro de una prueba con la función `codecept_debug`.

### Generadores

Hay un montón de útiles comandos Codeception:

* `generate:cept` *suite* *filename* - Generates a sample Cept scenario
* `generate:cest` *suite* *filename* - Generates a sample Cest test
* `generate:test` *suite* *filename* - Generates a sample PHPUnit Test with Codeception hooks
* `generate:phpunit` *suite* *filename* - Generates a classic PHPUnit Test
* `generate:suite` *suite* *actor* - Generates a new suite with the given Actor class name
* `generate:scenarios` *suite* - Generates text files containing scenarios from tests


## Conclusión

Veamos la estructura de Codeception. La mayoría de las cosas necesarias ya fueron creadas por el comando `bootstrap`.Después de haber revisado los conceptos y las configuraciones básicas, podemos comenzar a escribir el priemr escenario.
