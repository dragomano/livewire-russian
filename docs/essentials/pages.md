# Страницы

Компоненты Livewire можно использовать как полноценные страницы, привязывая их напрямую к маршрутам. Это позволяет создавать целые страницы приложения с помощью компонентов Livewire, используя дополнительные возможности, такие как пользовательские макеты, заголовки страниц и параметры маршрута.

## Маршрутизация к компонентам

Чтобы привязать маршрут к компоненту, используйте метод `Route::livewire()` в файле `routes/web.php`:

```php
Route::livewire('/posts/create', 'pages::post.create');
```

Когда вы заходите по указанному URL, компонент будет отображён как полноценная страница с использованием макета вашего приложения.

## Макеты

Компоненты, отображаемые через маршруты, будут использовать файл макета вашего приложения. По умолчанию Livewire ищет макет с именем `layouts::app`, который находится по пути `resources/views/layouts/app.blade.php`.

Вы можете создать этот файл, если его ещё нет, выполнив следующую команду:

```shell
php artisan livewire:layout
```

Эта команда создаст файл с именем `resources/views/layouts/app.blade.php`.

Убедитесь, что вы создали Blade-файл по этому пути и добавили в него плейсхолдер `{{ $slot }}`:

```html title="resources/views/layouts/app.blade.php"

<!DOCTYPE html>
<html lang="{{ str_replace('_', '-', app()->getLocale()) }}">
    <head>
        <meta charset="utf-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">

        <title>{{ $title ?? config('app.name') }}</title>

        @vite(['resources/css/app.css', 'resources/js/app.js'])

        @livewireStyles
    </head>
    <body>
        {{ $slot }}

        @livewireScripts
    </body>
</html>
```

Вы можете настроить макет по умолчанию, обновив параметр конфигурации `component_layout` в файле `config/livewire.php`:

```php
'component_layout' => 'layouts::dashboard',
```

### Макеты, специфичные для компонента

Чтобы использовать другой макет для конкретного компонента, вы можете разместить атрибут `#[Layout]` над классом вашего компонента:

```php hl_lines="6"
<?php

use Livewire\Attributes\Layout;
use Livewire\Component;

new #[Layout('layouts::dashboard')] class extends Component {
	// ...
};
```

Кроме того, вы можете использовать метод `->layout()` внутри метода `render()` вашего компонента:

```php hl_lines="11"
<?php

use Livewire\Component;

new class extends Component {
    // ...

    public function render()
    {
        return $this->view()
            ->layout('layouts::dashboard');
    }
};
```

## Установка заголовка страницы

Присвоение уникальных заголовков каждой странице в вашем приложении полезно как для пользователей, так и для поисковых систем.

Чтобы установить пользовательский заголовок страницы для компонента-страницы, сначала убедитесь, что ваш файл макета содержит динамический заголовок:

```html
<head>
    <title>{{ $title ?? config('app.name') }}</title>
</head>
```

Далее, над классом вашего компонента Livewire добавьте атрибут `#[Title]` и передайте ему заголовок страницы:

```php hl_lines="6"
<?php

use Livewire\Attributes\Title;
use Livewire\Component;

new #[Title('Создать пост')] class extends Component {
    // ...
};
```

Это установит заголовок страницы для компонента. В данном примере заголовок страницы будет «Создать пост», когда компонент будет отображён.

Если вам нужно передать динамический заголовок, например, заголовок, который использует свойство компонента, вы можете применить метод `->title()` в цепочке внутри метода `render()` компонента:

```php hl_lines="6"
<?php

public function render()
{
    return $this->view()
        ->title('Создать пост');
}
```

## Установка содержимого дополнительных слотов макета

Если в вашем файле макета есть именованные слоты в дополнение к `$slot`, вы можете задать их содержимое в Blade-шаблоне вашего компонента, определяя `<x-slot>` за пределами корневого элемента. Например, если вы хотите задавать язык страницы индивидуально для каждого компонента, можно добавить динамический слот `$lang` в открывающий тег `<html>` вашего файла макета:

```html title="resources/views/layouts/app.blade.php"

<!DOCTYPE html>
<html lang="{{ str_replace('_', '-', $lang ?? app()->getLocale()) }}"> <!-- [tl! highlight] -->
    <head>
        <meta charset="utf-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">

        <title>{{ $title ?? config('app.name') }}</title>

        @vite(['resources/css/app.css', 'resources/js/app.js'])

        @livewireStyles
    </head>
    <body>
        {{ $slot }}

        @livewireScripts
    </body>
</html>
```

Затем, в шаблоне вашего компонента, определите элемент `<x-slot>` за пределами корневого элемента:

```html hl_lines="1"
<x-slot:lang>ru</x-slot> // Этот компонент на русском языке

<div>
    // Контент на русском языке...
</div>
```

## Доступ к параметрам маршрута

При работе с полностраничными компонентами вам может потребоваться получить доступ к параметрам маршрута внутри компонента Livewire.

Для примера сначала определите маршрут с параметром в файле `routes/web.php`:

```php
Route::livewire('/posts/{id}', 'pages::show-post');
```

Здесь мы определили маршрут с параметром `id`, который представляет ID поста.

Далее обновите ваш компонент Livewire, чтобы он принимал параметр маршрута в методе `mount()`:

```php hl_lines="9"
<?php

use App\Models\Post;
use Livewire\Component;

new class extends Component {
    public Post $post;

    public function mount($id)
    {
        $this->post = Post::findOrFail($id);
    }
};
```

В этом примере, поскольку имя параметра `$id` совпадает с параметром маршрута `{id}`, при посещении URL `/posts/1` Livewire автоматически передаст значение «1» как параметр `$id`.

## Использование привязки модели к маршруту

Привязка модели к маршруту в Laravel позволяет автоматически получать экземпляры Eloquent-моделей из параметров маршрута.

После определения маршрута с параметром-моделью в файле `routes/web.php`:

```php
Route::livewire('/posts/{post}', 'pages::show-post');
```

Теперь вы можете принимать параметр модели маршрута через метод `mount()` вашего компонента:

```php hl_lines="9"
<?php

use App\Models\Post;
use Livewire\Component;

new class extends Component {
    public Post $post;

    public function mount(Post $post)
    {
        $this->post = $post;
    }
};
```

Livewire понимает, что нужно использовать «привязку модели к маршруту», потому что перед параметром `$post` в методе `mount()` указан тип-хинт `Post`.

Как и раньше, вы можете сократить шаблонный код, просто опустив метод `mount()`:

```php hl_lines="7"
<?php

use Livewire\Component;
use App\Models\Post;

new class extends Component {
    public Post $post;
};
```

Свойство `$post` будет автоматически присвоено модели, привязанной через параметр маршрута `{post}`.

## Смотрите также

- **[Компоненты](/essentials/components)** — Узнайте о создании и организации компонентов
- **[Навигация](/features/navigate)** — Создавайте навигацию в стиле SPA между страницами
- **[Перенаправление](/features/redirecting)** — Перенаправляйте пользователей после отправки форм или выполнения действий
- **[Атрибут Layout](/php-attributes/attribute-layout)** — Указывайте макеты для полностраничных компонентов
- **[Атрибут Title](/php-attributes/attribute-title)** — Динамически устанавливайте заголовки страниц
