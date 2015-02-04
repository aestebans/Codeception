# Introdución

La prueba en el origen, no es una idea nueva. No se puede dormir tranquilo, no sabiendo si los cambios realizados afectarán al conjunto de la aplicación
Tener la aplicación totalmente probada, da confianza en la estabilidad de la aplicación.

En la mayoría de los casos, las pruebas no garantizan que la aplicación funcione al 100%, eso solo una suposición. No se pueden predecir todos los posibles escenarios y situaciones excepcionales en las aplicaicones complejas, pero con las pruebas se pueden probar las partes mas importantes y asegurar que hacen lo previsto

Hay un montón de maneras de probar una aplicación. La mas popular es [Pruebas unitarias](http://es.wikipedia.org/wiki/Prueba_unitaria). Para aplicaciones web, probar las clases controladores o del modelo, no significa que la aplicación funcione. Para probar la aplicación en su conjunto es necesario escribir pruebas funcionales o de aceptación

El framework Codeception distingue los diferentes niveles de pruebas. 

Vamos a revisar los paradigmas de pruebas enumerados en el orden inverso.

### Test de aceptación

¿Como pueden saber, el cliente, gerente, probador o cualquier persona no técnica que el sitio web funciona correctamente?
Solamente, con el navegador, accediendo al sitio, pulsando en los enlaces, rellenando formularios y comprobar que ve el resultado esperado. Esta persona no tiene ni idea de framework, base de datos, servidor web, o del lenguaje de programamción usado o porque la aplicación no hizo lo esperado.

Las pruebas de acepctación pueden cubrir escenarios estandar complejos desde la perspectiva del usuario. Con las pruebas de aceptación se puede estar seguro de que los usuarios, en base a los escenarios definidos, no obtendrán errores.

hay que tener en cuenta que **cualquier sitio web** puede estar cubierto por pruebas de aceptación, independiente de que sea un CMS o un framework.

#### Ejemplo de test de aceptación

```php
<?php
$I = new AcceptanceTester($scenario);
$I->amOnPage('/');
$I->click('Sign Up');
$I->submitForm('#signup', array('username' => 'MilesDavis', 'email' => 'miles@davis.com'));
$I->see('Thank you for Signing Up!');
?>
```

#### Pros

* puede ejecutarse sobre cualquier sitio web
* puede realizar pruebas javascript y peticiones ajax
* se puede presentar a los clientes y gerentes la ejecución y resultados
* hace mas estable el mantenimiento: los cambios en el código fuente o en la tecnología se ven menos afectados

#### Contras
* lentitud: necesita ejecutarse en un navegador y rellenar las tablas
* con pocos controles pueden dar a resultados dudosos
* si, las pruebas son lentas
* la ejecución no es estable: la presentación y los script de javascript pueden ocasionar resultados impredecibles


### Test funcionales

¿Y si pudiesemos probar nuestra aplicación sin ejecutarlo en un servidor? De esta manera podemos ver excepciones detalladas sobre los errores, las pruebas son mas rápidas y podemos chequear la base de datos con los valores esperados. Para esto son las pruebas funcionales.

En las pruebas funcionales, se emulas las peticiones a la web (variables `$_GET` y `$_POST`) y recibir la respuesta HTML. Dentro de una prueba se pueden hacer confirmaciones y se puede comprobar si los datos se almacenan correctamente en la base de datos.

Para las pruebas funcionales, su aplicación tiene que estar preparada para ser ejecutada en un entorno de pruebas. Codeception tiene conectores para los frameworks PHP mas populares, o se puede escribir el suyo propio.

#### Ejemplo de test funcional

```php
<?php
$I = new FunctionalTester($scenario);
$I->amOnPage('/');
$I->click('Sign Up');
$I->submitForm('#signup', array('username' => 'MilesDavis', 'email' => 'miles@davis.com'));
$I->see('Thank you for Signing Up!');
$I->seeEmailSent('miles@davis.com', 'Thank you for registration');
$I->seeInDatabase('users', array('email' => 'miles@davis.com'));
?>
```

#### Pros

* Como las pruebas de aceptación, pero mucho mas rápido
* Proporciona informes mas detallados
* Se puede mostrar el código a los directivos y clientes
* Suficiente estable: solo grandes cambios, o el traslado a otro framework podrían alterarlo.

#### Contras

* javascript y ajax no pueden ser probados
* Al emular el navegador puede inducir a falsos resultados
* Necesita un framework

### Test unitarios

Probar trozos de código antes de integrarlos en la aplicación es muy importante. De esta manera aspectos que no pudieron ser probados por las pruebas funcionales o de aceptación, se comprueban. Esto permite demostrar que el código es estable y comprobable.

Codeception está creado como una capa superior de [PHPUnit](http://www.phpunit.de/). Si se tiene experiencia escribiendo pruebas en PHPUnit, se puede seguir hacieéndose. Codeception no tiene problemas para ejecutar pruebas estandar de PHPUnit, pero Codeception tiene herramientas para hacer de forma mas simple y clara las pruebas.

Los desarrolladores sin experiencia, tienen que entender los que se tiene que probar y como. Requisitos y código cambía muy a menudo y las pruebas unitarios deben de actualizarse para adpatarse a las necesidades. Un mayor conocimiento del escenario de las pruebas permite de una forma mas rápida, actualizar las pruebas para el cambio.

#### Ejemplo de test de integración

```php
<?php
function testSavingUser()
{
    $user = new User();
    $user->setName('Miles');
    $user->setSurname('Davis');
    $user->save();
    $this->assertEquals('Miles Davis', $user->getFullName());
    $this->unitTester->seeInDatabase('users',array('name' => 'Miles', 'surname' => 'Davis'));
}
?>
```

#### Pros

* rápido, pero es necesario rellenar las tablas
* cubre las funciones que raramente se usan
* permite probar la estabilidad del nucleo de la aplicación
* es necesario escribir pruebas de test para certificar que es un buen programador :)

#### Contras

* no prueba la conexión entre unidades
* no es muy estable para mantenimiento: es muy sensible a los cambios de código

## Conclusión

A pesar de la popularidad de TDD, no todos los desarrolladores escriben test para automatizar las pruebas de sus aplicaciones. El framework Codeception se ha desarrollado para hacer que las pruebas sean agradables. Permite escribir pruebas unitarias, funcionales, de integración y de aceptación de la misma manera.

Podría decirse que es un framework BDD. Todas las pruebas en Codeception se escriben de forma descriptiva. Con sólo mirar el texto de prueba se puede entender claramente lo que se está probando y cómo se realiza. Incluso las pruebas complejas, con muchas afirmaciones, se escriben de una forma sencilla DSL PHP.
