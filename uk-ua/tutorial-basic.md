---
layout: default
title: 'Посібник - основи'
keywords: 'tutorial, basic tutorial, step by step, mvc, посібник, навчання, основи, крок за кроком'
---

# Посібник - основи
- - -
![](/assets/images/document-status-stable-success.svg) ![](/assets/images/version-{{ pageVersion }}.svg)

## Огляд
В цьому посібнику ми створимо програму з простою реєстраційною формою, розкриваючи основні аспекти дизайну Phalcon.

Цей посібник охоплює реалізацію простого MVC додатку, показуючи, як швидко і легко це можна зробити за допомогою Phalcon. Після розробки ви можете скористатися цим додатком і розширити його для задоволення ваших потреб. Код в цьому посібнику також може використовуватися як майданчик для вивчення інших понять та ідей Phalcon.

<iframe width="560" height="315" src="https://www.youtube.com/embed/75W-emM4wNQ" frameborder="0" allowfullscreen></iframe>

Якщо ви хочете просто почати роботу, ви можете пропустити це і створити проект Phalcon автоматично за допомогою наших [інструментів розробника](devtools).

Найкращий спосіб використовувати цей посібник - вникнути в основи та спробувати насолодитись процесом. You can get the complete code [here][github_tutorial]. If you get stuck or have questions, please visit us on [Discord][discord] or in our [Discussions][discussions].

## Файлова структура
Однією з головних особливостей Phalcon є слабка зв'язаність. Через це ви можете використати будь-яку структуру каталогів, яка вам зручна. In this tutorial we will use a _standard_ directory structure, commonly used in MVC applications.

```text
.
└── tutorial
    ├── app
    │   ├── controllers
    │   │   ├── IndexController.php
    │   │   └── SignupController.php
    │   ├── models
    │   │   └── Users.php
    │   └── views
    └── public
        ├── css
        ├── img
        ├── index.php
        └── js
```

> **NOTE**: Since all the code that Phalcon exposes is encapsulated in the extension (that you have loaded on your web server), you will not see `vendor` directory containing Phalcon code. Все, що вам потрібно, знаходиться в пам'яті. Якщо Ви ще не встановили розширення, перейдіть на сторінку [установка](installation) і завершіть встановлення перед тим, як продовжувати виконувати вказівки цього посібника. 
> 
> {: .alert .alert-warning }

Якщо це все абсолютно нове для вас, рекомендується також встановити [Phalcon Devtools](devtools). DevTools доповнює вбудований веб-сервер PHP, що дозволяє вам запустити ваш продукт майже миттєво. Якщо ви виберете цю опцію, то вам знадобиться файл `.htrouter.php` в кореневому каталозі вашого проекту з наступним змістом:

```php
<?php

$uri = urldecode(
    parse_url($_SERVER['REQUEST_URI'], PHP_URL_PATH)
);

if ($uri !== '/' && file_exists(__DIR__ . '/public' . $uri)) {
    return false;
}

$_GET['_url'] = $_SERVER['REQUEST_URI'];

require_once __DIR__ . '/public/index.php';
```

У випадку цього посібника цей файл повинен знаходитися в каталозі `tutorial`.

Ви також можете використовувати nginX, apache, cherokee або інші веб-сервери. Ви можете відвідати [сторінку налаштування веб-сервера](webserver-setup) для отримання інструкцій.

## Bootstrap
Перший файл, який потрібно створити - це файл bootstrap. Цей файл працює як вхідна точка і базова конфігурація для вашого продукту. У цьому файлі можна реалізувати ініціалізацію компонентів, а також визначити поведінку програми.

Цей файл виконує три завдання:
- Реєстрація автозавантажувачів компонентів
- Налаштування сервісів та реєстрація їх у контейнері управління залежностями (Dependency Injection)
- Забезпечення виконання HTTP запитів вашого продукту

