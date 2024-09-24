# Лабораторная работа №2. HTTP-запросы и шаблонизация в Laravel

## Цель работы

Изучить основные принципы работы с HTTP-запросами в Laravel и шаблонизацию с использованием Blade на основе веб-приложения `To-Do App для команд` — приложения для управления задачами внутри команды.

Приложение предназначено для команды, которая хочет управлять своими задачами, назначать их участникам, отслеживать статус и приоритет задач (похоже на Github Issues).

### №1. Подготовка к работе, установка Laravel

1. Открываю терминал и создаю проект Laravel с именем todo-app

```bash
composer create-project laravel/laravel:^10 todo-app
```

2. Перехожу в директорию проекта

```bash
cd todo-app
```

3. Запускаю сервер Laravel

```bash
php artisan serve
```

**Вопрос**: Что вы видите в браузере, открыв страницу `http://localhost:8000`?

- Открыв страницу в браузере `http://localhost:8000`, увидел стандартную стартовую страницу с Laravel приветствием, ссылками на документацию и ресурсы

### №2. Настройка окружения

1. Открываю файл окружения `.env` и указываю следующие настройки приложения:

```ini
APP_NAME=ToDoApp
APP_ENV=local
APP_KEY=
APP_DEBUG=true
APP_URL=http://localhost:8000
```

2. Генерирую ключ приложения

```bash
php artisan key:generate
```

**Вопрос**: Что будет, если данный ключ попадет в руки злоумышленника?

- Злоумышленник сможет расшифровать все данные приложение, которые были зашифрованы этим ключом. Это может быть личная информация пользователей (личная информация, пароли и т.д.)

### №3. Основы работы с HTTP-запросами

#### №3.1. Создание маршрутов для главной страницы и страницы "О нас"

1. Создайте класс-контроллер `HomeController` для обработки запросов на главную страницу.

2. Добавьте метод `index` в `HomeController`, который будет отвечать за отображение главной страницы.

3. Создайте маршрут для главной страницы в файле `routes/web.php`.

   - Откройте браузер и перейдите по адресу `http://localhost:8000`. Убедитесь, что загружается пустая страница, так как представление home.blade.php пока не создано.

4. В этом же контроллере `HomeController` создайте метод для страницы **"О нас"**.

```php
class HomeController extends Controller
{
	public function index()
	{
		return view("home");
	}

	public function about()
	{
		return view("about");
	}
}
```

5. Добавьте маршрут для страницы "О нас" в файле `routes/web.php`.

```php
Route::get('/', [HomeController::class, 'index'])->name('home');
Route::get('/about', [HomeController::class, 'about'])->name('about');
```

#### №3.2. Создание маршрутов для задач

1. Создайте класс-контроллер `TaskController` для обработки запросов, связанных с задачами, и добавьте следующие методы:

   - `index` — отображение списка задач;
   - `create` — отображение формы создания задачи;
   - `store` — сохранение новой задачи;
   - `show` — отображение задачи;
   - `edit` — отображение формы редактирования задачи;
   - `update` — обновление задачи;
   - `destroy` — удаление задачи.

**Примечание:** _Пока мы не изучили работу с формами и запросами POST и PUT, методы `store`, `create`, `edit`, `update`, `destroy` оставьте пустыми. Мы вернемся к ним позже в курсе. В данный момент сосредоточимся на отображении информации._

```php
class TaskController extends Controller
{
	public function index()
	{
		return 'This a list of tasks';
	}
	public function create()
	{
		return 'Create a task';
	}
	public function store(Request $request)
	{
		return 'Save a task';
	}
	public function show($id)
	{
		return 'Show a task';
	}
	public function edit($id)
	{
		return 'Edit a task';
	}
	public function update(Request $request, $id)
	{
		return 'Update a task';
	}
	public function destroy($id)
	{
		return 'Destroy a task';
	}
}
```

2. Создайте маршруты для методов контроллера `TaskController` в файле `routes/web.php` и укажите правильные HTTP-методы для каждого маршрута.

3. Используйте группирование маршрутов для контроллера `TaskController` с префиксом `/tasks`, чтобы упростить маршрутизацию и улучшить читаемость кода.

4. Определите правильные имена маршрутов для методов контроллера `TaskController`, например:

   - `tasks.index` — список задач;
   - `tasks.show` — отображение отдельной задачи.
   - ...

