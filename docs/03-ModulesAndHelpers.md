# Módulos y Asistentes

Para crear un entorno confortable de pruebas, Codeception utiliza modularidad para grupo d epurebas que se escribe.
Los módulos permiten elegir las acciones y afirmaciones que se quieran realizar para las pruebas

Todas las acciones y afirmaciones que s epueden realizar en un objeto Tester en una clase, se definen en los módulos. Podría parecer que Codeception te limita en las pruebas, pero no es cierto. Puedes ampliar el grupo de pruebas con tus propias acciones y afirmaciones, escribiéndolos en un módulo personalizado.

Miremos el siguiente ejemplo:

```php
<?php
$I = new FunctionalTester($scenario);
$I->amOnPage('/');
$I->see('Hello');
$I->seeInDatabase('users', array('id' => 1));
$I->seeFileFound('running.lock');
?>
```

Se puede operar con diferentes entidades: la página web se carga con el módulo PHPBrowser, las afirmaciones de la base de datos utilizan el módulo Db, y un fichero de estado puede ser verificado con el módulo Filesystem.

Los módulos son adjuntados en la clase Actor en el fichero de configuración dle grupo .
Por ejemplo, en `tests/functional.suite.yml` podemos ver:

```yaml
class_name: FunctionalTester
modules:
    enabled: [PhpBrowser, Db, Filesystem]
```

La clase FunctionalTester tiene sus métodos definidos en módulos. Actualmente no contiene ninguna de ellas, sino que actua como un proxy. Se sabe que módulo ejecuta esta acción y los parámetros que pasa. Para que el IDE vea todos los métodos de FunctionalTester, se utiliza el comando `build`. El genera la definición de la clase FunctionalTester copiando las firmas en los correspondientes módulos.

## Módulos estandar

Codeception tiene muchos módulos empaquetados que ayudan a realizar pruebas para diferentes propósitos y diferentes ambientes. El número de módulos va creciendo constantemente a medida que se soportan mas frameworks y ORM`s. Se pueden ver la lista de módulos en el menú principal en la sección módulos.

Todos esos módulos estan documentados. Se puede revisar sus referencias detalladas en [GitHub](https://github.com/Codeception/Codeception/tree/master/docs/modules).

## Asistentes

Codeception no restringe solo los módulos desde el directorio principal. Sin duda, un proyecto podría necesitar añadir sus propias acciones al grupo de pruebas. Al ejecutar el comando `bootstrap`, Codeception genera tres módulos ficticios, uno por cada uno de los grupos de pruebas de nueva creación. Estos módulos personalizados son denominados 'Asistentes', y se encuentran en el directorio `tests/_support`.

Es una buena idea definir las acciones o comandos que faltan en los asistentes.

Nota: Los nombres de las clases asistentes deben terminar por "*Helper.php"

Digamos que vamos a extender la clase FunctionalHelper class. Por defecto el asistente esta relacionado con una clase FunctionalTester y un grupo de pruebas funcional.

```php
<?php
namespace Codeception\Module;
// here you can define custom functions for FunctionalTester

class FunctionalHelper extends \Codeception\Module
{
}
?>
```

En cuanto a las acciones, todo es bastante simple. Cada accion se define en una función pública. Escribe cualquier método público y ejecuta el comando `build`, y verá la nueva función añadidia a la clase FunctionalTester.

Nota: Los métodos públicos con el prefijo `_` son tratados como ocultos y no se añadirán a su clase actor. 

Las afirmaciones pueden ser un poco mas complicadas. Primero de todo, es recomendable poner los prefijos `see` o `dontSee` a todas las afirmaciones.

Ejemplo de como llamar a las afirmaciones:

```php
<?php
$I->seePageReloaded();
$I->seeClassIsLoaded($classname);
$I->dontSeeUserExist($user);
?>
```
Y luego llamarlar en sus pruebas:

```php
<?php
$I = new FunctionalTester($scenario);
$I->seePageReloaded();
$I->seeClassIsLoaded('FunctionalTester');
$I->dontSeeUserExist($user);
?>
```

Se pueden definir afirmaciones usando métodos assertXXX en módules. No todos métodos de afirmación de PHPUnit estan incluidos en módulos, pero podemos usar métodos estáticos de PHPUnit desd ela clase `PHPUnit_Framework_Assert` para aprovechar todos ellos.

```php
<?php

function seeClassExist($class)
{
    $this->assertTrue(class_exists($class));
    // or
    \PHPUnit_Framework_Assert::assertTrue(class_exists($class));
}
?>
```

En sus asistentes puede usar estas afirmaciones:

```php
<?php

