# Директива `wire:text`

`wire:text` — это директива, которая динамически обновляет текстовое содержимое элемента на основе свойства компонента или выражения. В отличие от использования синтаксиса `{{ }}` в Blade, `wire:text` обновляет содержимое без необходимости запроса к серверу для повторного рендеринга компонента.

Если вы знакомы с директивой `x-text` из Alpine, то они по сути одинаковы.

## Базовое использование

Вот пример использования `wire:text` для оптимистичного отображения обновлений свойства Livewire без ожидания запроса к серверу.

```php
<?php

use Livewire\Component;
use App\Models\Post;

class ShowPost extends Component
{
    public Post $post;

    public $likes;

    public function mount()
    {
        $this->likes = $this->post->like_count;
    }

    public function like()
    {
        $this->post->like();

        $this->likes = $this->post->fresh()->like_count;
    }
}
```

```html
<div>
    <button x-on:click="$wire.likes++" wire:click="like">❤️ Нравится</button>

    Нравится: <span wire:text="likes"></span>
</div>
```

При нажатии на кнопку `$wire.likes++` немедленно обновляет отображаемое количество через `wire:text`, в то время как `wire:click="like"` сохраняет изменение в базе данных в фоновом режиме.

Этот шаблон делает `wire:text` идеальным для создания оптимистичных UI в Livewire.

## Справочник

```php
wire:text="выражение"
```

Эта директива не имеет модификаторов.
