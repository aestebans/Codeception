
# Pruebas unitarias

Codeception utiliza PHPUnit como backend para la realización de pruebas. Por lo tanto, cualquier prueba PHPUnit se puede añadir al conjunto de pruebas Codeception y luego ejecutado. 
Si alguna vez escribiste una prueba PHPUnit hay que hacerlo como lo hiciste entonces. Codeception añade muchos asistentes para simplificar las tareas comunes. 
Los fundamentos de las pruebas unitarias se omiten aquí, en lugar obtendrá un conocimiento básico de lo que dispone Codeception se añadir a las pruebas unitarias.

____Hay que decirlo de nuevo: no es necesario instalar PHPUnit para ejecutar sus pruebas. Codeception puede correrlas también.__

## Creando la prueba

Codeception tiene buenos generadores para simplificar la creación de pruebas.
Puede comenzar con la generación de una prueba PHPUnit clásica con una clase que extiende de la clase  `\PHPUnit_Framework_TestCase`.

Esto puede hacerse con el comando:

```bash
$ php codecept.phar generate:phpunit unit Example
```

Codeception tiene complementos para las pruebas unitarias estándar, así que vamos a tratarlos. Necesitamos otro comando para crear pruebas unitarias con el motor Codeception.


```bash
$ php codecept.phar generate:test unit Example
```

Se creará un nuevo fichero `ExampleTest` dentro del directorio `tests/unit`.

La prueba creado por el comando `generate:test` se verá así:

```php
<?php
use Codeception\Util\Stub;

class ExampleTest extends \Codeception\TestCase\Test
{
   /**
    * @var UnitTester
    */
    protected $tester;

    // executed before each test
    protected function _before()
    {
    }

    // executed after each test
    protected function _after()
    {
    }
}
?>
```

Esta clase predefinida empieza por los métodos `_before` y `_after`. Se las puede utilizar para crear un objeto probado antes de cada prueba, y destruirlo después.

Como se ve, a diferencia de PHPUnit, los métodos `setup` y `tearDown` son reemplazados por sus alias: `_before` y ` _after`. 
El actual `setup` y `tearDown` fueron implementados por la clase padre `\Codeception\TestCase\test` y establecieron la clase UnitTester para tener todas las acciones  Cept-archivos frescas para ejecutarse como parte de las pruebas unitarias. 
Al igual que en la aceptación y pruebas funcionales se puede elegir los módulos adecuados para clase `UnitTester` en el fichero de configuración `unit.suite.yml`. 

```yaml
# Codeception Test Suite Configuration

# suite for unit (internal) tests.
class_name: UnitTester
modules:
    enabled: [UnitHelper, Asserts]
```

### Pruebas unitarias clásicas

Las pruebas unitarias en Codeception se escriben de la misma manera que como se hace en PHPUnit:

```php
<?php
class UserTest extends \Codeception\TestCase\Test
{
    public function testValidation()
    {
        $user = User::create();

        $user->username = null;
        $this->assertFalse($user->validate(['username']));

        $user->username = 'toolooooongnaaaaaaameeee';
        $this->assertFalse($user->validate(['username']));

        $user->username = 'davert';
        $this->assertTrue($user->validate(['username']));           
    }
}
?>
```

### Especificación de pruebas BDD

Al escribir las pruebas que se debe preparar para los constantes cambios en la aplicación. Las pruebas deben ser fáciles de leer y de mantener. Si se cambia una especificación en la aplicación, las pruebas deben actualizarse también. Si no se tiene un convenio para documentar las pruebas, dentro de su equipo, se tendrán problemas para averiguar qué pruebas se vieron afectados por la introducción de una nueva función. 

Es por eso que es muy importante no sólo para cubrir su solicitud con pruebas unitarias, sino hacer que las pruebas unitarias sean auto-explicativas. Se hace esto para el escenario de las pruebas de aceptación y pruebas funcionales, y debemos hacer esto tambien para las pruebas unitarias y de integración.

