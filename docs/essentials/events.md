# События

Livewire предлагает мощную систему событий, которую вы можете использовать для общения между различными компонентами на странице. Поскольку под капотом используются браузерные события, вы также можете использовать систему событий Livewire для взаимодействия с компонентами Alpine или даже с обычным, чистым JavaScript.

Чтобы вызвать событие, вы можете использовать метод `dispatch()` из любого места внутри вашего компонента и прослушивать это событие из любого другого компонента на странице.

## Отправка событий

Чтобы отправить событие из компонента Livewire, вы можете вызвать метод `dispatch()`, передав ему имя события и любые дополнительные данные, которые вы хотите отправить вместе с событием.

Ниже приведён пример отправки события `post-created` из компонента `post.create`:

```php hl_lines="10" title="resources/views/components/post/⚡create.blade.php"
<?php

use Livewire\Component;

new class extends Component {
    public function save()
    {
        // ...

        $this->dispatch('post-created');
    }
};
```

В этом примере, когда вызывается метод `dispatch()`, будет отправлено событие `post-created`, и каждый другой компонент на странице, который прослушивает это событие, получит уведомление.

Вы можете передать дополнительные данные вместе с событием, указав их в качестве второго параметра метода `dispatch()`:

```php
<?php

$this->dispatch('post-created', title: $post->title);
```

## Прослушивание событий

Чтобы прослушивать событие в компоненте Livewire, добавьте атрибут `#[On]` над методом, который должен быть вызван при отправке указанного события:

!!! warning "Убедитесь, что вы импортировали классы атрибутов"
    Обязательно импортируйте классы атрибутов. Например, для атрибута `#[On()]` ниже требуется следующий импорт: `use Livewire\Attributes\On;`.

```php hl_lines="4 7" title="resources/views/components/⚡dashboard.blade.php"
<?php

use Livewire\Component;
use Livewire\Attributes\On;

new class extends Component {
    #[On('post-created')]
    public function updatePostList($title)
    {
        // ...
    }
};
```

Теперь, когда событие `post-created` отправляется из компонента `post.create`, будет инициирован сетевой запрос, и будет вызвано действие `updatePostList()`.

Как вы можете видеть, дополнительные данные, отправленные вместе с событием, будут переданы в действие в качестве его первого аргумента.

### Прослушивание динамических имён событий

Иногда вам может понадобиться динамически генерировать имена слушателей событий во время выполнения, используя данные из вашего компонента.

Например, если вы хотите ограничить действие слушателя события конкретной моделью Eloquent, вы можете добавить ID модели к имени события при отправке следующим образом:

```php hl_lines="10" title="resources/views/components/post/⚡edit.blade.php"
<?php

use Livewire\Component;

new class extends Component {
    public function update()
    {
        // ...

        $this->dispatch("post-updated.{$post->id}");
    }
};
```

А затем прослушивать это конкретное событие для данной модели:

```php hl_lines="3 10" title="resources/views/components/post/⚡show.blade.php"
<?php

use Livewire\Attributes\On;
use Livewire\Component;
use App\Models\Post;

new class extends Component {
    public Post $post;

    #[On('post-updated.{post.id}')]
    public function refreshPost()
    {
        // ...
    }
};
```

Если бы вышеуказанная модель `$post` имела ID равный `3`, то метод `refreshPost()` был бы вызван только событием с именем: `post-updated.3`.

### Прослушивание событий от конкретных дочерних компонентов

Livewire позволяет прослушивать события непосредственно на отдельных дочерних компонентах в вашем Blade-шаблоне следующим образом:

```html
<div>
    <livewire:edit-post @saved="$refresh">

    <!-- ... -->
</div>
```

В вышеописанном сценарии, если дочерний компонент `edit-post` отправит событие `saved`, то в родительском компоненте будет вызван метод `$refresh`, и родительский компонент обновится.

Вместо передачи `$refresh` вы можете передать любой метод, который обычно использовали бы, например, с `wire:click`. Вот пример вызова метода `close()`, который, например, может закрывать модальное окно:

```html
<livewire:edit-post @saved="close">
```