5. Добавьте валидацию параметров маршрута `id` для задач. Убедитесь, что параметр `id` — это положительное целое число. Используйте метод `where`, чтобы ограничить значения для параметра `id`.

```php
Route::prefix('tasks')->group(function () {
    Route::get('/', [TaskController::class, 'index'])->name('tasks.index');
    Route::get('/create', [TaskController::class, 'create'])->name('tasks.create');
    Route::post('/', [TaskController::class, 'store'])->name('tasks.store');
    Route::get('/{id}', [TaskController::class, 'show'])->name('tasks.show')->where('id', '[0-9]+');
    Route::get('/{id}/edit', [TaskController::class, 'edit'])->name('tasks.edit')->where('id', '[0-9]+');
    Route::put('/{id}', [TaskController::class, 'update'])->name('tasks.update')->where('id', '[0-9]+');
    Route::delete('/{id}', [TaskController::class, 'destroy'])->name('tasks.destroy')->where('id', '[0-9]+');
});
```

6. Вместо ручного создания маршрутов для каждого метода можно использовать **ресурсный контроллер**, который автоматически создаст маршруты для всех **CRUD-операций**:
   - В файле `routes/web.php` замените ручное создание маршрутов для контроллера `TaskController` на ресурсный контроллер:
     ```php
     Route::resource('tasks', TaskController::class);
     ```
   - **Вопрос**: Объясните разницу между ручным созданием маршрутов и использованием ресурсного контроллера. Какие маршруты и имена маршрутов будут созданы автоматически?

```php
Route::resource('tasks', TaskController::class);
```

- **Ручное создание маршрутов:** Вы создаете каждый маршрут по отдельности, задавая контроллер, метод, HTTP-метод и маршрут. Это более гибкий способ, но требует больше кода и внимания к каждому маршруту.

- **Ресурсный контроллер:** Laravel автоматически создаёт все необходимые маршруты для операций CRUD (создание, чтение, обновление, удаление), что экономит время и делает код более компактным.

7. Проверьте созданные маршруты с помощью команды `php artisan route:list`.

```bash
GET|HEAD        / ............................... home › HomeController@index
POST            _ignition/execute-solution ignition.executeSolution › Spatie…
GET|HEAD        _ignition/health-check ignition.healthCheck › Spatie\Laravel…
POST            _ignition/update-config ignition.updateConfig › Spatie\Larav…
GET|HEAD        about .......................... about › HomeController@about
GET|HEAD        api/user ....................................................
GET|HEAD        sanctum/csrf-cookie sanctum.csrf-cookie › Laravel\Sanctum › …
GET|HEAD        tasks .................... tasks.index › TaskController@index
POST            tasks .................... tasks.store › TaskController@store
GET|HEAD        tasks/create ........... tasks.create › TaskController@create
GET|HEAD        tasks/{task} ............... tasks.show › TaskController@show
PUT|PATCH       tasks/{task} ........... tasks.update › TaskController@update
DELETE          tasks/{task} ......... tasks.destroy › TaskController@destroy
GET|HEAD        tasks/{task}/edit .......... tasks.edit › TaskController@edit
```

### №4. Шаблонизация с использованием Blade

#### №4.1. Создание макета страницы

1. Создайте макет основных страниц `layouts/app.blade.php` с общими элементами страницы:
   1. Заголовок страницы;
   2. Меню навигации;
   3. Контент страницы.
2. Используйте директиву `@yield` для определения области, в которую будут вставляться содержимое различных страниц.

```php
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<meta name="viewport" content="width=device-width, initial-scale=1.0">
	<link href="https://fonts.googleapis.com/css?family=Poppins:regular,600,700" rel="stylesheet" />
	@vite(['public/scss/style.scss'])
	<title>@yield('title') | Todo App</title>
</head>
<body>
	<header class="header">
		<x-header />
	</header>
	<main>
		@yield('content')
	</main>
</body>
</html>
```

#### №4.2. Использование шаблонов Blade

1. Создайте представление для главной страницы `home.blade.php` с использованием макета `layouts/app.blade.php` в каталоге `resources/views`.
2. На главной странице должно быть:
   1. **Приветственное сообщение**: заголовок и краткое описание приложения, например "To-Do App для команд".
   2. **Навигация**: ссылки на основные разделы, такие как:
      - Список задач;
      - Создание задачи.
   3. **Информация о приложении**: краткое описание назначения приложения и его основных функций.

