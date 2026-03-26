# Атрибут `On`

Атрибут `#[On]` позволяет компоненту прослушивать события и выполнять метод при их отправке.

## Базовое использование

Примените атрибут `#[On]` к любому методу, который должен вызываться при отправке события:

```php hl_lines="7" title="resources/views/components/⚡dashboard.blade.php"
<?php

use Livewire\Attributes\On;
use Livewire\Component;

new class extends Component {
    #[On('post-created')]
    public function updatePostList($title)
    {
        session()->flash('status', "Создан новый пост: {$title}");
    }
};
```

Когда другой компонент отправит событие `post-created`, метод `updatePostList()` будет вызван автоматически.

## Отправка событий

Для отправки события, которое активирует слушателей, используйте метод `dispatch()`:

```php hl_lines="13" title="resources/views/components/post/⚡create.blade.php"
<?php

use Livewire\Component;
use App\Models\Post;

new class extends Component {
    public $title = '';

    public function save()
    {
        $post = Post::create(['title' => $this->title]);

        $this->dispatch('post-created', title: $post->title);

        return redirect('/posts');
    }
};
```

Событие `post-created` активирует любые методы, помеченные атрибутом `#[On('post-created')]`.

## Передача данных слушателям

События могут передавать данные в виде именованных параметров:

```php
<?php

// Отправка с несколькими параметрами
$this->dispatch('post-updated', id: $post->id, title: $post->title);
```

```php
<?php

// Прослушивание и получение параметров
#[On('post-updated')]
public function handlePostUpdate($id, $title)
{
    // Используйте $id и $title...
}
```

## Динамические имена событий

Вы можете использовать свойства компонента в именах событий для ограниченного прослушивания:

```php hl_lines="10" title="resources/views/components/post/⚡show.blade.php"
<?php

use Livewire\Attributes\On;
use Livewire\Component;
use App\Models\Post;

new class extends Component {
    public Post $post;

    #[On('post-updated.{post.id}')]
    public function refreshPost()
    {
        $this->post->refresh();
    }
};
```

Если `$post->id` равен `3`, компонент будет слушать только события `post-updated.3`, игнорируя обновления других постов.

## Несколько слушателей событий

Один метод может прослушивать несколько событий:

```php
<?php

#[On('post-created')]
#[On('post-updated')]
#[On('post-deleted')]
public function refreshStats()
{
    // Обновить статистику при любом изменении поста
}
```

## Прослушивание событий браузера

Вы также можете прослушивать события браузера, отправленные из JavaScript:

```php
<?php

#[On('user-logged-in')]
public function handleUserLogin()
{
    // Обработка входа...
}
```

```javascript
// Из JavaScript
window.dispatchEvent(new CustomEvent('user-logged-in'));
```

## Альтернатива: Прослушивание в шаблоне

Вместо использования атрибута вы можете прослушивать события непосредственно на дочерних компонентах в шаблоне Blade:

```php
<?php

<livewire:post.edit @saved="$refresh" />
```

Это прослушивает событие `saved` от дочернего компонента `post.edit` и обновляет родительский компонент при его отправке.

Вы также можете вызывать конкретные методы:

```php
<?php

<livewire:post.edit @saved="handleSave($event.id)" />
```

## Когда использовать

Используйте `#[On]`, когда:

* Один компонент должен реагировать на действия в другом компоненте
* Реализуете уведомления или обновления в реальном времени
* Создаете слабосвязанные компоненты, которые взаимодействуют через события
* Прослушиваете события браузера или Laravel Echo
* Обновляете данные при возникновении внешних изменений

## Пример: Уведомления в реальном времени

Вот практический пример значка уведомлений, который прослушивает новые уведомления:

```php hl_lines="14 20" title="resources/views/components/⚡notification-bell.blade.php"
<?php

use Livewire\Attributes\On;
use Livewire\Component;

new class extends Component {
    public $unreadCount = 0;

    public function mount()
    {
        $this->unreadCount = auth()->user()->unreadNotifications()->count();
    }

    #[On('notification-sent')]
    public function incrementCount()
    {
        $this->unreadCount++;
    }

    #[On('notifications-read')]
    public function resetCount()
    {
        $this->unreadCount = 0;
    }
};
?>

<button class="relative">
    <svg><!-- Иконка колокольчика --></svg>
    @if($unreadCount > 0)
        <span class="absolute -top-1 -right-1 bg-red-500 text-white rounded-full px-2 py-1 text-xs">
            {{ $unreadCount }}
        </span>
    @endif
</button>
```

Другие компоненты могут отправлять события для обновления счётчика уведомлений:

```php
<?php

// Из любого места вашего приложения
$this->dispatch('notification-sent');
```

## Узнать больше

Для получения дополнительной информации о событиях, отправке конкретным компонентам и интеграции с Laravel Echo см. [документацию по событиям](/essentials/events).

## Справочник

```php
<?php

#[On(
    string $event,
)]
```

| Параметр | Тип | По умолчанию | Описание |
|-----------|------|---------|-------------|
| `$event` | `string` | *обязательно* | Имя события для прослушивания |
