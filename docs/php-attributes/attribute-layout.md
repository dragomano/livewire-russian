# Атрибут `Layout`

Атрибут `#[Layout]` указывает, какой Blade-макет должен использовать полностраничный компонент, позволяя настраивать макеты для каждого компонента индивидуально.

## Базовое использование

Примените атрибут `#[Layout]` к полностраничному компоненту, чтобы использовать определенный макет:

```php hl_lines="8" title="resources/views/pages/posts/⚡index.blade.php"
<?php

use Livewire\Attributes\Layout;
use Livewire\Attributes\Computed;
use Livewire\Component;
use App\Models\Post;

new #[Layout('layouts::dashboard')] class extends Component {
    #[Computed]
    public function posts()
    {
        return Post::all();
    }
};
?>

<div>
    <h1>Посты</h1>
    @foreach ($this->posts as $post)
        <div wire:key="{{ $post->id }}">{{ $post->title }}</div>
    @endforeach
</div>
```

Этот компонент будет отрисован с использованием макета `resources/views/layouts/dashboard.blade.php` вместо макета по умолчанию.

## Макет по умолчанию

По умолчанию Livewire использует макет, указанный в файле `config/livewire.php`:

```php
'component_layout' => 'layouts::app',
```

Атрибут `#[Layout]` переопределяет это значение для конкретных компонентов.

## Передача данных в макеты

Вы можете передать дополнительные данные в макет, используя синтаксис массива:

```php hl_lines="3"
<?php

new #[Layout('layouts::dashboard', ['title' => 'Панель управления постами'])] class extends Component {
    // ...
};
```

В файле макета будет доступна переменная `$title`:

```html
<!DOCTYPE html>
<html>
<head>
    <title>{{ $title ?? 'Мое приложение' }}</title>
</head>
<body>
    {{ $slot }}
</body>
</html>
```

## Альтернатива: использование метода layout()

Вместо атрибута вы можете использовать метод `layout()` внутри метода `render()`:

```php hl_lines="9" title="resources/views/pages/posts/⚡index.blade.php"
<?php

use Livewire\Component;

new class extends Component {
    public function render()
    {
        return view('livewire.posts.index')
            ->layout('layouts::dashboard', ['title' => 'Посты']);
    }
};
```

Подход с атрибутом выглядит чище для однофайловых компонентов, которым не требуется метод `render()`.

## Использование разных макетов для каждой страницы

Распространённый паттерн — использование разных макетов для различных разделов вашего приложения:

```php
<?php

// Админ-панель
new #[Layout('layouts::admin')] class extends Component { }

// Маркетинговые страницы
new #[Layout('layouts::marketing')] class extends Component { }

// Личный кабинет
new #[Layout('layouts::dashboard')] class extends Component { }
```

## Когда использовать

Используйте `#[Layout]`, когда:

* У вас в приложении несколько макетов (админка, маркетинг, дашборд и т. д.)
* Для конкретной страницы нужен макет, отличный от макета по умолчанию
* Вы создаете полностраничный компонент (а не обычный вложенный компонент)
* Вы хотите держать конфигурацию макета рядом с определением компонента

!!! info "Только для полностраничных компонентов"
    Атрибут `#[Layout]` применяется только к полностраничным компонентам. Обычные компоненты, которые рендерятся внутри других представлений, макеты не используют.

## Узнать больше

Для получения дополнительной информации о полностраничных компонентах и макетах см. [документацию по страницам](/essentials/pages#макеты).

## Справочник

```php
<?php

#[Layout(
    string $name,
    array $params = [],
)]
```

| Параметр | Тип | По умолчанию | Описание |
|-----------|------|---------|-------------|
| `$name` | `string` | *обязательно* | Имя Blade-макета для использования |
| `$params` | `array` | `[]` | Дополнительные данные для передачи в макет |