```php
@extends('layouts.app')

@section('title', 'Home')

@section('content')
    <section class="home">
        <div class="home-container">
            <div class="home__body">
                <h1 class="home__title">Organize your tasks, take control of your life!</h1>
                <div class="home-actions">
                    <a class="button button-gray" href="{{ route('tasks.index') }}">Task List</a>
                    <button class="button button-gray">Creating a task</button>
                </div>
                @if($lastTask)
                <h2>Последняя созданная задача</h2>
                <x-task :task="$lastTask" />
            @else
                <p>Задач пока нет.</p>
            @endif
            </div>
        </div>
    </section>
@endsection

```

3. Создайте представление для страницы "О нас" — `about.blade.php` с использованием макета `layouts/app.blade.php` в каталоге `resources/views`.

4. Создайте представления для задач со следующими шаблонами в каталоге `resources/views/tasks`:
   - `index.blade.php` — список задач;
   - `show.blade.php` — отображение задачи;
   - ...

```php
@extends('layouts.app')

@section('title', 'About')

@section('content')
    <section class="about">
        <div class="about-container">
            <div class="about-body">
                <div class="about-header">
                    <h2 class="about-title">What We do</h2>
                </div>
                <div class="about-content">
                    <div class="about-item">
                        <div class="about-item-icon">
                            <img src="{{ asset('assets/about/01.svg') }}" alt="Icon">
                        </div>
                        <h4 class="about-item-title">Organize Tasks</h4>
                        <p class="about-item-text">From concept to launch, we create stunning, user-centric websites that
                            elevate your brand and engage your audience.</p>
                    </div>
                    <div class="about-item">
                        <div class="about-item-icon">
                            <img src="{{ asset('assets/about/02.svg') }}" alt="Icon">
                        </div>
                        <h4 class="about-item-title">Boost Productivity</h4>
                        <p class="about-item-text">From concept to launch, we create stunning, user-centric websites that
                            elevate your brand and engage your audience.</p>
                    </div>
                    <div class="about-item">
                        <div class="about-item-icon">
                            <img src="{{ asset('assets/about/03.svg') }}" alt="Icon">
                        </div>
                        <h4 class="about-item-title">Simplify Planning</h4>
                        <p class="about-item-text">From concept to launch, we create stunning, user-centric websites that
                            elevate your brand and engage your audience.</p>
                    </div>
                    <div class="about-item">
                        <div class="about-item-icon">
                            <img src="{{ asset('assets/about/04.svg') }}" alt="Icon">
                        </div>
                        <h4 class="about-item-title">Achieve Goals</h4>
                        <p class="about-item-text">From concept to launch, we create stunning, user-centric websites that
                            elevate your brand and engage your audience.</p>
                    </div>
                </div>
            </div>
        </div>
    </section>
@endsection

```

5. Отрендерите список задач на странице `index.blade.php` с использованием статических данных, передаваемых из контроллера с помощью директивы `@foreach`.

**Примечание**: Поскольку мы пока не работаем с базой данных и моделями, используйте статические данные, передаваемые из контроллера в шаблон, для отображения информации о задачах. Логика обработки данных пока не требуется.

```php
@extends('layouts.app')

@section('title', 'List of Tasks')

@section('content')
    <section class="tasks">
        <div class="tasks-container">
            <div class="task-body">
                <div class="tasks-header">
                    <h2>List of Tasks</h2>
                </div>
                <div class="tasks-content">
                    @if (count($tasks) > 0)
                        @foreach ($tasks as $task)
                            <a class="tasks-item" href="{{ route('tasks.show', $task['id']) }}">
                                <h4 class="tasks-item-title">{{ $task['title'] }}</h4>
                                <p class="tasks-item-desc">{{ $task['description'] }}</p>
                                <p class="tasks-item-assignee">{{ $task['assignee'] }}</p>
                            </a>
                            </li>
                        @endforeach
                    @else
                        <p>Задач нет</p>
                    @endif
                </div>
            </div>
        </div>
    </section>
@endsection

```

#### №4.3. Анонимные компоненты Blade

1. Создайте анонимный компонент для отображения `header`. Используйте созданный компонент в макете `layouts/app.blade.php`.

