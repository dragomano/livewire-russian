# Директива `wire:show`

Директива Livewire `wire:show` упрощает показ и скрытие элементов на основе результата вычисления выражения.

Директива `wire:show` отличается от использования `@if` в Blade тем, что она переключает видимость элемента с помощью CSS (`display: none`), а не удаляет элемент полностью из DOM. Это означает, что элемент остаётся на странице, но скрыт, что обеспечивает более плавные переходы без необходимости запроса к серверу.

## Базовое использование

Вот практический пример использования `wire:show` для переключения модального окна «Создать пост»:

```php
<?php

use Livewire\Component;
use App\Models\Post;

class CreatePost extends Component
{
    public $showModal = false;

    public $content = '';

    public function save()
    {
        Post::create(['content' => $this->content]);

        $this->reset('content');

        $this->showModal = false;
    }
}
```

```html
<div>
    <button x-on:click="$wire.showModal = true">Новый пост</button>

    <div wire:show="showModal">
        <form wire:submit="save">
            <textarea wire:model="content"></textarea>

            <button type="submit">Сохранить пост</button>
        </form>
    </div>
</div>
```

При нажатии на кнопку «Создать новый пост» модальное окно появляется без запроса к серверу. После успешного сохранения поста модальное окно скрывается, а форма сбрасывается.

## Использование переходов

Вы можете комбинировать `wire:show` с переходами Alpine.js для создания плавных анимаций показа/скрытия. Поскольку `wire:show` только переключает CSS-свойство `display`, директивы `x-transition` из Alpine идеально с ним работают:

```html
<div>
    <button x-on:click="$wire.showModal = true">Новый пост</button>

    <div wire:show="showModal" x-transition.duration.500ms>
        <form wire:submit="save">
            <textarea wire:model="content"></textarea>
            <button type="submit">Сохранить пост</button>
        </form>
    </div>
</div>
```

Приведённые выше классы переходов Alpine.js создадут эффект затухания и масштабирования при показе и скрытии модального окна.

[Полная документация по x-transition →](https://alpinejs.dragomano.ru/directives/transition)

## Справочник

```php
wire:show="выражение"
```

Эта директива не имеет модификаторов.
