# Pruebas de Aceptación

Las pruebas de aceptación puede ser realizadas por una persona no técnica. Esa persona puede ser su probador, gerente o incluso clientes. 
Si está desarrollando una aplicación web (y probablemente lo eres) el probador no necesita nada más que un navegador web para comprobar que su sitio funcione correctamente. Puede reproducir las acciones AcceptanceTester en escenarios y ejecutarlos automáticamente después de cada cambio de sitio. Codeception mantiene pruebas limpio y sencillo, como si se hubiesen registrado a partir de las palabras de AcceptanceTester. 
No hace ninguna diferencia si el sitio utiliza un CMS o un framework. Se puede probar incluso sitios creados en diferentes plataformas, como Java, .NET, etc. Siempre es una buena idea añadir pruebas a su sitio web. Esto nos asegura que las funcionalidades se mantienen después de realizar los últimos cambios. 

## Escenario de ejemplo 

Probablemente, la primera prueba que se desea ejecutar sería iniciar sesión. Par poder escribir una prueba de este tipo, todavía se requieren conocimientos básicos de PHP y HTML.


```php
<?php
$I = new AcceptanceTester($scenario);
$I->wantTo('sign in');
$I->amOnPage('/login');
$I->fillField('username', 'davert');
$I->fillField('password', 'qwerty');
$I->click('LOGIN');
$I->see('Welcome, Davert!');
?>
```

Este escenario probablemente puede ser leído por personas sin conocimientos técnicos. Codeception puede incluso "naturalizar" esta situación, convirtiéndola texto plano Inglés:

```bash
I WANT TO SIGN IN
I am on page '/login'
I fill field 'username', 'davert'
I fill field 'password', 'qwerty'
I click 'LOGIN'
I see 'Welcome, Davert!'
```

Estas transformaciones se pueden realizar por el comando:

``` bash
$ php codecept.phar generate:scenarios
```

Los escenarios generados se guardan en el directorio ___data__ en ficheros de texto.

** Este escenario se puede realizar ya sea mediante un simple navegador PHP o por un navegador con Selenium WebDriver **. Vamos a empezar a escribir nuestras primeras pruebas de aceptación con un PhpBrowser.

## Navegador PHP

