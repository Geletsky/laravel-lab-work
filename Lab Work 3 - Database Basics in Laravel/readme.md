# Лабораторная работа №3. Основы работы с базами данных в Laravel

## Цель работы

Познакомиться с основными принципами работы с базами данных в Laravel. Научиться создавать миграции, модели и сиды на основе веб-приложения `To-Do App`.

### №1. Подготовка к работе

1. Устанавливаю СУБД SQLite.

2. Создаю новую базу данных для приложения todo_app.

3. Настраиваю переменные окружения в файле .env для подключения к базе данных:

```env
DB_CONNECTION=sqlite
DB_DATABASE=database/db.sqlite
```

### №2. Создание моделей и миграций

1. Создаю модель `Category` - категория задачи

```bash
php artisan make:model Category -m
```

2. Определяю структуры таблицы `category` в миграции:

```php
Schema::create('categories', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->text('description')->nullable();
            $table->timestamps();
        });
```

3. Создаю модель `Task` - категория задачи

```bash
php artisan make:model Task -m
```

4. Определяю структуры таблицы `category` в миграции:

```php
Schema::create('tasks', function (Blueprint $table) {
            $table->id();
            $table->string('title');
            $table->text('description')->nullable();
            $table->timestamps();
        });
```

5. Создаю модель `Tag` - категория задачи

```bash
php artisan make:model Tag -m
```

6. Определяю структуры таблицы `tag` в миграции:

```php
Schema::create('tags', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->timestamps();
        });
```

7. Добавляю поле `$fillable` в модели `Task`, `Category` и `Tag` для массового заполнения данных.

```php
// Category
protected $fillable = ['name', 'description'];

// Tag
protected $fillable = ['name'];

// Task
protected $fillable = ['title', 'description'];
```

8. Запускаю миграцию для создания таблицы в базе данных:

```bash
php artisan migrate
```

### №3. Связь между таблицами

1. Создаю миграцию для добавления поля `category_id` в таблицу `task`

```bash
php artisan make:migration add_category_id_to_tasks_table --table=tasks
```

- Определяю структуру поля `category_id` и добавляю внешний ключ для связи с таблицей `category`.

```php
Schema::table('tasks', function (Blueprint $table) {
    $table->foreignId('category_id')->constrained()->onDelete('cascade');
});
```

2. Создаю промежуточную таблицу для связи многие ко многим между задачами и тегами:

```bash
php artisan make:migration create_task_tag_table
```

3. Определяю соответствующие структуры таблицы в миграции.

```php
Schema::create('task_tag', function (Blueprint $table) {
            $table->foreignId('task_id')->constrained()->onDelete('cascade');
            $table->foreignId('tag_id')->constrained()->onDelete('cascade');
        });
```

4. Запускаю миграцию для создания таблицы в базе данных.

```bash
php artisan migrate
```

### №4. Связи между моделями

1. Добавляю отношения в модель `Category` (Категория может иметь много задач)

```php
public function tasks()
{
    return $this->hasMany(Task::class);
}
```

2. Добавляю отношения в модель `Task`

```php
		public function category()
    {
        return $this->belongsTo(Category::class);
    }

    public function tags()
    {
        return $this->belongsToMany(Tag::class);
    }
```

3. Добавляю отношения в модель `Tag` (Тег может быть прикреплен к многим задачам)

```php
		public function tasks()
    {
        return $this->belongsToMany(Task::class);
    }
```

### №5. Создание фабрик и сидов

1. Создаю фабрику для модели `Category`:

```bash
php artisan make:factory CategoryFactory --model=Category
```

- Определяю структуру данных для генерации категорий.

```php
return [
        'name' => $this->faker->word,
        'description' => $this->faker->sentence,
    ];
```

2. Создаю фабрику для модели `Task`.

```bash
php artisan make:factory TaskFactory --model=Task
```

3. Создаю фабрику для модели `Tag`.

```bash
php artisan make:factory TagFactory --model=Tag
```

4. Создаю сиды (seeders) для заполнения таблиц начальными данными для моделей: `Category`, `Task`, `Tag`.

```bash
php artisan make:seeder CategorySeeder
php artisan make:seeder TaskSeeder
php artisan make:seeder TagSeeder
```

5. Обновляю файл `DatabaseSeeder` для запуска сидов и запустите их:

```php
$this->call([
    CategorySeeder::class,
    TaskSeeder::class,
    TagSeeder::class,
]);
```

```bash
php artisan db:seed
```

### №6. Работа с контроллерами и представлениями

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use App\Models\Task;
use App\Models\Category;
use App\Models\Tag;

class TaskController extends Controller
{
    public function index()
    {
        // Получаем все задачи из базы данных с их категориями и тегами
        $tasks = Task::with('category', 'tags')->get();

        // Возвращаем представление с переданными задачами
        return view('tasks.index', compact('tasks'));
    }

    public function create()
    {
        // Получаем все категории для отображения в форме создания задачи
        $categories = Category::all();
        // Возвращаем представление с формой создания задачи
        return view('tasks.create', compact('categories'));
    }

