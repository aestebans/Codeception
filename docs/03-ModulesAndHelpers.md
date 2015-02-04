# Módules y Asistentes

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

### Hooks

Each module can handle events from the running test. A module can be executed before the test starts, or after the test is finished. This can be useful for bootstrap/cleanup actions.
You can also define special behavior for when the test fails. This may help you in debugging the issue.
For example, the PhpBrowser module saves the current webpage to the `tests/_output` directory when a test fails.

All hooks are defined in `\Codeception\Module` and are listed here. You are free to redefine them in your module.

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

Please note that methods with a `_` prefix are not added to the Actor class. This allows them to be defined as public but used only for internal purposes.

### Debug

As we mentioned, the `_failed` hook can help in debugging a failed test. You have the opportunity to save the current test's state and show it to the user, but you are not limited to this.

Each module can output internal values that may be useful during debug.
For example, the PhpBrowser module prints the response code and current URL every time it moves to a new page.
Thus, modules are not black boxes. They are trying to show you what is happening during the test. This makes debugging your tests less painful.

To display additional information, use the `debug` and `debugSection` methods of the module.
Here is an example of how it works for PhpBrowser:

```php
<?php
    $this->debugSection('Request', $params));
    $client->request($method, $uri, $params);
    $this->debug('Response Code: ' . $this->client->getStatusCode());
?>    
```

This test, running with the PhpBrowser module in debug mode, will print something like this:

```bash
I click "All pages"
* Request (GET) http://localhost/pages {}
* Response code: 200
```



### Configuration

Modules can be configured from the suite config file, or globally from `codeception.yml`.

Mandatory parameters should be defined in the `$requiredFields` property of the module class. Here is how it is done in the Db module:

```php
<?php
class Db extends \Codeception\Module {
    protected $requiredFields = array('dsn', 'user', 'password');
?>
```

The next time you start the suite without setting one of these values, an exception will be thrown. 

For optional parameters, you should set default values. The `$config` property is used to define optional parameters as well as their values. In the WebDriver module we use default Selenium Server address and port. 

```php
<?php
class WebDriver extends \Codeception\Module
{
    protected $requiredFields = array('browser', 'url');    
    protected $config = array('host' => '127.0.0.1', 'port' => '4444');
?>    
```

The host and port parameter can be redefined in the suite config. Values are set in the `modules:config` section of the configuration file.

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

Optional and mandatory parameters can be accessed through the `$config` property. Use `$this->config['parameter']` to get its value. 

### Dynamic Configuration

If you want to reconfigure a module at runtime, you can use the `_reconfigure` method of the module.
You may call it from a helper class and pass in all the fields you want to change.

```php
<?php
$this->getModule('WebDriver')->_reconfigure(array('browser' => 'chrome'));
?>
```

At the end of a test, all your changes will be rolled back to the original config values.

### Additional options

Like each class, a Helper can be inherited from a module.

```php
<?php
namespace Codeception\Module;
class MySeleniumHelper extends \Codeception\Module\WebDriver  {
}
?>
```

In an inherited helper, you replace implemented methods with your own realization.
You can also replace `_before` and `_after` hooks, which might be an option when you need to customize starting and stopping of a testing session.

If some of the methods of the parent class should not be used in a child module, you can disable them. Codeception has several options for this:

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

Setting `$includeInheritedActions` to false adds the ability to create aliases for parent methods.
 It allows you to resolve conflicts between modules. Let's say we want to use the `Db` module with our `SecondDbHelper`
 that actually inherits from `Db`. How can we use `seeInDatabase` methods from both modules? Let's find out.

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

Setting `$includeInheritedActions` to false won't include the methods from parent classes into the generated Actor.
Still, you can use inherited methods in your helper class.

## Conclusion

Modules are the true power of Codeception. They are used to emulate multiple inheritances for Actor classes (UnitTester, FunctionalTester, AcceptanceTester, etc).
Codeception provides modules to emulate web requests, access data, interact with popular PHP libraries, etc.
For your application you might need custom actions. These can be defined in helper classes.
If you have written a module that may be useful to others, share it.
Fork the Codeception repository, put the module into the __src/Codeception/Module__ directory, and send a pull request.