### Автозавантажувач
We are going to use [Phalcon\Autoload\Loader](autoload) a [PSR-4][psr-4] compliant file loader. Загальні речі, які слід додати до автозавантажувача, це ваші контролери і моделі. Ви також можете зареєструвати каталоги, які будуть проскановані для пошуку файлів, необхідних вашій програмі.

To start, lets register our app's `controllers` and `models` directories using [Phalcon\Autoload\Loader](autoload):

`public/index.php`
```php
<?php

use Phalcon\Autoload\Loader;

define('BASE_PATH', dirname(__DIR__));
define('APP_PATH', BASE_PATH . '/app');
// ...

$loader = new Loader();
$loader->setDirectories(
    [
        APP_PATH . '/controllers/',
        APP_PATH . '/models/',
    ]
);


$loader->register();
```

### Керування залежностями
Since Phalcon is loosely coupled, services are registered with the frameworks Dependency Manager, so they can be injected automatically to components and services wrapped in the [IoC][ioc] container. Часто ви стикатиметесь з терміном DI, який означає впровадження залежностей (Dependency Injection). Впровадження залежностей та Інверсія контролю (IoC) можуть звучати складно, але Phalcon гарантує, що їх використання є простим, практичним та ефективним. Контейнер IoC Phalcon містить такі концепції:
- Сервіс-контейнер - "сховище", де ми зберігаємо сервіси, які наш продукт потребує для функціонування.
- Послуга або компонент - об'єкт обробки даних, який буде включено до компонентів

Кожен раз, коли фреймворк потребує компонента або послугу, він буде звертатися до контейнера використовуючи певне ім'я сервісу. Таким чином, ми маємо простий спосіб отримання об'єктів, необхідних для нашої програми, таких як логер, з'єднання бази даних і тощо.

> **NOTE**: If you are still interested in the details please see this article by [Martin Fowler][injection]. Also, we have [a great tutorial](di) covering many use cases. 
> 
> {: .alert .alert-warning }

### Factory Default
The [Phalcon\Di\FactoryDefault][di-factorydefault] is a variant of [Phalcon\Di\Di][di]. Щоб спростити роботу, він автоматично зареєструє більшість компонентів, що необхідні для продукту та стандартно поставляється з Phalcon. Although it is recommended to set up services manually, you can use the [Phalcon\Di\FactoryDefault][di-factorydefault] container initially and later on customize it to fit your needs.

Services can be registered in several ways, but for our tutorial, we will use an [anonymous function][anonymous_function]:

`public/index.php`

```php
<?php

use Phalcon\Di\FactoryDefault;

$container = new FactoryDefault();
```

Now we need to register the _view_ service, setting the directory where the framework will find the view files. Оскільки представлення не належать до класів, вони не можуть бути автоматично завантажені нашим автозавантажувачем.

`public/index.php`
```php
<?php

use Phalcon\Mvc\View;

// ...

$container->set(
    'view',
    function () {
        $view = new View();
        $view->setViewsDir(APP_PATH . '/views/');

        return $view;
    }
);
```

Тепер нам потрібно зареєструвати базову URI, яка надасть можливість створення всіх URL-адрес за допомогою Phalcon. Цей компонент буде гарантувати, що якщо ви запускатимете програму через верхній каталог або підкаталог, всі ваші URI будуть правильними. Для цього навчального посібника наш базовий шлях є `/`. Це матиме значення пізніше у цьому посібнику, коли ми використовуватимемо клас `Phalcon\Tag` для генерування гіперпосилань.

`public/index.php`
```php
<?php

use Phalcon\Mvc\Url;

// ...

$container->set(
    'url',
    function () {
        $url = new Url();
        $url->setBaseUri('/');

        return $url;
    }
);
```

### Обробка запитів додатка
Для того, щоб обробляти будь-які запити, використовуєтьс об'єкт [Phalcon\Mvc\Application](application), який виконує всі самі важкі завдання. The component will accept the request by the user, detect the routes and dispatch the controller and render the view returning the results.

