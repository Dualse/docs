git b5117eaad549a50cd9630e831709ad01e5a7d4d3

---

# Консольные команды

- [Введение](#introduction)
- [Написание команд](#writing-commands)
    - [Генерация команд](#generating-commands)
    - [Структура команды](#command-structure)
    - [Команды-Замыкания](#closure-commands)
- [Определение входных параметров](#defining-input-expectations)
    - [Аргументы](#arguments)
    - [Опции](#options)
    - [Массивы](#input-arrays)
    - [Описание](#input-descriptions)
- [Ввод/Вывод](#command-io)
    - [Получение ввода](#retrieving-input)
    - [Диалог для ввода](#prompting-for-input)
    - [Вывод](#writing-output)
- [Регистрация команд](#registering-commands)
- [Программное исполнение команд](#programatically-executing-commands)
    - [Вызов команд из других команд](#calling-commands-from-other-commands)

<a name="introduction"></a>
## Введение

Artisan - это интерфейс командной строки (CLI), поставляющийся в комплекте с Laravel. Он предоставляет ряд полезных команд, которые могут помочь вам при создании приложения. Чтобы просмотреть список всех доступных команд Artisan, вы можете использовать команду `list`:
```
php artisan list
```

<a name="writing-command"></a>
## Написание команд

В дополнение к командам предоставленым Artisan'ом, вы также можете создавать свои собственные команды. Команды, как правило, хранятся в директории `app/Console/Commands`; Но, вы можете хранить их где угодно, лишь бы их смог загрузить Composer.

<a name="generating-commands"></a>
### Генерация команд

Чтобы создать новую команду, используйте команду Artisan'а `make:command`. Эта команда создаст новый класс команды в каталоге `app/Console/Commands`. Не беспокойтесь, если этот каталог пока не существует - он будет создан при первом запуске команды `make:command`. Сформированный класс команды будет включать в себя набор параметров по умолчанию свойств и методов, которые присутствуют во всех командах:

```
php artisan make:command SendEmails
```


<a name="command-structure"></a>
### Структура команды

После создания вашей команды, вы должны определить свойства класса `signature` и `description` , которые будут использоваться при отображении вашей команды на экране `list`. Метод `handle` будет исполнен, каждый раз когда ваша команда будет вызвана из терминала. Вы можете поместить логику команды в этом методе.

> Для лучшего переиспользования кода, хорошая практика - сохранять ваши консольные команды легкими, а тяжелые операции делегировать  сервисам приложения.  Обратите внимание, что в приведенном ниже примере мы вводим сервисный класс, чтобы сделать "тяжелую" отправку сообщений.

Давайте рассмотрим пример команды. Обратите внимание, что мы можем добавлять зависимости, которые нам необходимы в конструктор команды. Сервис-контейнер Laravel будет автоматически разрешать все типизированные зависимости:

```php
<?php

namespace App\Console\Commands;

use App\User;
use App\DripEmailer;
use Illuminate\Console\Command;

class SendEmails extends Command
{
    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'email:send {user}';

    /**
     * The console command description.
     *
     * @var string
     */
    protected $description = 'Send drip e-mails to a user';

    /**
     * The drip e-mail service.
     *
     * @var DripEmailer
     */
    protected $drip;

    /**
     * Create a new command instance.
     *
     * @param  DripEmailer  $drip
     * @return void
     */
    public function __construct(DripEmailer $drip)
    {
        parent::__construct();

        $this->drip = $drip;
    }

    /**
     * Execute the console command.
     *
     * @return mixed
     */
    public function handle()
    {
        $this->drip->send(User::find($this->argument('user')));
    }
}
```

<a name="closure-commands"></a>
### Команды-Замыкания


Команды, основанные на замыканиях - это альтернативный способ определения консольных команд. Точно так же, как замыкания в маршрутах являются альтернативой контроллерам. В методе `commands` файла `app/Console/Kernel.php`, Laravel загружает файл `routes/console.php`:

```php
/**
 * Register the Closure based commands for the application.
 *
 * @return void
 */
protected function commands()
{
    require base_path('routes/console.php');
}
```

Хотя этот файл не определяет HTTP-маршрутов, он определяет консоль на основе точек входа (маршрутов) в приложение. В этом файле вы можете определить все ваши маршруты, основанные на замыканиях, с помощью метода `Artisan::command` . Метод `command` принимает два аргумента: сигнатуру команды и замыкание, которорое, в свою очередь, может принимать аргументы и опции команды:

```php
Artisan::command('build {project}', function ($project) {
    $this->info("Building {$project}!");
});
```

Контекст замыкания привязан к экземпляру базовой команды, так что у вас есть полный доступ ко всем вспомогательным методам которые могут быть доступны при использовании класса.

#### Типизированые зависимости

Кроме получаемых опций команды, замыкание может принимаить типизированные зависимости, которые разрешит сервис-контейнер:

```php
use App\User;
use App\DripEmailer;

Artisan::command('email:send {user}', function (DripEmailer $drip, $user) {
    $drip->send(User::find($user));
});
```

#### Описание команд-замыканий

При определении команд на основе замыканий, вы можете использовать метод `describe`, чтобы добавить описание к команде. Это описание будет отображаться при запуске команд `php artisan list` и `php artisan help`:

```php
Artisan::command('build {project}', function ($project) {
    $this->info("Building {$project}!");
})->describe('Build the project');
```

<a name="defining-input-expectations"></a>
## Определение входных параметров

При написании консольных команд, часто бывает нужно получить данные от пользователя через аргументы или параметры. В Laravel можно очень удобно поределить ожидаемые на вход параметры или опции используя свойство команды `signature`. Свойство `signature` позволяет определить имя, аргументы и опции команды одним, выразительным, маршруто-подобным синтаксисом.

<a name="arguments"></a>
### Aргументы

Все пользовательские аргументы и опции оборачиваются в фигурные скобки. В следующем примере команда определяет один обязательный аргумент: `user`:

```php
/**
 * The name and signature of the console command.
 *
 * @var string
 */
protected $signature = 'email:send {user}';
```

Вы можете сделать аргументы необязательны или даже определить значение по умолчанию:

```
// Необязательный аргумент...
email:send {user?}

// Необязательный аргумент со значением по умолчанию...
email:send {user=foo}
```

<a name="options"></a>
### Опции 

Опции, как и аргументы - еще одна форма ввода данных пользователем. Параметры имеют префикс два дефиса (-), когда этот префикс указан в командной строке. Есть два типа опций: те, которые получают значение и те, которые этого не делают. Параметры, которые не получают значения служат как логический "переключатель". Давайте посмотрим на пример этого типа опций:

```php
/**
 * The name and signature of the console command.
 *
 * @var string
 */
protected $signature = 'email:send {user} {--queue}';
```

В этом примере --queue переключатель может быть указан при вызове команды Artisan. Если --queue переключатель принят, значение этой опции будет `true`. В противном случае значение будет `false`:

```
php artisan email:send 1 --queue
```

#### Опции со значениями

Далее, давайте взглянем на вариант, который ожидает значение. Если пользователь должен задать значение для опции, нужно указать суффикс `=` для имени параметра:

```php
/**
 * The name and signature of the console command.
 *
 * @var string
 */
protected $signature = 'email:send {user} {--queue=}';
```

В этом примере, пользователь может передать значение для параметра, таким образом:

```
php artisan email:send 1 --queue=default
```


Конечно же, вы назначить значение по умолчанию для опции, указав его после имени опции и суффикса. Если значение опции не передается пользователем, будет использоваться значение по умолчанию:

```
email:send {user} {--queue=default}
```

<a name="input-arrays"></a>
### Массивы

Если вы хотите, определить аргументы или параметры ожидающие на вход массив, вы можете использовать символ `*`. Во-первых, давайте взглянем на пример, который определяет аргумент-массив:

```
email:send {user*}
```
При вызове этого метода, аргументы пользователя могут быть переданы последовательно в командной строке. Например, следующая команда установит значение `user` как `['Foo', 'Bar']`:

```
php artisan email:send foo bar
```

При определении параметра, который ожидает на вход массив, каждое значение опции передаваемое команде должно начинаться с префикса имени опции:

```
// сигнатура...
email:send {--id=*}

// команда...
php artisan email:send --id=1 --id=2
```

<a name="input-descriptions"></a>
### Описание

Вы можете назначить описание для параметров ввода и опций, отделяя параметр от описания, используя двоеточие. Если вам нужно немного больше места, чтобы определить вашу команду, не стесняйтесь растянуть определение на несколько строк:

```php
/**
 * The name and signature of the console command.
 *
 * @var string
 */
protected $signature = 'email:send
                        {user : The ID of the user}
                        {--queue= : Whether the job should be queued}';
```

<a name="command-io"></a>
## Ввод/Вывод

### Получение данных ввода

Во время исполнения команды, вам очевидно необходимо иметь доступ к значениям аргументов и опций, принятых командой. Для этого вы можете использовать соответствующие методы `argument` и `options` :

```php
/**
 * Execute the console command.
 *
 * @return mixed
 */
public function handle()
{
    $userId = $this->argument('user');

    //
}
```

Если вам нужно получить все аргументы в виде массива, вызовите метод `arguments`:

```
$arguments = $this->arguments();
```

Опции могут быть получены точно так же как аргументы, используя метод `options`. Чтобы получить все опции в виде массива, вызовите метод `options`:

```php
// Получение конкретной опции...
$queueName = $this->option('queue');

// Получение всех опций...
$options = $this->options();
```

Если аргумент или опция не существует, будет возвращен `null`.

<a name="prompting-for-input"></a>
### Prompting For Input

По мимо отображения вывода, вы также можете попросить пользователя внести какие-либо данные прямо во время исполнения вашей команды. Метод `ask` вызовет диалог с переданным вопросом, примет ввод от пользователя, и затем передаст пользовательский ввод обратно в команду:

```php
/**
 * Execute the console command.
 *
 * @return mixed
 */
public function handle()
{
    $name = $this->ask('What is your name?');
}
```
Метод `secret` похож на `ask`, но ввод пользователя не будет виден ему, при вводе. Этот метод полезен при запросе конфиденциальной информации, например пароля:

```php
$password = $this->secret('What is the password?');
```

#### Запрос подтверждения

Если вам нужно запросить у пользователя простое подтверждение, вы можете использовать метод `confirm`. По умолчанию этот метод возвращает `false`. Однако, если пользователь вводит `y` в ответ на приглашение, метод возвращает `true`.

```
if ($this->confirm('Do you wish to continue? [y|N]')) {
    //
}
```

#### Авто-дополнение

Метод `anticipate` может использоваться для автодополнения возможного выбора. Пользователь может выбрать ответ, независимо от подсказок aвто-дополнения:

```php
$name = $this->anticipate('What is your name?', ['Taylor', 'Dayle']);
```



#### Вопросы с множественным выбором

Если вам нужно дать пользователю заранее определенный набор вариантов, вы можете использовать метод `choice`. Вы можете установить значение по умолчанию, которое будет возвращено, если не выбран ни один вариант:

```php
$name = $this->choice('What is your name?', ['Taylor', 'Dayle'], $default);
```

<a name="writing-output"></a>
### Writing Output

Для вывода данных в консоль, используйте методы `line`, `info`, `comment`, `question` и `error`. Каждый из этих методов будет использовать соответствующие цвета ANSI для своих целей. Например, давайте покажем некоторую общую информацию пользователю. Как правило, метод `info` выведет в консоль зеленый текст:

```php
/**
 * Execute the console command.
 *
 * @return mixed
 */
public function handle()
{
    $this->info('Display this on the screen');
}
```

Чтобы отобразить сообщение об ошибке, используйте метод `error`. Текст сообщения об ошибке обычно отображается красным цветом:
```php
$this->error('Something went wrong!');
```

Если вы хотите, чтобы отобразить простой, неокрашенный выход консоли, используйте метод `line`:

```php
$this->line('Display this on the screen');
```

#### Макеты таблиц

Метод `table` позволяет легко  отформатировать несколько строк/столбцов данных. Просто передайте заголовки и строки в метод. Ширина и высота будет динамически вычисляется на основании приведенных данных:

```php
$headers = ['Name', 'Email'];

$users = App\User::all(['name', 'email'])->toArray();

$this->table($headers, $users);
```

#### Индикаторы прогресса

Для длительных задач может оказаться полезныи вывести на экран индикатор прогресса. Используя объект вывода, мы можем начать, продвинуть или остановить индикатор. Сначала, нужно определить общее количество шагов которые процесс будет перебирать. Затем, продвинуть индикатор после обработки каждого элемента:

```
$users = App\User::all();

$bar = $this->output->createProgressBar(count($users));

foreach ($users as $user) {
    $this->performTask($user);

    $bar->advance();
}

$bar->finish();
```

Для использования более продвинутых опций, читайте документацию компонента Symfony Progress Bar.

<a name="registering-commands"></a>
## Регистрация команд

После того, как ваша команда будет написана, вы должны зарегистрировать ее в Artisan. Все команды регистрируются в файле `app/Console/Kernel.php`. В этом файле вы найдете список команд в свойстве `commands`. Чтобы зарегистрировать команду, просто добавьте имя класса команды к списку. Когда Artisan загрузится, все команды, перечисленные в этом свойстве, будут разпрешены сервис-контейнером и зарегистрированы в Artisan:

```php
protected $commands = [
    Commands\SendEmails::class
];
```

<a name="programatically-executing-commands"></a>
## Программное исполнение команд

Иногда вам может понадобиться выполнить команду Artisan вне CLI. Например, вы можете захотеть вызвать команду Artisan из маршрута или контроллера. Эля этого вы можете использовать метод `call` фасада `Artisan`. Метод `call` принимает имя команды в качестве первого аргумента, и массив параметров команды в качестве второго аргумента. Код завершения команды будет возвращен:

```php
Route::get('/foo', function () {
    $exitCode = Artisan::call('email:send', [
        'user' => 1, '--queue' => 'default'
    ]);

    //
});
```

Используя метод `queue` фасада `Artisan`, даже вы можете вызывать очереди команд Artisan, тогда они будут обработаны в фоновом режиме вашими процессами очередей. Перед использованием этого метода, убедитесь, что вы настроили свою очередь и слушатель очереди зхапущен:

```php
Route::get('/foo', function () {
    Artisan::queue('email:send', [
        'user' => 1, '--queue' => 'default'
    ]);

    //
});
```

Если вам необходимо указать значение опции, которая не принимает строковые значения, например флаг `--force`  на команды `migrate:refresh` , вы можете передать `true` или `false`:

```php
$exitCode = Artisan::call('migrate:refresh', [
    '--force' => true,
]);
```

<a name="calling-commands-from-other-commands"></a>
### Вызов команд из других команд

Вы можете вызывать другие команды из текущей команды Artisan. Вы можете сделать это с помощью метода `call`. Этот метод принимает имя команды и массив параметров команды:

```php
/**
 * Execute the console command.
 *
 * @return mixed
 */
public function handle()
{
    $this->call('email:send', [
        'user' => 1, '--queue' => 'default'
    ]);

    //
}
```
Если вы хотите вызвать другую команду консоли и заглушить весь ее вывод, используйте метод `callSilent` . Метод `callSilent` имеет ту же сигнатуру, что и метод `call`:

```php
$this->callSilent('email:send', [
    'user' => 1, '--queue' => 'default'
]);
```