Esta es la forma más rápida de ejecutar las pruebas de aceptación, ya que no requiere la ejecución de un navegador real. Utilizamos un capturador web en PHP, que actúa como un navegador: envía una solicitud, entonces recibe y analiza la respuesta. Codeception usa [Guzzle] (http://guzzlephp.org) y Symfony BrowserKit para interactuar con páginas Web HTML. Tenga en cuenta que no se puede probar la visibilidad real de elementos, o interacciones javascript. Lo bueno de PhpBrowser es que se puede ejecutar en cualquier entorno, solo es necesario PHP y CURL.

Inconvenientes comunes de PhpBrowser: 

* Puede hacer clic sólo en los enlaces con URL válidos o botones de envío de un formulario 
* No se puede rellenar los campos que no están dentro de un formulario 
* No se puede trabajar con las interacciones de JavaScript: ventanas modales, datepickers, etc. 
 
Antes de empezar, necesitamos una copia local del sitio que se ejecuta en el host. Tenemos que especificar el parámetro `url` en la configuración de grupo de aceptación (tests/acceptance.suite.yml).

``` yaml
class_name: AcceptanceTester
modules:
    enabled:
        - PhpBrowser
        - AcceptanceHelper
        - Db
    config:
        PhpBrowser:
            url: [your site's url]
```

Deberíamos empezar por crear un archivo 'Cept' en el directorio __tests/acceptance__ . Llamémoslo __SigninCept.php__. Vamos a escribir las primeras líneas en él.

```php
<?php
$I = new AcceptanceTester($scenario);
$I->wantTo('sign in with valid account');
?>
```
La sección `wantTo` describe el escenario de una manera breve. Existen métodos adicionales de comentarios que son útiles, para hacer un escenario Codeception en una historia BDD. Si alguna vez has escrito un escenario BDD en Gherkin, puede escribir una historia de usuario clasica

```bash
As an Account Holder
I want to withdraw cash from an ATM
So that I can get money when the bank is closed
```

en formato Codeception:

```php
<?php
$I = new AcceptanceTester($scenario);
$I->am('Account Holder'); 
$I->wantTo('withdraw cash from an ATM');
$I->lookForwardTo('get money when the bank is closed');
?>
```

Después de haber descrito la historia de fondo, vamos a empezar a escribir un escenario.

El objeto `$I` se utiliza para escribir todas las interacciones. Los métodos del objeto `$I` se toman de los modulos `db` y `PhpBrowser`. Vamos a describirlo brevemente aquí:

```php
<?php
$I->amOnPage('/login');
?>
```
Asumimos que todos los comandos `am` deben describir el entorno de partida. El comando `amOnPage` establece el punto de partida de una prueba para la página __ / login__. 

Con los `PhpBrowser` se puede hacer clic en los enlaces y llenar los formularios. Eso serán probablemente la mayoría de las acciones.


#### Click 

Un clic emula pinchar sobre enlaces válidos. La página del parámetro "href" se abrirá. Como parámetro se puede especificar el nombre de enlace o un CSS válido o selector XPath.


```php
<?php
$I->click('Log in'); 
// CSS selector applied
$I->click('#login a');
// XPath
$I->click('//a[@id=login]');
// Using context as second argument
$I->click('Login', '.nav');
?>
```

Codeception intenta localizar el elemento, ya sea por su texto, nombre, CSS o XPath. Puede especificar el tipo de localización manualmente pasando array como parámetro. A esto le llamamos localizador estricto **strict locator**. Los tipos disponibles de localización estrictos son:

* id
* name
* css
* xpath
* link
* class

```php
<?php
// By specifying locator type
$I->click(['link' => 'Login']);
$I->click(['class' => 'btn']);
?>
```
Antes de hacer clic en el enlace se puede realizar una comprobación de si el enlace de una página existe. Esto se puede hacer por la acción `seeLink`.

```php
<?php
// checking that link actually exists
$I->seeLink('Login');
$I->seeLink('Login','/login');
$I->seeLink('#login a','/login');
?>
```

#### Formularios

Hacer clic en los enlaces no es lo que toma más tiempo durante las pruebas de un sitio web. Si el sitio se compone sólo de enlaces puede omitir la automatización de pruebas.
Lo que lleva mas tiempo son las pruebas de formularios. Codeception proporciona varias maneras de hacerlo. 

Vamos a enviar este formulario de muestra, dentro de la prueba Codeception.


```html
<form method="post" action="/update" id="update_form">
     <label for="user_name">Name</label>
     <input type="text" name="user[name]" id="user_name" />
     <label for="user_email">Email</label>
     <input type="text" name="user[email]" id="user_email" />     
     <label for="user_gender">Gender</label>
     <select id="user_gender" name="user[gender]">
          <option value="m">Male</option>
          <option value="f">Female</option>
     </select>     
     <input type="submit" name="submitButton" value="Update" />
</form>
```

Desde la perpectiva del usuario , un formulario consiste en un conjunto de campos que tienen que rellenarse y que se envían cuando pincho en el boton Update.

```php
<?php
// we are using label to match user_name field
$I->fillField('Name', 'Miles');
// we can use input name or id
$I->fillField('user[email]','miles@davis.com');
$I->selectOption('Gender','Male');
$I->click('Update');
?>
```
Para hacer coincidir los campos por sus etiquetas, debe escribir un atributo `for` en la etiqueta. 

Desde la perspectiva del desarrollador, la presentación de un formulario se acaba enviando una solicitud válida al servidor. Muchas veces es más fácil llenar todos los campos a la vez y enviar el formulario sin hacer clic en un botón 'Enviar'. 

Un escenario similar se puede reescribir con un solo comando.


```php
<?php
$I->submitForm('#update_form', array('user' => array(
     'name' => 'Miles',
     'email' => 'Davis',
     'gender' => 'm'
)));
?>
```
El `submitForm` no está emulando las acciones del usuario, pero es muy útil en situaciones en las que el formulario no tiene el formato adecuado, por ejemplo, descubrir que las etiquetas no están o que los campos tienen nombres mal formados o los ids mal escritos, o el formulario es enviado por una llamada javascript. 

Por defecto, submitForm no envía los valores para los botones. El último parámetro permite especificar qué valores botón debe enviarse, o los valores a enviar se pueden especificar de manera implícita en un segundo parámetro.


```php
<?php
$I->submitForm('#update_form', array('user' => array(
     'name' => 'Miles',
     'email' => 'Davis',
     'gender' => 'm'
)), 'submitButton');
// this would be the same effect, but the value has to be implicitly specified
$I->submitForm('#update_form', array('user' => array(
     'name' => 'Miles',
     'email' => 'Davis',
     'gender' => 'm',
	 'submitButton' => 'Update'
)));
?>
```

#### Emulación AJAX 

Como sabemos, el navegador PHP no puede procesar JavaScript. Aún así, todas las llamadas ajax pueden ser fácilmente emuladas mediante el envío de las solicitudes apropiadas al servidor. Considere el uso de estos métodos para las interacciones Ajax.

```php
<?php
$I->sendAjaxGetRequest('/refresh');
$I->sendAjaxPostRequest('/update', array('name' => 'Miles', 'email' => 'Davis'));
?>
```

#### Las confirmaciones

En el navegador PHP se puede probar la página de contenidos. En la mayoría de los casos sólo tiene que comprobar que el texto o elemento necesario está en la página. 

El comando más útil para esto es `see`.

```php
<?php
// We check that 'Thank you, Miles' is on page.
$I->see('Thank you, Miles');
// We check that 'Thank you Miles' is inside 
// the element with 'notice' class.
$I->see('Thank you, Miles', '.notice');
// Or using XPath locators
$I->see('Thank you, Miles', "descendant-or-self::*[contains(concat(' ', normalize-space(@class), ' '), ' notice ')]");
// We check this message is not on page.
$I->dontSee('Form is filled incorrectly');
?>
```

Se puece chequear que un elemento específico existe (o no) en una página

```php
<?php
$I->seeElement('.notice');
$I->dontSeeElement('.error');
?>
```

También tenemos otros comandos útiles para realizar comprobaciones. Tenga en cuenta que todos ellos comienzan con el prefijo `see`.


```php
<?php
$I->seeInCurrentUrl('/user/miles');
$I->seeCheckboxIsChecked('#agree');
$I->seeInField('user[name]', 'Miles');
$I->seeLink('Login');
?>
```

#### Las confirmaciones condicionales

A veces no se quiere que la prueba se detenga cuando falla una confirmación, cuando se tiene una prueba de larga duración y se desea que se ejecute hasta el final. En este caso se puede usar confirmaciones condicionales. Cada método `see` su correspondiente metodo `canSee`, y el método `dontSee` tiene un método` cantSee`.


```php
<?php
$I->canSeeInCurrentUrl('/user/miles');
$I->canSeeCheckboxIsChecked('#agree');
$I->cantSeeInField('user[name]', 'Miles');
?>
```

Cada confirmación fallda se mostrará en los resultados de pruebas. Aún así, no detendrá la prueba.

#### Capturadores 

Estos comandos capturan datos que se pueden reutilizar en la prueba. Imagínese, su sitio genera una contraseña para cada usuario y quiere comprobar que el usuario puede iniciar sesión en el sitio utilizando esta contraseña.


```php
<?php
$I->fillField('email', 'miles@davis.com')
$I->click('Generate Password');
$password = $I->grabTextFrom('#password');
$I->click('Login');
$I->fillField('email', 'miles@davis.com');
$I->fillField('password', $password);
$I->click('Log in!');
?>
```

Los capturadores permiten le permiten obtener un valor único de la página actual con los comandos.


```php
<?php
$token = $I->grabTextFrom('.token');
$password = $I->grabTextFrom("descendant::input/descendant::*[@id = 'password']");
$api_key = $I->grabValueFrom('input[name=api]');
?>
```

#### Comentarios

Dentro de un escenario grande, se debe describir las acciones que van a realizar y qué resultados se esperan alcanzar. Comandos como `amGoingTo`, `expect`, `expectTo` permiten hacer las pruebas mas descriptivas

```php
<?php
$I->amGoingTo('submit user form with invalid values');
$I->fillField('user[email]', 'miles');
$I->click('Update');
$I->expect('the form is not submitted');
$I->see('Form is filled incorrectly');
?>
```

#### Cookies, Urls, Title, etc

Acciones para cookies:

```php
<?php
$I->setCookie('auth', '123345');
$I->grabCookie('auth');
$I->seeCookie('auth');
?>
```

Acciones para chequear el título de una página:

```php
<?php
$I->seeInTitle('Login');
$I->dontSeeInTitle('Register');
?>
```

Acciones para la url:

```php
<?php
$I->seeCurrentUrlEquals('/login');
$I->seeCurrentUrlMatches('~$/users/(\d+)~');
$I->seeInCurrentUrl('user/1');
$user_id = $I->grabFromCurrentUrl('~$/user/(\d+)/~');
?>
```

## Selenium WebDriver

Una característica interesante de Codeception es que la mayoría de los escenarios se pueden trasladar fácilmente entre los backends de prueba. 
Sus pruebas PhpBrowser que escribimos anteriormente pueden ejecutarse dentro de un navegador real (o incluso PhantomJS) con Selenium WebDriver. 

Lo único que tenemos que cambiar es reconfigurar y reconstruir la clase AcceptanceTester, utilizar **WebDriver ** en lugar de PhpBrowser.


Modificque el fichero `acceptance.suite.yml`:

```yaml
class_name: AcceptanceTester
modules:
    enabled:
        - WebDriver
        - AcceptanceHelper
    config:
        WebDriver:
            url: 'http://localhost/myapp/'
            browser: firefox            
```

Con el fin de ejecutar las pruebas de Selenium se necesita  [download Selenium Server](http://seleniumhq.org/download/) y ponerlo en marcha [PhantomJS](http://phantomjs.org/) en modo `ghostdriver`). 

Si ejecuta pruebas de aceptación con Selenium, Firefox se iniciará y todas las acciones se realizará paso a paso utilizando el motor de navegador. 

En este caso `seeElement` no sólo comprueba que existe el elemento en una página, sino que también comprobará que elemento es realmente visible al usuario.


```php
<?php 
$I->seeElement('#modal'); 
?>
```


#### Esperar

Al probar la aplicación web, es posible que tenga que esperar a que ocurran los eventos de JavaScript. Debido a su naturaleza asincrónica, las interacciones complejas de JavaScript son difíciles de probar. Es por eso que usted puede necesitar usar acciones `wait`, que pueden ser utilizados para especificar qué evento se espera que ocurra en una página, antes de continuar la prueba.

Por ejemplo: 

```php
<?php
$I->waitForElement('#agree_button', 30); // secs
$I->click('#agree_button');
?>
```

En este caso estamos esperando a que aparezca el agree_button botón para luego hacer clic en él. Si no aparece durante 30 segundos, la prueba fallará. 

Hay otros métodos de `wait` que se pueden utilizar. Ver [WebDriver module documentation](http://codeception.com/docs/modules/WebDriver) en la documentación de Codeception con la referencia completa. 

### Pruebas Multi-sessión

Codeception le permite ejecutar acciones en sesiones simultáneas. El caso más evidente de que está poniendo a prueba la mensajería en tiempo real entre los usuarios en el sitio. Para ello, habrá que poner en marcha dos ventanas del navegador al mismo tiempo para la misma prueba. Codeception tiene concepto muy inteligente para hacer esto. Se llama Amigos **Friends**.


```php
<?php
$I = new AcceptanceTester($scenario);
$I->wantTo('try multi session');
$I->amOnPage('/messages');
$nick = $I->haveFriend('nick');
$nick->does(function(AcceptanceTester $I) {
    $I->amOnPage('/messages/new');
    $I->fillField('body', 'Hello all!')
    $I->click('Send');
    $I->see('Hello all!', '.message');
});
$I->wait(3);
$I->see('Hello all!', '.message');
?>
```

En este caso hemos hecho algunas acciones en segunda ventana con el comando `does` en un objeto Friend. 

### Limpieza de datos 

Durante la prueba, sus acciones pueden cambiar los datos en el sitio. Las pruebas fallarán si tratara de crear o actualizar los mismos datos dos veces. Para evitar este problema, la base de datos debe ser repoblada por cada prueba. Codeception proporciona un módulo `db` para ese propósito. Se carga un volcado de base de datos después de cada prueba superada. Para hacer el trabajo de repoblación, crear un volcado sql de su base de datos y ponerlo en el directorio __/tests/_data__. Establezca la conexión de base de datos y ruta de acceso al volcado en el fichero de configuración  global de Codeception.

In this case we did some actions in second window with `does` command on a friend object.


```yaml
# in codeception.yml:
modules:
    config:
        Db:
            dsn: '[set pdo dsn here]'
            user: '[set user]'
            password: '[set password]'
            dump: tests/_data/dump.sql
```

### Debugging

Los módulos de Codeception pueden mostrar información mientras se está ejecutando. Se debe ejecutar los tests con la opción `--debug` para poder ver detalles de la ejecución. Para cualquier tipo personalizado de salida use la función `codecept_debug`.


```php
<?php
codecept_debug($I->grabTextFrom('#name'));
?>
```


En cada fracaso, la instantánea de la última página que se muestra, se almacenará en el directorio __tests/_log__. PhpBrowser almacenará código HTML y WebDriver guardará la captura de pantalla de la página. 

## Conclusión 

Escribir pruebas de aceptación con Codeception y PhpBrowser es un buen comienzo. Se puede probar fácilmente sus sitios Joomla, Drupal, WordPress, así como los realizados con los frameworks. Escribir pruebas de aceptación es como describir las acciones de un probador en PHP, son muy fácil de leer y muy fácil de escribir. No hay qu eolvidar de repoblar la base de datos en cada ensayo.