Если дочерний компонент отправил параметры вместе с событием, например `$this->dispatch('saved', postId: 1)`, вы можете передать эти значения в метод родительского компонента, используя следующий синтаксис:

```html
<livewire:edit-post @saved="close($event.detail.postId)">
```

## Использование JavaScript для взаимодействия с событиями

Система событий Livewire становится намного мощнее, когда вы взаимодействуете с ней из JavaScript внутри вашего приложения. Это открывает возможность для любого другого JavaScript-кода в вашем приложении общаться с компонентами Livewire на странице.

### Прослушивание событий внутри скриптов компонента

Вы можете легко прослушивать событие `post-created` внутри шаблона вашего компонента из тега `<script>` следующим образом:

```html
<script>
    this.$on('post-created', () => {
        //
    });
</script>
```

Приведённый выше фрагмент кода будет прослушивать событие `post-created` от компонента, в котором он зарегистрирован. Если компонент больше не присутствует на странице, слушатель события перестанет срабатывать.

[Подробнее об использовании JavaScript внутри компонентов Livewire →](/advanced/javascript#использование-javascript-в-компонентах-livewire)

### Отправка событий из скриптов компонента

Кроме того, вы можете отправлять события из тега `<script>` внутри компонента следующим образом:

```html
<script>
    this.$dispatch('post-created');
</script>
```

Когда приведённый выше скрипт выполняется, событие `post-created` будет отправлено в компонент, внутри которого оно определено.

Чтобы отправить событие **только** в тот компонент, где находится скрипт, и не в другие компоненты на странице (предотвращая «всплытие» события), вы можете использовать метод `dispatchSelf()`:

```js
this.$dispatchSelf('post-created');
```

Вы можете передать любые дополнительные параметры в событие, передав объект в качестве второго аргумента методу `dispatch()`:

```html
<script>
    this.$dispatch('post-created', { refreshPosts: true });
</script>
```

Теперь вы можете получить доступ к этим параметрам события как из вашего класса Livewire, так и из других JavaScript-слушателей событий.

Вот пример получения параметра `refreshPosts` внутри класса Livewire:

```php
<?php

use Livewire\Attributes\On;

// ...

#[On('post-created')]
public function handleNewPost($refreshPosts = false)
{
    //
}
```

Вы также можете получить доступ к параметру `refreshPosts` из JavaScript-слушателя событий через свойство `detail` объекта события:

```html
<script>
    this.$on('post-created', (event) => {
        let refreshPosts = event.detail.refreshPosts

        // ...
    });
</script>
```

[Подробнее об использовании JavaScript внутри компонентов Livewire →](/advanced/javascript#использование-javascript-в-компонентах-livewire)

### Прослушивание событий Livewire из глобального JavaScript

Кроме того, вы можете прослушивать события Livewire глобально, используя `Livewire.on` из любого скрипта в вашем приложении:

```html
<script>
    document.addEventListener('livewire:init', () => {
       Livewire.on('post-created', (event) => {
           //
       });
    });
</script>
```

Приведённый выше фрагмент кода будет прослушивать событие `post-created`, отправленное из любого компонента на странице.

Если вы захотите удалить этот слушатель событий по какой-либо причине, вы можете сделать это с помощью возвращаемой функции `cleanup`:

```html
<script>
    document.addEventListener('livewire:init', () => {
        let cleanup = Livewire.on('post-created', (event) => {
            //
        });

        // Вызов `cleanup()` отменит регистрацию вышеуказанного слушателя событий...
        cleanup();
    });
</script>
```

## События в Alpine

Поскольку события Livewire под капотом являются обычными браузерными событиями, вы можете использовать Alpine для их прослушивания или даже для их отправки.

### Прослушивание событий Livewire в Alpine

Например, мы можем легко прослушивать событие `post-created` с помощью Alpine:

```html
<div x-on:post-created="..."></div>
```

Приведённый выше фрагмент кода будет прослушивать событие `post-created` от любых компонентов Livewire, которые являются дочерними по отношению к HTML-элементу, на который назначена директива `x-on`.

Чтобы прослушивать событие от любого компонента Livewire на всей странице, вы можете добавить модификатор `.window` к слушателю:

```html
<div x-on:post-created.window="..."></div>
```

Если вы хотите получить доступ к дополнительным данным, которые были отправлены вместе с событием, вы можете сделать это с помощью `$event.detail`:

```html
<div x-on:post-created="notify('New post: ' + $event.detail.title)"></div>
```

Документация Alpine предоставляет дополнительную информацию о [прослушивании событий](https://alpinejs.dragomano.ru/directives/on).

### Отправка событий Livewire из Alpine

Любое событие, отправленное из Alpine, может быть перехвачено компонентом Livewire.

Например, мы можем легко отправить событие `post-created` из Alpine:

```html
<button x-on:click="$dispatch('post-created')">...</button>
```

Подобно методу `dispatch()` в Livewire, вы можете передать дополнительные данные вместе с событием, указав их в качестве второго параметра метода:

```html
<button x-on:click="$dispatch('post-created', { title: 'Заголовок поста' })">...</button>
```

Чтобы узнать больше об отправке событий с помощью Alpine, обратитесь к [документации Alpine](https://alpinejs.dragomano.ru/magics/dispatch).

!!! tip "Возможно, события вам не понадобятся"
    Если вы используете события, чтобы вызвать поведение родительского компонента из дочернего, вместо этого вы можете напрямую вызвать действие из дочернего компонента, используя `$parent` в вашем Blade-шаблоне. Например:

    ```html
    <button wire:click="$parent.showCreatePostForm()">Создать пост</button>
    ```

    [Подробнее о $parent](/essentials/nesting#directly-accessing-the-parent-from-the-child).

## Отправка события напрямую в другой компонент

Если вы хотите использовать события для прямого общения между двумя компонентами на странице, можно применить модификатор `dispatch()->to()`.

Ниже приведён пример, как компонент `post.create` отправляет событие `post-created` напрямую в компонент `dashboard`, пропуская любые другие компоненты, которые могли бы прослушивать это конкретное событие:

```php title="resources/views/components/post/⚡create.blade.php"
<?php

use Livewire\Component;

new class extends Component {
    public function save()
    {
		// ...

		$this->dispatch('post-created')->to(component: Dashboard::class);
    }
};
```

## Отправка события компоненту самому себе

Используя модификатор `dispatch()->self()`, вы можете ограничить событие так, чтобы его перехватывал **только** тот компонент, из которого оно было инициировано:

```php title="resources/views/components/post/⚡create.blade.php"
<?php

use Livewire\Component;

new class extends Component {
    public function save()
    {
        // ...

        $this->dispatch('post-created')->to(self: true);
    }
};
```

## Отправка событий из Blade-шаблонов

Вы можете отправлять события прямо из ваших Blade-шаблонов, используя JavaScript-функцию `$dispatch`. Это удобно, когда вы хотите инициировать событие в ответ на действие пользователя, например, при клике по кнопке:

```html
<button wire:click="$dispatch('show-post-modal', { id: {{ $post->id }} })">
    EditPost
</button>
```

В этом примере, при клике по кнопке будет отправлено событие `show-post-modal` вместе с указанными данными.

Если вы хотите отправить событие напрямую в другой компонент, вы можете использовать JavaScript-функцию `$dispatchTo()`:

```html
<button wire:click="$dispatchTo('posts', 'show-post-modal', { id: {{ $post->id }} })">
    EditPost
</button>
```

В этом примере, при клике по кнопке событие `show-post-modal` будет отправлено напрямую в компонент `Posts`.

## Тестирование отправленных событий

Чтобы протестировать события, отправленные вашим компонентом, используйте метод `assertDispatched()` в тестах Livewire. Этот метод проверяет, что конкретное событие было отправлено в течение жизненного цикла компонента:

```php
<?php

namespace Tests\Feature;

use Illuminate\Foundation\Testing\RefreshDatabase;
use App\Livewire\CreatePost;
use Livewire\Livewire;

class CreatePostTest extends TestCase
{
    use RefreshDatabase;

    public function test_it_dispatches_post_created_event()
    {
        Livewire::test(CreatePost::class)
            ->call('save')
            ->assertDispatched('post-created');
    }
}
```

В этом примере тест проверяет, что событие `post-created` отправляется с указанными данными при вызове метода `save()` в компоненте `post.create`.

### Тестирование слушателей событий

Чтобы протестировать слушатели событий, вы можете отправлять события из тестовой среды и проверять, что ожидаемые действия выполняются в ответ на это событие:

```php
<?php

namespace Tests\Feature;

use Illuminate\Foundation\Testing\RefreshDatabase;
use App\Livewire\Dashboard;
use Livewire\Livewire;

class DashboardTest extends TestCase
{
    use RefreshDatabase;

    public function test_it_updates_post_count_when_a_post_is_created()
    {
        Livewire::test(Dashboard::class)
            ->assertSee('Постов создано: 0')
            ->dispatch('post-created')
            ->assertSee('Постов создано: 1');
    }
}
```

В этом примере тест отправляет событие `post-created`, а затем проверяет, что компонент `dashboard` корректно обрабатывает это событие и отображает обновлённое количество.

## События в реальном времени с использованием Laravel Echo

Livewire отлично сочетается с [Laravel Echo](https://laravel.com/docs/broadcasting#client-side-installation), чтобы обеспечить функциональность реального времени на ваших веб-страницах с использованием WebSockets.

!!! warning "Установка Laravel Echo — обязательное условие"
    Эта возможность предполагает, что у вас установлен Laravel Echo и объект `window.Echo` глобально доступен в вашем приложении. Более подробную информацию об установке Echo можно найти в [документации Laravel Echo](https://laravel.com/docs/broadcasting#client-side-installation).

### Прослушивание событий Echo

Представьте, что в вашем Laravel-приложении есть событие с именем `OrderShipped`:

```php
<?php

namespace App\Events;

use App\Models\Order;
use Illuminate\Broadcasting\Channel;
use Illuminate\Broadcasting\InteractsWithSockets;
use Illuminate\Contracts\Broadcasting\ShouldBroadcast;
use Illuminate\Foundation\Events\Dispatchable;
use Illuminate\Queue\SerializesModels;

class OrderShipped implements ShouldBroadcast
{
    use Dispatchable, InteractsWithSockets, SerializesModels;

    public Order $order;

    public function broadcastOn()
    {
        return new Channel('orders');
    }
}
```

Вы можете отправить это событие из другой части вашего приложения следующим образом:

```php
<?php

use App\Events\OrderShipped;

OrderShipped::dispatch();
```

Если бы вы прослушивали это событие в JavaScript, используя только Laravel Echo, это выглядело бы примерно так:

```js
Echo.channel('orders')
    .listen('OrderShipped', e => {
        console.log(e.order)
    })
```

Предполагая, что у вас установлен и настроен Laravel Echo, вы можете прослушивать это событие изнутри компонента Livewire.

Ниже приведён пример компонента `order-tracker`, который прослушивает событие `OrderShipped`, чтобы показать пользователям визуальное уведомление о новом заказе:

```php hl_lines="3" title="resources/views/components/⚡order-tracker.blade.php"
<?php

use Livewire\Attributes\On;
use Livewire\Component;

new class extends Component {
    public $showNewOrderNotification = false;

    #[On('echo:orders,OrderShipped')]
    public function notifyNewOrder()
    {
        $this->showNewOrderNotification = true;
    }

    // ...
};
```

Если у вас есть каналы Echo с переменными, встроенными в них (например, ID заказа), вы можете определять слушателей через метод `getListeners()` вместо атрибута `#[On]`:

```php hl_lines="3" title="resources/views/components/⚡order-tracker.blade.php"
<?php

use Livewire\Attributes\On;
use Livewire\Component;
use App\Models\Order;

new class extends Component {
    public Order $order;

    public $showOrderShippedNotification = false;

    public function getListeners()
    {
        return [
            "echo:orders.{$this->order->id},OrderShipped" => 'notifyShipped',
        ];
    }

    public function notifyShipped()
    {
        $this->showOrderShippedNotification = true;
    }

    // ...
};
```

Или, если вам удобнее, вы можете использовать синтаксис динамического имени события:

```php
<?php

#[On('echo:orders.{order.id},OrderShipped')]
public function notifyNewOrder()
{
    $this->showNewOrderNotification = true;
}
```

Если вам нужно получить доступ к данным события (payload), вы можете сделать это через переданный параметр `$event`:

```php
<?php

#[On('echo:orders.{order.id},OrderShipped')]
public function notifyNewOrder($event)
{
    $order = Order::find($event['orderId']);

    //
}
```

### Настройка имён событий трансляции с помощью `broadcastAs()`

По умолчанию Laravel транслирует события, используя имя класса события. Однако вы можете настроить имя транслируемого события, реализовав метод `broadcastAs()` в вашем классе события.

Например, если у вас есть событие `ScoreSubmitted`, но вы хотите транслировать его как `score.submitted`:

```php
<?php

namespace App\Events;

use Illuminate\Broadcasting\Channel;
use Illuminate\Broadcasting\InteractsWithSockets;
use Illuminate\Contracts\Broadcasting\ShouldBroadcast;
use Illuminate\Foundation\Events\Dispatchable;
use Illuminate\Queue\SerializesModels;

class ScoreSubmitted implements ShouldBroadcast
{
    use Dispatchable, InteractsWithSockets, SerializesModels;

    public function broadcastOn()
    {
        return new Channel('scores');
    }

    public function broadcastAs(): string
    {
        return 'score.submitted';
    }
}
```

При прослушивании этого события в компоненте Livewire следует использовать пользовательское имя трансляции, возвращаемое методом `broadcastAs()`, вместо имени класса. **Важно:** При использовании пользовательского имени трансляции его необходимо предварять точкой (`.`) — это позволяет отличить его от имён событий с пространством имён (классов). Это [соглашение Laravel Echo](https://laravel.com/docs/broadcasting#broadcast-name):

```php
<?php

namespace App\Livewire;

use Livewire\Attributes\On;
use Livewire\Component;

class ScoreBoard extends Component
{
    public $scores = [];

    #[On('echo:scores,.score.submitted')]
    public function handleScoreSubmitted($event)
    {
        $this->scores[] = $event['score'];
    }
}
```

В приведённом выше примере компонент Livewire прослушивает событие `.score.submitted` (пользовательское имя трансляции с префиксом в виде точки), а не `ScoreSubmitted` (имя класса). Точка в начале указывает Laravel Echo не добавлять пространство имён приложения (`App\Events`) к имени события.

Вы также можете использовать пользовательское имя трансляции вместе с динамическими именами каналов:

```php
<?php

#[On('echo:scores.{game.id},.score.submitted')]
public function handleScoreSubmitted($event)
{
    $this->scores[] = $event['score'];
}
```

### Приватные и присутствующие каналы

Вы также можете прослушивать события, транслируемые в приватные и присутствующие каналы:

!!! tip "Совет"
    Прежде чем продолжить, убедитесь, что вы определили <a href="https://laravel.com/docs/master/broadcasting#defining-authorization-callbacks">колбэки авторизации</a> для ваших каналов трансляции.

```php title="resources/views/components/⚡order-tracker.blade.php"
<?php

use Livewire\Component;

new class extends Component {
    public $showNewOrderNotification = false;

    public function getListeners()
    {
        return [
            // Публичный канал
            "echo:orders,OrderShipped" => 'notifyNewOrder',

            // Приватный канал
            "echo-private:orders,OrderShipped" => 'notifyNewOrder',

            // Присутствующий канал
            "echo-presence:orders,OrderShipped" => 'notifyNewOrder',
            "echo-presence:orders,here" => 'notifyNewOrder',
            "echo-presence:orders,joining" => 'notifyNewOrder',
            "echo-presence:orders,leaving" => 'notifyNewOrder',
        ];
    }

    public function notifyNewOrder()
    {
        $this->showNewOrderNotification = true;
    }
};
```

## Смотрите также

- **[Вложенность](/essentials/nesting)** — Общение между родительскими и дочерними компонентами
- **[Действия](/essentials/actions)** — Запуск событий из действий компонента
- **[Alpine](/features/alpine)** — Отправка и прослушивание событий с помощью Alpine
- **[Атрибут On](/php-attributes/attribute-on)** — Прослушивание событий с использованием атрибута #[On]