function seeCanCheckEverything($thing)
{
    $this->assertTrue(isset($thing), "this thing is set");
    $this->assertFalse(empty($any), "this thing is not empty");
    $this->assertNotNull($thing, "this thing is not null");
    $this->assertContains("world", $thing, "this thing contains 'world'");
    $this->assertNotContains("bye", $thing, "this thing doesn`t contain 'bye'");
    $this->assertEquals("hello world", $thing, "this thing is 'Hello world'!");
    // ...
}
?>
```

Con `$this->assert` se pueden ver todos ellos.


### Resolviendo colisiones

¿Que ocurre si tenemos dos módulos que contienen acciones con el mismo nombre?
Codeception permite anular acciones cambiando el orden de los módulos
La acción del segundo módulo será leída y la del primer módulo será ignorada.
El orden de los módulos se define en la configuración del grupo.

### Módulos de conexión

Es posible que se necesite acceder a datos o funciones internas de otro módulo. Por ejemplo se podría necesitar una conexion con Doctrine o un navegador web de Symfony.

Los módulos pueden interactuar entre ellos a través del método`getModule`. Hay que tener en cuenta que dará una excepción si el módulo requerido no está cargado.

Imaginemos que se quiere escribir un módulo que conecte con la base de datos. Se supone que debe usar los valores de conexión dbh del módulo Db.

```php
<?php

function reconnectToDatabase() {
    $dbh = $this->getModule('Db')->dbh;
    $dbh->close();
    $dbh->open();
}
?>
```

Mediante el uso de la función `getModule`, se tnedrá acceso a todos los métodos públicos y propiedades del módulo solicitado. La propiedad dbh se definió como pública especificamente para estar accesible para otros módulos.

Esta técnica se puede utilizar cuando se necesite llevar a cabo una secuencia de acciones de otros métodos.

Por ejemplo:

```php
<?php
function seeConfigFilesCreated()
{
    $filesystem = $this->getModule('Filesystem');
    $filesystem->seeFileFound('codeception.yml');
    $filesystem->openFile('codeception.yml');
    $filesystem->seeInFile('paths');
}
?>
```

### Ganchos 

Cada módulo puede manejar eventos de la prueba de funcionamiento. Un módulo puede ser ejecutado antes de que comience la prueba, o después de finalizada la prueba. Esto puede ser útil para las acciones de arranque / limpieza. 
También puede definir un comportamiento especial para cuando falla la prueba. Esto le puede ayudar en la depuración de la cuestión. 
Por ejemplo, el módulo PhpBrowser guarda la página web actual en el directorio `tests/_output` cuando falla una prueba. 

Todos los ganchos están definidos en `\Codeception\module` y se enumeran aquí. Usted es libre de redefinirlos en su módulo.

```php
<?php

    // HOOK: used after configuration is loaded
    public function _initialize() {
    }

    // HOOK: on every Actor class initialization
    public function _cleanup() {
    }

    // HOOK: before each suite
    public function _beforeSuite($settings = array()) {
    }

    // HOOK: after suite
    public function _afterSuite() {
    }    

    // HOOK: before each step
    public function _beforeStep(\Codeception\Step $step) {
    }

    // HOOK: after each step
    public function _afterStep(\Codeception\Step $step) {
    }

    // HOOK: before test
    public function _before(\Codeception\TestCase $test) {
    }

    // HOOK: after test
    public function _after(\Codeception\TestCase $test) {
    }

    // HOOK: on fail
    public function _failed(\Codeception\TestCase $test, $fail) {
    }
?>
```
Tenga en cuenta que los métodos con un prefijo `_` no se agregan a la clase Actor. Esto les permite ser definidos como público, sino utilizados únicamente para fines internos.

### Depuración 

Como hemos mencionado, el gancho `_failed` puede ayudar en la depuración de una prueba fallida. Usted tiene la oportunidad de guardar el estado de la prueba actual y mostrarla al usuario, pero no se limitan a esto. 

Cada módulo puede sacar valores internos que puede ser útil durante la depuración. 
Por ejemplo, el módulo PhpBrowser imprime el código de respuesta y la URL actual cada vez que se mueve a una nueva página. De este modo, los módulos no son cajas negras. Ellos están tratando de mostrar lo que está sucediendo durante la prueba. Esto hace que la depuración de pruebas menos doloroso. 
Para mostrar información adicional, utilice los metodos `debug` y` debugSection` del módulo. 
He aquí un ejemplo de cómo funciona para PhpBrowser:

```php
<?php
    $this->debugSection('Request', $params));
    $client->request($method, $uri, $params);
    $this->debug('Response Code: ' . $this->client->getStatusCode());