Para el caso de tener un proyecto independiente [Specify](https://github.com/Codeception/Specify) (el cual está incluido en el paquete phar) escribir las especificaciones dentro de las pruebas unitarias.


```php
<?php
class UserTest extends \Codeception\TestCase\Test
{
    use \Codeception\Specify;

    public function testValidation()
    {
        $user = User::create();

        $this->specify("username is required", function() {
            $user->username = null;
            $this->assertFalse($user->validate(['username']));
        });

        $this->specify("username is too long", function() {
            $user->username = 'toolooooongnaaaaaaameeee';
            $this->assertFalse($user->validate(['username']));
        });

        $this->specify("username is ok", function() {
            $user->username = 'davert';
            $this->assertTrue($user->validate(['username']));           
        });     
    }
}
?>        
```

Using `specify` codeblocks you can describe any piece of test. This makes tests much cleaner and understandable for everyone in your team.

Code inside `specify` blocks is isolated. In the example above any change to `$this->user` (as any other object property), will not be reflected in other code blocks.

Also you may add [Codeception\Verify](https://github.com/Codeception/Verify) for BDD-style assertions. This tiny library adds more readable assertions, which is quite nice, if you are always confused of which argument in `assert` calls is expected and which one is actual.

```php
<?php
verify($user->getName())->equals('john');
?>
```

## Using Modules

As in scenario-driven functional or acceptance tests you can access Actor class methods. If you write integration tests, it may be useful to include `Db` module for database testing. 

```yaml
# Codeception Test Suite Configuration

# suite for unit (internal) tests.
class_name: UnitTester
modules:
    enabled: [Db, UnitHelper]
```

To access UnitTester methods you can use `UnitTester` property in a test.

### Testing Database

Let's see how you can do some database testing:

```php
<?php
function testSavingUser()
{
    $user = new User();
    $user->setName('Miles');
    $user->setSurname('Davis');
    $user->save();
    $this->assertEquals('Miles Davis', $user->getFullName());
    $this->tester->seeInDatabase('users', array('name' => 'Miles', 'surname' => 'Davis'));
}
?>
```

Database will be cleaned and populated after each test, as it happens for acceptance and functional tests.
If it's not your required behavior, please change the settings of `Db` module for the current suite.

### Accessing Module

Codeception allows you to access properties and methods of all modules defined for this suite. Unlike using the UnitTester class for this purpose, using module directly grants you access to all public properties of that module.

For example, if you use `Symfony2` module, here is the way you can access Symfony container:

```php
<?php
/**
 * @var Symfony\Component\DependencyInjection\Container
 */
$container = $this->getModule('Symfony2')->container;
?>
```

All public variables are listed in references for corresponding modules.

### Cest

Alternatively to testcases extended from `PHPUnit_Framework_TestCase` you may use Codeception-specific Cest format. It does not require to be extended from any other class. All public methods of this class are tests.

The example above can be rewritten in scenario-driven manner like this:

```php
<?php
class UserCest
{
    public function validateUser(UnitTester $t)
    {
        $user = $t->createUser();
        $user->username = null;
        $t->assertFalse($user->validate(['username']); 

        $user->username = 'toolooooongnaaaaaaameeee';
        $t->assertFalse($user->validate(['username']));

        $user->username = 'davert';
        $t->assertTrue($user->validate(['username']));

        $t->seeInDatabase('users', ['name' => 'Miles', 'surname' => 'Davis']);
    }
}
?>
```

For unit testing you may include `Asserts` module, that adds regular assertions to UnitTester which you may access from `$t` variable.

```yaml
# Codeception Test Suite Configuration

# suite for unit (internal) tests.
class_name: UnitTester
modules:
    enabled: [Asserts, Db, UnitHelper]
```

[Learn more about Cest format](http://codeception.com/docs/07-AdvancedUsage#Cest-Classes).

### Stubs

Codeception provides a tiny wrapper over PHPUnit mocking framework to create stubs easily. Include `\Codeception\Util\Stub` to start creating dummy objects.

In this example we instantiate object without calling a constructor and replace `getName` method to return value *john*.

```php
<?php
$user = Stub::make('User', ['getName' => 'john']);
$name = $user->getName(); // 'john'
?>
```

Stubs are created with PHPUnit's mocking framework. Alternatively you can use [Mockery](https://github.com/padraic/mockery) (with [Mockery module](https://github.com/Codeception/MockeryModule)), [AspectMock](https://github.com/Codeception/AspectMock) or others.

Full reference on Stub util class can be found [here](/docs/reference/Stub).

## Conclusion

PHPUnit tests are first-class citizens in test suites. Whenever you need to write and execute unit tests, you don't need to install PHPUnit, but use Codeception to execute them. Some nice features can be added to common unit tests by integrating Codeception modules. For the most of unit and integration testing PHPUnit tests are just enough. They are fast and easy to maintain.
