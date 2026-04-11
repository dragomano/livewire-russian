# Атрибут `Authorize`

Атрибут `#[Authorize]` интегрирует систему Gate из Laravel напрямую в ваши действия Livewire. Он гарантирует, что действие будет выполнено только в том случае, если у пользователя есть необходимые разрешения, в противном случае выбрасывая ответ `403 Forbidden`.

## Базовое использование

Примените атрибут `#[Authorize]` к любому методу действия. Передайте название возможности и необязательный аргумент:

```php hl_lines="10" title="resources/views/components/post/⚡edit.blade.php"
<?php

use Livewire\Attributes\Authorize;
use Livewire\Component;
use App\Models\Post;

new class extends Component {
    public Post $post;

    #[Authorize('update', 'post')]
    public function save()
    {
        $this->post->save();
    }
};
```

```html
<button wire:click="save">
    Обновить пост
</button>
```

При вызове `save()` Livewire автоматически проверяет, авторизован ли текущий пользователь для выполнения `update` модели `$post`, сохранённой в компоненте.

## Разрешение аргументов

Атрибут разрешает объект для проверки авторизации в следующем порядке:

* **Без аргумента** — проверяет простую возможность, не требующую модели (например, `#[Authorize('view-dashboard')]`).
* **Строка класса** — полезно для разрешений `create`, когда экземпляр ещё не существует (например, `#[Authorize('create', Post::class)]`).
* **Параметр метода** — разрешает аргумент из собственных параметров метода.
* **Свойство компонента** — ищет свойство в компоненте, соответствующее имени аргумента (например, `public Post $post`).

### Разрешение из параметров метода

При авторизации на основе параметра метода вы должны указать тип параметра, чтобы Livewire знал, какую модель разрешить:

```php hl_lines="8-9" title="resources/views/components/⚡comment-manager.blade.php"
<?php

use Livewire\Attributes\Authorize;
use Livewire\Component;
use App\Models\Comment;

new class extends Component {
    #[Authorize('delete', 'comment')]
    public function deleteComment(Comment $comment)
    {
        $comment->delete();
    }
};
```

!!! note "Примечание"
    Если модель разрешается через параметр метода, требуется указание типа (например, `Comment $comment`). Без него Livewire не сможет определить, какую модель разрешить, и проверка авторизации завершится неудачей.

## Множественные проверки

Атрибут можно использовать многократно, что позволяет применять несколько проверок авторизации к одному методу:

```php
<?php

#[Authorize('create', Post::class)]
#[Authorize('update', 'post')]
public function save()
{
    // Обе проверки должны пройти успешно...
}
```

## Когда НЕ стоит использовать

!!! warning "Предупреждение"
    Атрибут `#[Authorize]` защищает только выполнение действия на стороне сервера. Он не скрывает элементы интерфейса в вашем Blade-шаблоне.

Для скрытия кнопок, которые пользователь не может использовать, следует использовать директивы `@can` в Blade:

```html
@can('update', $post)
    <button wire:click="save">Сохранить</button>
@endcan
```

## Подробнее

Для получения дополнительной информации об определении возможностей и политик см. [документацию Laravel по авторизации](https://laravel.com/docs/authorization).
