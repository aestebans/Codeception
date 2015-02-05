
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

Usando los bloques de código `specify` se puede describir una trozo de código de prueba. Esto hace las pruebas mucho mas claras y entendibles para cada uno de los elementos del equipo.

El código dentro de cada bloque `specify`está aislado. En el ejemplo anterior cualquier cambio en `$this-> User` (como cualquier otra propiedad del objeto), no se verá reflejada en otros bloques de código.

También se puede agregar [Codeception\Verify](https://github.com/Codeception/Verify) para las confirmaciones de tipo BDD. Esta pequeña librería añade confirmaciones más legibles, que es bastante agradable, si se confunde con el argumento en el `assert` que espera llamadas y cuál es actual.


```php
<?php
verify($user->getName())->equals('john');
?>
```

## Usando Módulos

Al igual que en el escenario impulsado por pruebas funcionales o de aceptación se puede acceder a métodos de la clase Actor. Para escribir las pruebas de integración, puede ser útil incluir módulo `db` para las pruebas de base de datos.


```yaml
# Codeception Test Suite Configuration

# suite for unit (internal) tests.
class_name: UnitTester
modules:
    enabled: [Db, UnitHelper]
```

Para acceder a los métodos UnitTester puede usar `propiedad UnitTester` en una prueba.

### Probando la base de datos

Veamos como se puede hacer realizar pruebas a una base de datos:


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

La base de datos se limpiará y se llenará después de cada prueba, en el caso de pruebas de aceptación y pruebas funcionales. Y si no tiene el comportamiento requerido, cambiar la configuración de módulo `db` para el grupo de pruebas actual. 


### Módulo de Acceso 

Codeception le permite acceder a propiedades y métodos de todos los módulos definidos en el grupo de pruebas. A diferencia de uso de la clase UnitTester para este fin, utilizando el módulo donaciones directamente el acceso a todas las propiedades públicas de ese módulo.

Por ejemplo, si se utiliza el módulo `Symfony2`, esta es la manera para acceder contenedor Symfony:


```php
<?php
/**
 * @var Symfony\Component\DependencyInjection\Container
 */
$container = $this->getModule('Symfony2')->container;
?>
```

Todas las variables públicas están en la relación de referencias de los correspondientes módulos.

### Cest

Como alternativa para casos de prueba se extendió desde `PHPUnit_Framework_TestCase` se puede usar el espécifico formato Cest de Codeception. No requiere que se extienda a cualquier otra clase. Todos los métodos públicos de esta clase son pruebas. 

El ejemplo anterior se puede reescribir de manera escenario impulsado por la siguiente:


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
Para la prueba unitaria se puede incluir el módulo `Asserts`, que añade confirmaciones regulares para la clase UnitTester a los que puede acceder desde la variable `$t`.


```yaml
# Codeception Test Suite Configuration

# suite for unit (internal) tests.
class_name: UnitTester
modules:
    enabled: [Asserts, Db, UnitHelper]
```

[Aprender mas sobre el formato Cest format](http://codeception.com/docs/07-AdvancedUsage#Cest-Classes).

### Stubs

Codeception ofrece un pequeño envoltorio sobre PHPUnit burlándose marco para crear stubs fácilmente. Incluir `\Codeception\util\Stub` para iniciar la creación de objetos ficticios. 

En este ejemplo, se crea una instancia del objeto sin llamar a un constructor y reemplazamos método `getName` para devolver el valorvolver valor *john*.


```php
<?php
$user = Stub::make('User', ['getName' => 'john']);
$name = $user->getName(); // 'john'
?>
```
Stubs se crean con PHPUnit burlaándose de framework. Alternativamente, puede utilizar
[Mockery](https://github.com/padraic/mockery) (with [Mockery module](https://github.com/Codeception/MockeryModule)), [AspectMock](https://github.com/Codeception/AspectMock) u otros.

Una completa referencia de la util clase Stub se puede encontrar en [here](/docs/reference/Stub).

## Conclusion

Las pruebas PHPUnit son ciudadanos de primera clase en los bancos de pruebas. Siempre que necesite para escribir y ejecutar pruebas unitarias, no es necesario instalar PHPUnit, pero utilice Codeception para ejecutarlos. Algunas características agradables se pueden añadir a las pruebas unitarias comunes mediante la integración de módulos Codeception. Para la mayor parte de las pruebas PHPUnit unitarias y pruebas de integración son lo suficiente. Los módulos son rápidos y fáciles de mantener.