```php
<div class="header-container">
	<div class="header-logo">
		<img src="{{ asset('assets/logo.svg')}}" alt="">
		<div class="header-logo-text">ToDo App</div>
	</div>
	<nav class="header-nav">
		<ul>
			<li><a href="{{ route('home')}}">Home</a></li>
			<li><a href="{{ route('about')}}">About</a></li>
			<li><a href="{{ route('about')}}">How it Works</a></li>
			<li><a href="{{ route('about')}}">Services</a></li>
		</ul>
	</nav>
	<div class="header-actions">
		<button class="button">Sign Up</button>
	</div>
</div>
```

2. Создайте анонимный компонент для отображения задачи:

   1. Компонент должен быть простым и использовать передаваемые параметры с помощью директивы `@props`. Это сделает шаблоны более гибкими и переиспользуемыми на различных страницах.
   2. Компонент должен отображать информацию о задаче:
      1. Название задачи;
      2. Описание задачи;
      3. Дата создания задачи;
      4. Дата обновления задачи;
      5. Действия над задачей (редактирование, удаление);
      6. Статус задачи (выполнена/не выполнена);
      7. Приоритет задачи (низкий/средний/высокий);
      8. Исполнитель задачи (Assignment), то есть имя пользователя, которому назначена задача.

3. Отобразите созданный компонент задачи на странице `show.blade.php` с использованием передаваемых параметров.

```php
@extends('layouts.app')

@section('title', 'Task -' . $task['title'])

@section('content')
    <section class="task">
        <div class="task-container">
            <h1>{{ $task['title'] }}</h1>
            <p>{{ $task['description'] }}</p>
            <p><strong>Дата создания:</strong> {{ $task['created_at'] }}</p>
            <p><strong>Дата обновления:</strong> {{ $task['updated_at'] }}</p>
            <p><strong>Приоритет:</strong> {{ $task['priority'] }}</p>
            <p><strong>Статус:</strong> {{ $task['status'] ? 'Выполнена' : 'Не выполнена' }}</p>
            <p><strong>Исполнитель:</strong> {{ $task['assignee'] }}</p>
        </div>
    </section>
@endsection

```

#### №4.4. Стилизация страниц

1. Добавьте стили для страниц с использованием CSS или препроцессоров (например, Sass или Less).
2. Создайте файл стилей `app.css` в каталоге `public/css` и подключите его к макету `layouts/app.blade.php`.
3. Добавьте стили для элементов страницы, таких как заголовки, меню навигации, кнопки, формы и т. д.
4. При желании можно использовать библиотеки стилей, такие как `Bootstrap` или `Tailwind CSS`.

```scss
@import './base/reset';
@import './base/variables';
@import './base/global';

//! Components
@import './components/header';

//! Pages
@import './pages/home';
@import './pages/about';

//! Tasks
@import './tasks/index';
@import './tasks/show';
```

## Контрольные вопросы

1. **Что такое ресурсный контроллер в Laravel и какие маршруты он создает?**  
   **Ресурсный контроллер** в Laravel — это контроллер, который автоматически создает маршруты для операций CRUD: `index`, `create`, `store`, `show`, `edit`, `update`, и `destroy`.

2. **Объясните разницу между ручным созданием маршрутов и использованием ресурсного контроллера.**  
   **Ручное создание маршрутов** требует явного указания каждого маршрута для действий контроллера. **Ресурсный контроллер** автоматически создает стандартные маршруты для всех операций CRUD.

3. **Какие преимущества предоставляет использование анонимных компонентов Blade?**  
   **Анонимные компоненты Blade** позволяют писать более чистый и читаемый код, так как не требуют явного указания параметров в компонентах и обеспечивают инкапсуляцию логики.

4. **Какие методы HTTP-запросов используются для выполнения операций CRUD?**  
   Методы HTTP-запросов для **CRUD**:
   - **Create**: `POST`
   - **Read**: `GET`
   - **Update**: `PUT` или `PATCH`
   - **Delete**: `DELETE`

## Скриншоты с веб-браузера

![Скриншот страницы](<../Lab Work 2 - HTTP requests and templating in Laravel/images/01.png>)
*Скриншот главной страницы*

![Скриншот страницы](<../Lab Work 2 - HTTP requests and templating in Laravel/images/02.png>)
*Скриншот страницы "О Нас"*

![Скриншот страницы](<../Lab Work 2 - HTTP requests and templating in Laravel/images/03.png>)
*Скриншот списков задач*

![Скриншот страницы](<../Lab Work 2 - HTTP requests and templating in Laravel/images/04.png>)
*Скриншот самой задачи*