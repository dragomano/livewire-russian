# Атрибут `Title`

Атрибут `#[Title]` устанавливает заголовок страницы для полностраничных компонентов Livewire.

## Базовое использование

Примените атрибут `#[Title]` к полностраничному компоненту, чтобы установить его заголовок:

```php hl_lines="6" title="resources/views/pages/posts/⚡create.blade.php"
<?php

use Livewire\Attributes\Title;
use Livewire\Component;

new #[Title('Создать пост')] class extends Component {
    public $title = '';
    public $content = '';

    public function save()
    {
        // Сохранение поста...
    }
};
?>

<div>
    <h1>Создать новый пост</h1>

    <input type="text" wire:model="title" placeholder="Заголовок поста">
    <textarea wire:model="content" placeholder="Содержимое поста"></textarea>

    <button wire:click="save">Сохранить пост</button>
</div>
```

Вкладка браузера будет отображать «Создать пост» в качестве заголовка страницы.

## Конфигурация макета

Чтобы атрибут `#[Title]` работал, ваш файл макета должен включать переменную `$title`:

```html hl_lines="4" title="resources/views/components/layouts/app.blade.php"
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

`?? 'Мое приложение'` обеспечивает заголовок по умолчанию, если он не указан.

## Динамические заголовки

Для динамических заголовков, использующих свойства компонента, используйте метод `title()` в методе `render()`:

```php hl_lines="17" title="resources/views/pages/posts/⚡edit.blade.php"
<?php

use Livewire\Component;
use App\Models\Post;

new class extends Component {
    public Post $post;

    public function mount($id)
    {
        $this->post = Post::findOrFail($id);
    }

    public function render()
    {
        return $this->view()
            ->title("Редактирование: {$this->post->title}");
    }
};
?>

<div>
    <h1>Редактировать пост</h1>
    <!-- ... -->
</div>
```

Заголовок будет динамически включать название поста.

## Сочетание с макетами

Вы можете использовать `#[Title]` и `#[Layout]` вместе:

```php title="resources/views/pages/posts/⚡create.blade.php"
<?php

use Livewire\Attributes\Layout;
use Livewire\Attributes\Title;
use Livewire\Component;

new
#[Layout('layouts.admin')]
#[Title('Создать пост')]
class extends Component {
    // ...
};
```

Этот компонент будет использовать макет админки с заголовком «Создать пост».

## Когда использовать

Используйте `#[Title]`, когда:

* Создаете полностраничные компоненты
* Хотите чистое, декларативное определение заголовка
* Заголовок статичен или редко меняется
* Вы следуете рекомендациям по SEO

Используйте метод `title()`, когда:

* Заголовок зависит от свойств компонента
* Вам нужно вычислить заголовок динамически
* Заголовок меняется в зависимости от состояния компонента

## Пример: Страницы CRUD

Вот полный пример, показывающий заголовки для операций CRUD:

```php title="resources/views/pages/posts/⚡index.blade.php"
<?php

use Livewire\Attributes\Title;
use Livewire\Component;

new #[Title('Все посты')] class extends Component { };
```

```php title="resources/views/pages/posts/⚡create.blade.php"
<?php

use Livewire\Attributes\Title;
use Livewire\Component;

new #[Title('Создать пост')] class extends Component { };
```

```php title="resources/views/pages/posts/⚡edit.blade.php"
<?php

use Livewire\Component;
use App\Models\Post;

new class extends Component {
    public Post $post;

    public function render()
    {
        return $this->view()->title("Редактирование: {$this->post->title}");
    }
};
```

```php title="resources/views/pages/posts/⚡show.blade.php"
<?php

use Livewire\Component;
use App\Models\Post;

new class extends Component {
    public Post $post;

    public function render()
    {
        return $this->view()->title($this->post->title);
    }
};
```

Каждая страница имеет подходящий по смыслу заголовок, что делает интерфейс понятнее и помогает в SEO-продвижении.

## Рекомендации по SEO

Хорошие заголовки страниц имеют решающее значение для SEO:

* **Будьте описательны** — «Редактировать пост: Начало работы с Laravel» лучше, чем просто «Редактировать»
* **Будьте кратки** — стремитесь к 50-60 символам, чтобы избежать обрезки в результатах поиска
* **Включайте ключевые слова** — помогите поисковым системам понять содержание вашей страницы
* **Будьте уникальны** — каждая страница должна иметь свой заголовок

## Только для полностраничных компонентов

!!! info "Только для полностраничных компонентов"
    Атрибут `#[Title]` работает только для полностраничных компонентов, доступ к которым осуществляется через маршруты. Обычные компоненты, отрисованные внутри других представлений, не используют заголовки — они наследуют заголовок родительской страницы.

## Узнать больше

Для получения дополнительной информации о полностраничных компонентах, макетах и маршрутизации см. [документацию по страницам](/essentials/pages#установка-заголовка-страницы).

## Справочник

```php
<?php

#[Title(
    string $content,
)]
```

| Параметр | Тип | По умолчанию | Описание |
|-----------|------|---------|-------------|
| `$content` | `string` | *обязательно* | Текст для отображения в строке заголовка браузера |
