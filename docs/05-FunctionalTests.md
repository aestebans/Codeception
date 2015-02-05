# Pruebas Funcionales 

Ahora que hemos escrito algunas pruebas de aceptación, las pruebas funcionales son casi lo mismo, con una sola diferencia importante: las pruebas funcionales no requieren un servidor web para ejecutar las pruebas. 

Establecemos las variables `$ _REQUEST`,` $ _GET` y `$ _POST` y luego ejecutamos la aplicación de una prueba. Esto es muy valorable porque las pruebas funcionales son más rápidas y ofrecen trazas detalladas en los fallos.

Codeception puede conectarse a diferentes frameworks que soportan pruebas funcionales: Symfony2, Laravel4, Yii2, Zend Framework y otros. Sólo tiene que activar el módulo deseado en la configuración para comenzar. 

Los módulos para todos estos frameworks comparten la misma interfaz, y por lo tanto sus pruebas no son específicas de ninguno de ellos. 

Esta es un ejemplo de prueba funcional.



```php
<?php
$I = new FunctionalTester($scenario);
$I->amOnPage('/');
$I->click('Login');
$I->fillField('Username', 'Miles');
$I->fillField('Password', 'Davis');
$I->click('Enter');
$I->see('Hello, Miles', 'h1');
// $I->seeEmailIsSent() - special for Symfony2
?>
```

Como se puede ver se puede usar mismas pruebas para pruebas funcionales y de aceptación.

## Trampas

Las pruebas de aceptación son generalmente mucho más lentas que las pruebas funcionales. Pero las pruebas funcionales son menos estables, ya que corren Codeception y aplicación en un mismo entorno.

#### Headers, Cookies, Sessions

Uno de los problemas comunes con las pruebas funcionales es el uso de las funciones de PHP que tienen que ver con `headers`,` sessions`, `cookies`. 
La función `header` desencadena un error si se ejecuta más de una vez por el mismo encabezado. En las pruebas funcionales corremos aplicaciones múltiples veces, por lo tanto, vamos a tener un montón de errores de basura en el resultado.

#### Memoria Compartida 

En las pruebas funcionales a diferencia de la forma tradicional, aplicaciones PHP no se detiene después de que terminó de procesar una solicitud. 

Como todas las solicitudes se ejecutan en un contenedor de memoria no son aislados. 

Así ** si usted ve que sus pruebas estan fallando misteriosamente cuando no deberían. - Hay que intentar ejecutar una sola prueba ** Esto comprobará si se aislaron las pruebas durante la marcha. Debido a que es muy fácil de estropear el entorno ya que todas las pruebas se practican en la memoria compartida. Mantenga su memoria limpia, evitar pérdidas de memoria y las variables globales y estáticas limpias.


## Habilitando módulos de frameworks

Tenemos un grupo de pruebas funcionales en el directorio `tests/functional`. Para empezar es necesario incluir uno de los módulos del framework en el archivo de configuración del grupo: `tests/functional.suite.yml`. A continuación se proporcionan instrucciones simplificadas para la creación de pruebas funcionales con los frameworks PHP más populares.


### Symfony2

Para llevar a cabo integraciones Symfony2 no es necesario instalar nada. Sólo tiene se debe incluir el módulo `Symfony2` en el grupo de pruebas. Si también se utiliza Doctrine2, hay que incluir éste también.


Ejemplo de `functional.suite.yml`

```yaml
class_name: FunctionalTester
modules:
    enabled: [Symfony2, Doctrine2, TestHelper] 
```

Por defecto, este módulo buscará App kernel en el directorio `app`. El módulo utiliza el perfil Symfony para proporcionar información y confirmaciones adicionales.