    public function store(Request $request)
    {
        // Валидация данных
        $validatedData = $request->validate([
            'title' => 'required|string|max:255',
            'description' => 'nullable|string',
            'category_id' => 'required|exists:categories,id', // Убедитесь, что категория существует
            'tags' => 'nullable|array', // Проверяем, что теги - это массив
        ]);

        // Создание новой задачи
        $task = Task::create([
            'title' => $validatedData['title'],
            'description' => $validatedData['description'],
            'category_id' => $validatedData['category_id'],
        ]);

        // Привязываем теги к задаче, если они указаны
        if (isset($validatedData['tags'])) {
            $task->tags()->attach($validatedData['tags']);
        }

        // Перенаправление или возврат ответа
        return redirect()->route('tasks.index')->with('success', 'Task created successfully!');
    }

    public function show($id)
    {
        // Получаем задачу по ее ID с жадной загрузкой связанных моделей
        $task = Task::with('category', 'tags')->findOrFail($id);

        // Возвращаем представление с полученной задачей
        return view('tasks.show', compact('task'));
    }

    public function edit($id)
    {
        // Получаем задачу по ее ID
        $task = Task::findOrFail($id);
        // Получаем все категории для отображения в форме редактирования
        $categories = Category::all();
        // Возвращаем представление с формой редактирования задачи
        return view('tasks.edit', compact('task', 'categories'));
    }

    public function update(Request $request, $id)
    {
        // Валидация данных
        $validatedData = $request->validate([
            'title' => 'required|string|max:255',
            'description' => 'nullable|string',
            'category_id' => 'required|exists:categories,id',
            'tags' => 'nullable|array',
        ]);

        // Находим задачу по ID и обновляем ее
        $task = Task::findOrFail($id);
        $task->update($validatedData);

        // Обновляем теги
        if (isset($validatedData['tags'])) {
            $task->tags()->sync($validatedData['tags']); // Обновляем теги
        }

        // Перенаправление или возврат ответа
        return redirect()->route('tasks.index')->with('success', 'Task updated successfully!');
    }

    public function destroy($id)
    {
        // Находим задачу по ID и удаляем ее
        $task = Task::findOrFail($id);
        $task->delete();

        // Перенаправление или возврат ответа
        return redirect()->route('tasks.index')->with('success', 'Task deleted successfully!');
    }
}
```

## Контрольные вопросы

Вот краткие ответы на ваши контрольные вопросы:

### 1. Что такое миграции и для чего они используются?

**Миграции** — это механизм в фреймворках, таких как Laravel, который позволяет управлять изменениями в структуре базы данных с помощью кода. Они используются для:

- **Версионирования**: Миграции позволяют отслеживать изменения в структуре базы данных и откатывать их при необходимости.
- **Согласованности**: Обеспечивают, что все разработчики в команде имеют одинаковую структуру базы данных.
- **Автоматизации**: Позволяют создавать и изменять таблицы, столбцы и индексы с помощью командной строки, что упрощает процесс разработки и развертывания.

### 2. Что такое фабрики и сиды, и как они упрощают процесс разработки и тестирования?

**Фабрики** — это инструменты для создания поддельных (fake) данных в вашем приложении. Они позволяют определять, как создаются экземпляры моделей, например, для тестирования.

**Сиды** (seeds) — это сценарии, которые заполняют базу данных начальными данными. Они используются для:

- **Тестирования**: Позволяют быстро заполнять базу данных тестовыми данными для проверки функциональности.
- **Разработки**: Упрощают начальную настройку приложения, позволяя разработчикам быстро получить рабочую версию базы данных с необходимыми данными.

### 3. Что такое ORM? В чем различия между паттернами DataMapper и ActiveRecord?

**ORM (Object-Relational Mapping)** — это метод, который позволяет работать с базами данных через объекты, не используя прямые SQL-запросы. ORM автоматически преобразует данные между реляционной базой данных и объектами в вашем приложении.

**Различия между DataMapper и ActiveRecord**:

- **ActiveRecord**: Каждый объект модели отвечает за свою логику базы данных (сохранение, удаление, загрузка и т.д.). Модель и база данных тесно связаны. Пример: Laravel Eloquent.
- **DataMapper**: Объекты и база данных слабо связаны. Логика работы с базой данных отделена от модели. Это позволяет легче менять структуру базы данных без изменения моделей. Пример: Doctrine.

### 4. В чем преимущества использования ORM по сравнению с прямыми SQL-запросами?

Преимущества ORM:

- **Простота использования**: ORM позволяет работать с базой данных с помощью методов объектов, что часто более интуитивно и понятно, чем написание SQL-запросов.
- **Портативность**: Легче менять тип базы данных, так как код не зависит от специфичных SQL-запросов.
- **Безопасность**: ORM автоматически защищает от SQL-инъекций, используя подготовленные запросы.
- **Упрощение разработки**: Меньше кода для написания, так как ORM генерирует SQL автоматически.

### 5. Что такое транзакции и зачем они нужны при работе с базами данных?

**Транзакции** — это последовательность операций, которые выполняются как единое целое. Если одна из операций не может быть выполнена, все изменения отменяются. Это позволяет обеспечить:

- **Целостность данных**: Гарантирует, что база данных остается в согласованном состоянии, даже если что-то пойдет не так.
- **Изоляцию**: Обеспечивает, что изменения, сделанные в одной транзакции, не будут видны другим операциям до завершения транзакции.
- **Устойчивость**: Изменения сохраняются даже в случае сбоя системы, если транзакция была успешно завершена.