?>    
```

Esta prueba, ejecutándose con el módulo PhpBrowser en modo debug, presentará algo como esto:

```bash
I click "All pages"
* Request (GET) http://localhost/pages {}
* Response code: 200
```


### Configuración 

Los módulos se pueden configurar desde el archivo de configuración del grupo, o globalmente en `codeception.yml`. 

Parámetros obligatorios deben ser definidos en la propiedad `$requiredFields` de la clase del módulo. Aquí es cómo se hace en el módulo Db:

```php
<?php
class Db extends \Codeception\Module {
    protected $requiredFields = array('dsn', 'user', 'password');
?>
```

La próxima vez que inicie el grupo de pruebas sin configurar uno de estos valores, se lanzará una excepción. 

Para los parámetros opcionales, debe establecer los valores por defecto. La propiedad `$config` se utiliza para definir los parámetros opcionales, así como sus valores. En el módulo WebDriver usamos por defecto dirección del servidor de Selenium y el puerto.

```php
<?php
class WebDriver extends \Codeception\Module
{
    protected $requiredFields = array('browser', 'url');    
    protected $config = array('host' => '127.0.0.1', 'port' => '4444');
?>    
```
Los parámetros del host y el puerto se puede redefinir en la configuración del grupo. Los valores se encuentran en la sección `modules:config` del archivo de configuración.

```yaml
modules:
    enabled:
        - WebDriver
        - Db
    config:
        WebDriver:
            url: 'http://mysite.com/'
            browser: 'firefox'
        Db:
            cleanup: false
            repopulate: false
```

Los parámetros opcionales y obligatorios se puede acceder a través de la propiedad `$config`. Use `$this-> config['parámetro']` para acceder a su valor.

### Configuración dinámica

Si desea volver a configurar un módulo en tiempo de ejecución, puede utilizar el método `_reconfigure` del módulo. Usted puede llamarlo desde una clase de ayuda y modificar todos los campos que desee cambiar.


```php
<?php
$this->getModule('WebDriver')->_reconfigure(array('browser' => 'chrome'));
?>
```

Al final de la prueba, todos los cambios se revertirán a los valores de configuración originales.

### Opciones adicionales

Como cada clase, un asistente puede ser heredado de un módulo.

```php
<?php
namespace Codeception\Module;
class MySeleniumHelper extends \Codeception\Module\WebDriver  {
}
?>
```
En un asistente heredado, se reemplazan los métodos implementados con los suyos propios. También puede reemplazar los ganchos `_before`y `_after`, que podrían ser una opción cuando necesita personalizar arranque y paro de una sesión de evaluación. Si algunos de los métodos de la clase padre no debe utilizarse en un módulo hijo, puede desactivarlas. Codeception tiene varias opciones para ello:

```php
<?php
namespace Codeception\Module;
class MySeleniumHelper extends \Codeception\Module\WebDriver 
{
    // disable all inherited actions
    public static $includeInheritedActions = false;

    // include only "see" and "click" actions
    public static $onlyActions = array('see','click');

    // exclude "seeElement" action
    public static $excludeActions = array('seeElement');
}
?>
```
El parámetro `$includeInheritedActions` a falso, añade la posibilidad de crear alias para los métodos de padres.  Esto permite resolver conflictos entre módulos. Digamos que si queremos usar el módulo `db` con nuestro `SecondDbHelper`  que hereda de `db`. ¿Cómo podemos usar métodos `seeInDatabase` de ambos módulos? Vamos a ver.


```php
<?php
namespace Codeception\Module;
class SecondDbHelper extends Db {
    public static $includeInheritedActions = false;

    public function seeInSecondDb($table, $data)
    {
        $this->seeInDatabase($table, $data);
    }
}
?>
```

El parámetro `$includeInheritedActions` a falso no incluirá los métodos de las clases padre en el Actor generado. Aún así, puede utilizar los métodos heredados en su clase asistente. 

## Conclusión 

Los módulos son el verdadero poder de Codeception. Se utilizan para emular múltiples herencias para las clases Actor (UnitTester, FunctionalTester, AcceptanceTester, etc). 
Codeception ofrece módulos para emular las peticiones web, datos de acceso, interactuar con las bibliotecas populares de PHP, etc. 
Para su aplicación es posible que tenga que personalizar acciones. Estas pueden definirse en las clases de asistentes. 
Si usted ha escrito un módulo que puede ser útil a los demás, compartelo. 