`public/index.php`
```php
<?php

use Phalcon\Mvc\Application;

// ...

$application = new Application($container);

$response = $application->handle(
    $_SERVER["REQUEST_URI"]
);

$response->send();
```

### А тепер зберемо все разом
Файл `tutorial/public/index.php` повинен виглядати так:

`public/index.php`
```php
<?php

use Phalcon\Di\FactoryDefault;
use Phalcon\Loader\Loader;
use Phalcon\Mvc\View;
use Phalcon\Mvc\Application;
use Phalcon\Url;

define('BASE_PATH', dirname(__DIR__));
define('APP_PATH', BASE_PATH . '/app');

$loader = new Loader();

$loader->registerDirs(
    [
        APP_PATH . '/controllers/',
        APP_PATH . '/models/',
    ]
);

$loader->register();

$container = new FactoryDefault();

$container->set(
    'view',
    function () {
        $view = new View();
        $view->setViewsDir(APP_PATH . '/views/');
        return $view;
    }
);

$container->set(
    'url',
    function () {
        $url = new Url();
        $url->setBaseUri('/');
        return $url;
    }
);

$application = new Application($container);

try {
    // Опрацьовуємо запити
    $response = $application->handle(
        $_SERVER["REQUEST_URI"]
    );

    $response->send();
} catch (\Exception $e) {
    echo 'Exception: ', $e->getMessage();
}
```

> **NOTE** In the tutorial files from our [GitHub][github_tutorial] repository, to register services in the `DI` container, we use the array notation i.e. `$container['url'] = ....`. 
> 
> {: .alert .alert-info }

As you can see, the bootstrap file is very short, and we do not need to include any additional files. Ви маєте змогу створити гнучкий MVC додаток менш ніж за 30 рядків коду.

## Створення контролера
By default, Phalcon will look for a controller named `IndexController`. It is the starting point when no controller or action has been added in the request (e.g. `https://localhost/`). `IndexController` та його `IndexAction` повинен бути схожим на такий приклад:

`app/controllers/IndexController.php`
```php
<?php

use Phalcon\Mvc\Controller;

class IndexController extends Controller
{
    public function indexAction()
    {
        return '<h1>Привіт!</h1>';
    }
}
```

Класи контролера повинні містити суфікс `Controller`, а дії контролера повинні мати суфікс `Action`. Для отримання додаткової інформації ви можете прочитати наш документ про [контролери](controllers). Якщо ви спробуєте отримати досту до вашого продукта через браузер, то побачите щось на зразок цього:

![](/assets/images/content/tutorial-basic-1.png)

> **Congratulations, you are Phlying with Phalcon!** 
> 
> {: .alert .alert-info }

## Відправка результату до View
Виведення результату на екран безпосередньо з контролера часом потрібне, але не бажане, і це підтвердять більшість пуристів MVC спільноти. Всі результати мають передаватись компоненту view, який відповідальний за виведення інформації на екран. Phalcon шукатиме view з такою ж назвою, як і остання виконана дія, в папці з ім'ям контролера, якому така дія належить.

Therefore, in our case if the URL is:

```php
http://localhost/
```

буде викликано `IndexController` і `indexAction`, який буде шукати подання:

```php
/views/index/index.phtml
```

Якщо такий файл існує, його буде зчитано і виведено результат на екран. Наш view в такому разі матиме вміст:

`app/views/index/index.phtml`
```php
<?php echo "<h1>Привіт!</h1>";
```

і оскільки ми перемістили `echo` з нашої дії контролера у подання, то дія буде порожньою:

`app/controllers/IndexController.php`
```php
<?php

use Phalcon\Mvc\Controller;

class IndexController extends Controller
{
    public function indexAction()
    {

    }
}
```

Виведена браузером інформація не зміниться. Компонент `Phalcon\Mvc\View` автоматично створюється по завершенню виконання дії. Інформацію про подання у Phalcon ви можете почитати [тут](views).

## Створення форми реєстрації
Now we will change the `index.phtml` view file, to add a link to a new controller named _signup_. Мета - дозволити користувачам зареєструватися у нашому додатку.