[Referencias completas](http://codeception.com/docs/modules/Symfony2)

### Laravel 4

[Laravel](http://codeception.com/docs/modules/Laravel4) el módulo no requiere configuración y se pone en marcha facilmente.

```yaml
class_name: FunctionalTester
modules:
    enabled: [Laravel4, TestHelper]
```


### Yii2

Yii2 pruebas se incluyen en [Básico] (https://github.com/yiisoft/yii2-app-basic) y [Advanced] (https://github.com/yiisoft/yii2-app-advanced) plantillas de aplicación. Siga Yii2 guía para empezar.

Plantillas de pruebas Yii2 estan incluidas en [Basic](https://github.com/yiisoft/yii2-app-basic) and [Advanced](https://github.com/yiisoft/yii2-app-advanced). Ver las guías Yii2 para empezar.

### Yii

El framework Yii no tiene un motor de pruebas funcionales. Es por ello que Codeception es el primero y el único framework de pruebas funcionales para Yii. Para utilizarlo con Yii incluye el módulo `Yii1` en config.


```yaml
class_name: FunctionalTester
modules:
    enabled: [Yii1, TestHelper]
```

Para evitar los errores comunes, Codeception proporciona enlaces básicos sobre motor Yii. Por favor configurarlas siguiendo la guía [instalación paso a paso, referencia del módulo](http://codeception.com/docs/modules/Yii1).

### Zend Framework 2

Usar el módulo [ZF2](http://codeception.com/docs/modules/ZF2) para ejecutar pruebas funcionales con el framework Zend Framework 2.

```yaml
class_name: FunctionalTester
modules:
    enabled: [ZF2, TestHelper]
```

### Zend Framework 1.x

El módulo de Zend Framework está muy inspirado por la clase ControllerTestCase, utilizado para pruebas funcionales con PHPUnit. 
De ello se desprende enfoques similares para arranque y la limpieza. Para empezar a utilizar Zend Framework en sus pruebas funcionales, incluir el módulo `ZF1 `.

Ejemplo de `functional.suite.yml`

```yaml
class_name: FunctionalTester
modules:
    enabled: [ZF1, TestHelper] 
```

[See the full reference](http://codeception.com/docs/modules/ZF1)

### Phalcon 1.x

El módulo `Phalcon1` requiere la creación de archivo de arranque que devuelve instancia de `\Phalcon\Mvc\A pplication`. Para comenzar a escribir pruebas funcionales con apoyo Phalcon debe habilitar módulo `Phalcon1` y proporcionar ruta del archivo de arranque:

```yaml
class_name: FunctionalTester
modules:
    enabled: [Phalcon1, FunctionalHelper]
    config:
        Phalcon1
            bootstrap: 'app/config/bootstrap.php'
```

[Guía de referencia](http://codeception.com/docs/modules/Phalcon1)

## Escribiendo pruebas funcionales

Las pruebas funcionales se escriben de la misma manera que las pruebas de Aceptación con el módulo `PhpBrowser` habilitado [Pruebas de Aceptación](http://codeception.com/docs/04-AcceptanceTests). Todos los módulos de framework y el módulo `PhpBrowser` comparten los mismos métodos y el mismo motor.

Por lo tanto podemos abrir una página web con el comando `amOnPage`.

```php
<?php
$I = new FunctionalTester;
$I->amOnPage('/login');
?>
```

Podemos hacer clic en vínculos para abrir páginas web de la aplicación.

```php
<?php
$I->click('Logout');
// click link inside .nav element
$I->click('Logout', '.nav');
// click by CSS
$I->click('a.logout');
// click with strict locator
$I->click(['class' => 'logout']);
?>
```

Podemos enviar formularios, así:


```php
<?php
$I->submitForm('form#login', ['name' => 'john', 'password' => '123456']);
// alternatively
$I->fillField('#login input[name=name]', 'john');
$I->fillField('#login input[name=password]', '123456');
$I->click('Submit', '#login');
?>
```

Y hacer confirmaciones:

```php
<?php
$I->see('Welcome, john');
$I->see('Logged in successfully', '.notice');
$I->seeCurrentUrlEquals('/profile/john');
?>
```

Los módulos de Framework también contienen métodos adicionales para accesos internos a los frameworks. Por ejemplo, los módulos `Laravel4`,` Phalcon1`, y `Yii2` tienen `seeRecord` que utiliza la capa ActiveRecord para comprobar que ese registro existe en la base de datos. 

El módulo `Laravel4` también contiene métodos adicionales para hacer comprobaciones de sesión. Se puede encontrar `seeSessionHasErrors` cuando se prueba validaciones de formularios. 

Hay una referencia completa para el módulo que se está utilizando. La mayoría de sus métodos son comunes para todos los módulos, pero algunos de ellos son únicos.

También se puede acceder a variables globales del framework dentro de una prueba o acceso a los contenedores de Inywcción de Dependencias dentro de la clase `FunctionalHelper`.

```php
<?php
class FunctionalHelper extends \Codeception\Module
{
    function doSomethingWithMyService()
    {
        $service = $this->getModule('Symfony2') // lookup for Symfony 2 module
            ->container // get current DI container
            ->get('my_service'); // access a service

        $service->doSomething();
    }
}
?>
```

Nosotros accedemos al nucleo interno de Symfony2 y tomamos un servicio de contenedor. También hemos creado un método personalizado en la clase `FunctionalTester` para poder ser utilizado en las pruebas. 

Se puede aprender más acerca de cómo acceder al framework que se utiliza chekeando en la sección *Public Properties* en la referencia del módulo correspondiente.


## Informes de error

Por defecto el niverl de informes en Codeception es `E_ALL & ~E_STRICT & ~E_DEPRECATED`. 
En las pruebas funcionales es posible que desee cambiar este nivel, según la política error de framework.
El nivel de informes se pueede poner en el fichero de configuración del grupo de pruebas:

```yaml
class_name: FunctionalTester
modules:
    enabled: [Yii1, TestHelper]
error_level: "E_ALL & ~E_STRICT & ~E_DEPRECATED"
```

`error_level` puede ponerse de forma global en el fichero `codeception.yml`.


## Conclusión

Las pruebas funcionales son grandes si se está utilizando toda la potencia de los frameworks. Mediante el uso de las pruebas de funcionamiento se puede acceder y manipular su estado interno. 
Esto hace que sus pruebas sean más cortas y más rápidas. En otros casos, si no se utiliza frameworks no hay ninguna razón práctica para escribir pruebas funcionales. 
Si se está utilizando un framework distinto de los enumerados aquí, cree un módulo para ello y compartirlo con la comunidad.