`app/views/index/index.phtml`
```php
<?php

echo "<h1>Hello!</h1>";

echo PHP_EOL;

echo PHP_EOL;

echo $this->tag->linkTo(
    'signup',
    'Sign Up Here!'
);
```

Згенерований HTML код показує посилання (`&lt;a&gt;</code), що веде до нового контролера:</p>

<p spaces-before="0"><code>app/views/index/index.phtml` (зчитано)
```html
<h1>Hello!</h1>

<a href="/signup">Sign Up Here!</a>
```

To generate the link for the `<a>` tag, we use the [Phalcon\Html\TagFactory](html-tagfactory) component. Це допоміжний інструмент, який пропонує простий спосіб побудови HTML-тегів з урахуванням правил фреймворку. This class is also a service registered in the Dependency Injector, so we can use `$this->tag` to access its functionality.

> **NOTE**: `Phalcon\Html\TagFactory` is already registered in the DI container since we have used the `Phalcon\Di\FactoryDefault` container. Якщо ви вирішили реєструвати усі сервіси самостійно, то потрібно зареєструвати у цьому контейнері кожен такий сервіс, щоб зробити його доступним у вашому продукті. 
> 
> {: .alert .alert-info }

![](/assets/images/content/tutorial-basic-2.png)

А контролер реєстрації - (`app/controllers/SignupController.php`):

`app/controllers/SignupController.php`
```php
<?php

use Phalcon\Mvc\Controller;

class SignupController extends Controller
{
    public function indexAction()
    {

    }
}
```

Порожня дія індексу забезпечує безпосередній перехід до файлу подання з визначенням форми (`app/views/signup/index.phtml`):

`app/views/signup/index.phtml`
```html
<h2>Sign up using this form</h2>

<?php echo $this->tag->form("signup/register"); ?>

    <p>
        <label for="name">Name</label>
        <?php echo $this->tag->textField("name"); ?>
    </p>

    <p>
        <label for="email">E-Mail</label>
        <?php echo $this->tag->textField("email"); ?>
    </p>

    <p>
        <?php echo $this->tag->submitButton("Register"); ?>
    </p>

</form>
```

Перегляд форми у вашому браузері покаже наступне:

![](/assets/images/content/tutorial-basic-3.png)

As mentioned above, the [Phalcon\Html\TagFactory](html-tagfactory) utility class, exposes useful methods allowing you to build form HTML elements with ease. The `form()` method receives an array of key/value pairs that set up the form, for example a relative URI to a controller/action in the application. The `inputText()` creates a text HTML element with the name as the passed parameter, while the `inputSubmit()` creates a submit HTML button. Finally, a call to `close()` will close our `<form>` tag.

By clicking the _Register_ button, you will notice an exception thrown from the framework, indicating that we are missing the `register` action in the controller `signup`. Наш `public/index.php` згенерує цей виняток:

```bash
Exception: Action "register" was not found on handler "signup"
```

Реалізація цього методу дозволить уникнути виняткової ситуації:

`app/controllers/SignupController.php`
```php
<?php

use Phalcon\Mvc\Controller;

class SignupController extends Controller
{
    public function indexAction()
    {

    }

    public function registerAction()
    {

    }
}
```

If you click the _Register_ button again, you will see a blank page. Трохи пізніше ми додамо подання, що міститиме корисний відгук. Але спочатку ми маємо написати код для зберігання записів користувачів у базі даних.

Відповідно до рекомендацій MVC, взаємодії з базами даних повинні здійснюватися за допомогою моделей для гарантування чистого об'єктно-орієнтованого коду.

## Створення моделі
Phalcon пропонує перший ORM для PHP повністю написаний на мові C. Замість того, щоб підвищити складність розробки, він спрощує її.

Перед створенням нашої першої моделі, нам потрібно створити таблицю баз даних використовуючи інструмент доступу до бази даних або командний рядок бази даних. Для цього навчального посібника ми використовуємо MySQL як нашу базу даних. Просту таблицю для зберігання зареєстрованих користувачів можна створити наступним чином:

`create_users_table.sql`
```sql
CREATE TABLE `users`
(
    `id`    int unsigned NOT NULL AUTO_INCREMENT COMMENT 'Record ID',
    `name`  varchar(255) NOT NULL COMMENT 'User Name',
    `email` varchar(255) NOT NULL COMMENT 'User Email Address',
    PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

Модель має розташовуватись у каталозі `app/models` (`app/models/Users.php`). The model maps to the _users_ table:

`app/models/Users.php`
```php
<?php

use Phalcon\Mvc\Model;

class Users extends Model
{
    public $id;
    public $name;
    public $email;
}
```

> **NOTE**: Note that the public properties of the model correspond to the names of the fields in our table. 
> 
> {: .alert .alert-info }

## Встановлення підключення до бази даних
Для того, щоб використовувати з'єднання з базою даних і за потреби отримати доступ до даних через наші моделі, ми повинні це визначити в нашому процесі завантаження. Зв'язок з базою даних - це ще один сервіс нашого додатка, який у ньому може використовуватися:

`public/index.php`
```php
<?php

use Phalcon\Db\Adapter\Pdo\Mysql;

$container->set(
    'db',
    function () {
        return new Mysql(
            [
                'host'     => '127.0.0.1',
                'username' => 'root',
                'password' => 'secret',
                'dbname'   => 'tutorial',
            ]
        );
    }
);
```

Змініть вищевказаний фрагмент коду відповідно до налаштувань доступу до вашої бази даних.

With the correct database parameters, our model is ready to interact with the rest of the application, so we can save the user's input. First, let's take a moment and create a view for `SignupController::registerAction()` that will display a message letting the user know the outcome of the _save_ operation.

`app/views/signup/register.phtml`
```php
<div class="alert alert-<?php echo $success === true ? 'success' : 'danger'; ?>">
    <?php echo $message; ?>
</div>

<?php echo $this->tag->linkTo(['/', 'Go back', 'class' => 'btn btn-primary']); ?>
```
Зауважте, що ми додали деякі CSS стилі в зазначений код. Ми розкриємо вміст таблиці стилів нижче у розділі [Оформлення](#styling).

## Зберігання даних за допомогою моделей

`app/controllers/SignupController.php`
```php
<?php

use Phalcon\Mvc\Controller;

class SignupController extends Controller
{
    public function indexAction()
    {

    }

    public function registerAction()
    {
        $post = $this->request->getPost();

        // Store and check for errors
        $user        = new Users();
        $user->name  = $post['name'];
        $user->email = $post['email'];
        // Store and check for errors
        $success = $user->save();

        // passing the result to the view
        $this->view->success = $success;

        if ($success) {
            $message = "Thanks for registering!";
        } else {
            $message = "Sorry, the following problems were generated:<br>"
                . implode('<br>', $user->getMessages());
        }

        // передача повідомлення поданню
        $this->view->message = $message;
    }
}
```

На початку `registerAction` ми створили порожній об'єкт користувача, використовуючи клас користувачів `Users`, який ми створили раніше. Ми будемо використовувати цей клас для керування записами користувача. Як було зазначено вище, публічні властивості класу ведуть до полів таблиці `users` у нашій базі даних. Вставлення відповідних значень до нового запису і виклик `save()` збереже дані цього запису у базі даних. Метод `save()` повертає значення `boolean`, яке сигналізує про успіх чи невдачу операції збереження.

The ORM will automatically escape the input preventing SQL injections, so we only need to pass the request to the `save()` method.

Додаткова перевірка відбувається автоматично в усіх полях, що визначені як не null (обов’язкові). Якщо ми не заповнимо жодного з обов'язкових полів у реєстраційній формі, наш екран виглядатиме схоже на це:

![](/assets/images/content/tutorial-basic-4.png)

## Список зареєстрованих користувачів
Тепер нам потрібно буде відобразити всіх зареєстрованих користувачів у нашій базі даних

Перше, що ми зробимо в `indexAction` контролера `IndexController` - це показати результат пошуку всіх користувачів, що робиться просто шляхом виклику статичного методу `find()` в нашій моделі (`Users::find()`).

`indexAction` буде змінено так:

`app/controllers/IndexController.php`
```php
<?php

use Phalcon\Mvc\Controller;

class IndexController extends Controller
{
    /**
     * Вітання та список користувачів
     */
    public function indexAction()
    {
        $this->view->users = Users::find();
    }
}
```

> **NOTE**: We assign the results of the `find` to a magic property on the `view` object. This sets this variable with the assigned data and makes it available in our view 
> 
> {: .alert .alert-info }

У нашому файлі подання `views/index/index.phtml` ми можемо використовувати змінну `$users` так:

Ураховуючи все вищезазначене, наше подання виглядатиме приблизно так:

`views/index/index.phtml`
```html
<?php

echo "<h1>Привіт!</h1>";

echo $this->tag->linkTo(["signup", "Зареєструватися!", 'class' => 'btn btn-primary']);

if ($users->count() > 0) {
    ?>
    <table class="table table-bordered table-hover">
        <thead class="thead-light">
        <tr>
            <th>N п/п</th>
            <th>Ім'я</th>
            <th>Email</th>
        </tr>
        </thead>
        <tfoot>
        <tr>
            <td colspan="3"> Кількість користувачів: <?php echo $users->count(); ?></td>
        </tr>
        </tfoot>
        <tbody>
        <?php foreach ($users as $user) { ?>
            <tr>
                <td><?php echo $user->id; ?></td>
                <td><?php echo $user->name; ?></td>
                <td><?php echo $user->email; ?></td>
            </tr>
        <?php } ?>
        </tbody>
    </table>
    <?php
}
```

As you can see our variable `$users` can be iterated and counted. Ви можете отримати більше інформації про те, як функціонують моделі, в нашому документі про [моделі](db-models).

![](/assets/images/content/tutorial-basic-5.png)

## Styling
Тепер ми можемо трохи покращити зовнішній вигляд нашої програми. We can add the [Bootstrap CSS][bootstrap] in our code so that it is used throughout our views. Ми додамо файл `index.phtml` до теки`views` з таким змістом:

`app/views/index.phtml`
```html
<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Phalcon Tutorial</title>
    <link rel="stylesheet" 
          href="https://cdn.jsdelivr.net/npm/bootstrap@5.2.0/dist/css/bootstrap.min.css">
</head>
<body>
<div class="container">
    <?php echo $this->getContent(); ?>
</div>
</body>
</html>
```

У наведеному вище шаблоні найважливіший рядок це виклик метода `getContent()`. Цей метод повертає весь вміст, створений нашим поданням. Тепер наш додаток показуватиме:

![](/assets/images/content/tutorial-basic-6.png)

## Підсумок
Як бачите, почати будувати застосунок за допомогою Phalcon досить просто. Оскільки Phalcon це розширення, завантажене в пам'ять, обсяг коду вашого проекту буде мінімальним, тоді як вам сподобається гарний приріст продуктивності.

Якщо ви готові ще більше дізнатися, то перейдіть далі до [Посібника Vökuró](tutorial-vokuro).

[anonymous_function]: https://php.net/manual/en/functions.anonymous.php
[discord]: https://phalcon.io/discord
[discussions]: https://phalcon.io/discussions
[github_tutorial]: https://github.com/phalcon/tutorial
[github_tutorial]: https://github.com/phalcon/tutorial
[injection]: https://martinfowler.com/articles/injection.html
[ioc]: https://en.wikipedia.org/wiki/Inversion_of_control
[psr-4]: https://www.php-fig.org/psr/psr-4/
[di]: api/phalcon_di
[di-factorydefault]: api/phalcon_di#di-factorydefault
[bootstrap]: https://getbootstrap.com/
