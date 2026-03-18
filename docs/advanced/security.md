# Безопасность

Важно убедиться, что ваши приложения Livewire защищены и не подвержены уязвимостям. Livewire имеет встроенные функции безопасности для обработки многих случаев, однако иногда именно код вашего приложения отвечает за безопасность компонентов.

## Авторизация параметров действий

Действия Livewire чрезвычайно мощны, однако любые параметры, передаваемые действиям Livewire, могут быть изменены на стороне клиента и должны рассматриваться как недоверенный пользовательский ввод.

Пожалуй, самой распространённой ошибкой безопасности в Livewire является отсутствие валидации и авторизации вызовов действий Livewire перед сохранением изменений в базе данных.

Вот пример уязвимости, возникшей из-за отсутствия авторизации:

```php
<?php

use App\Models\Post;
use Livewire\Component;

class ShowPost extends Component
{
    // ...

    public function delete($id)
    {
        // НЕБЕЗОПАСНО!

        $post = Post::find($id);

        $post->delete();
    }
}
```

```html
<button wire:click="delete({{ $post->id }})">Удалить пост</button>
```


Причина, по которой приведённый выше пример небезопасен, заключается в том, что `wire:click="delete(...)"` может быть изменен в браузере для передачи ЛЮБОГО идентификатора поста, который пожелает злоумышленник.

Параметры действия (например, `$id` в данном случае) должны рассматриваться так же, как и любой недоверенный ввод из браузера.

Поэтому, чтобы обеспечить безопасность приложения и не дать пользователю удалить пост другого пользователя, мы должны добавить авторизацию к действию `delete()`.

Во-первых, давайте создадим [Laravel Policy](https://laravel.com/docs/authorization#creating-policies) для модели Post, выполнив следующую команду:

```bash
php artisan make:policy PostPolicy --model=Post
```

После выполнения вышеуказанной команды внутри `app/Policies/PostPolicy.php` будет создана новая политика. Затем мы можем обновить её содержимое методом `delete` следующим образом:

```php
<?php

namespace App\Policies;

use App\Models\Post;
use App\Models\User;

class PostPolicy
{
    /**
     * Определить, может ли данный пост быть удален пользователем.
     */
    public function delete(?User $user, Post $post): bool
    {
        return $user?->id === $post->user_id;
    }
}
```

Теперь мы можем использовать метод `$this->authorize()` из компонента Livewire, чтобы убедиться, что пользователь владеет постом перед его удалением:

```php hl_lines="9"
<?php

public function delete($id)
{
    $post = Post::find($id);

    // Если пользователь не владеет постом,
    // будет выброшено исключение AuthorizationException...
    $this->authorize('delete', $post);

    $post->delete();
}
```

Дополнительная литература:

* [Laravel Gates](https://laravel.com/docs/authorization#gates)
* [Laravel Policies](https://laravel.com/docs/authorization#creating-policies)

## Авторизация публичных свойств

Аналогично параметрам действий, публичные свойства в Livewire должны рассматриваться как недоверенный ввод от пользователя.

Вот тот же пример удаления поста, написанный небезопасно другим способом:

```php
<?php

use App\Models\Post;
use Livewire\Component;

class ShowPost extends Component
{
    public $postId;

    public function mount($postId)
    {
        $this->postId = $postId;
    }

    public function delete()
    {
        // НЕБЕЗОПАСНО!

        $post = Post::find($this->postId);

        $post->delete();
    }
}
```

```html
<button wire:click="delete">Удалить пост</button>
```

Как вы видите, вместо передачи `$postId` в качестве параметра методу `delete` из `wire:click`, мы сохраняем его как публичное свойство в компоненте Livewire.

Проблема такого подхода заключается в том, что любой злоумышленник может вставить на страницу собственный элемент, например:

```html
<input type="text" wire:model="postId">
```

Это позволит им свободно изменять `$postId` перед нажатием кнопки «Удалить пост». Поскольку действие `delete` не авторизует значение `$postId`, пользователь теперь может удалить любой пост в базе данных, независимо от того, владеет он им или нет.

Для защиты от этого риска существует два возможных решения:

### Использование свойств модели

При установке публичных свойств Livewire обрабатывает модели иначе, чем простые значения, такие как строки и целые числа. Благодаря этому, если мы вместо этого будем хранить всю модель поста как свойство компонента, Livewire гарантирует, что ID никогда не будет подделан.

Вот пример хранения свойства `$post` вместо простого свойства `$postId`:

```php
<?php

use App\Models\Post;
use Livewire\Component;

class ShowPost extends Component
{
    public Post $post;

    public function mount($postId)
    {
        $this->post = Post::find($postId);
    }

    public function delete()
    {
        $this->post->delete();
    }
}
```

```html
<button wire:click="delete">Удалить пост</button>
```

Этот компонент теперь защищен, так как у злоумышленника нет возможности изменить свойство `$post` на другую модель Eloquent.

### Блокировка свойства

Ещё один способ предотвратить установку нежелательных значений для свойств — использовать [атрибут `#[Locked]`](/php-attributes/attribute-locked). Блокировка свойств осуществляется путём применения атрибута `#[Locked]`. Теперь, если пользователи попытаются подделать это значение, будет выброшена ошибка.

Обратите внимание, что свойства с атрибутом Locked всё ещё могут быть изменены на стороне сервера, поэтому всё равно нужно следить за тем, чтобы недоверенный пользовательский ввод не передавался свойству в ваших собственных функциях Livewire.

```php hl_lines="9"
<?php

use App\Models\Post;
use Livewire\Component;
use Livewire\Attributes\Locked;

class ShowPost extends Component
{
    #[Locked]
    public $postId;

    public function mount($postId)
    {
        $this->postId = $postId;
    }

    public function delete()
    {
        $post = Post::find($this->postId);

        $post->delete();
    }
}
```

### Авторизация свойства

Если использование свойства модели нежелательно в вашем сценарии, вы, конечно, можете прибегнуть к ручной авторизации удаления поста внутри действия `delete`:

```php hl_lines="19"
<?php

use App\Models\Post;
use Livewire\Component;

class ShowPost extends Component
{
    public $postId;

    public function mount($postId)
    {
        $this->postId = $postId;
    }

    public function delete()
    {
        $post = Post::find($this->postId);

        $this->authorize('delete', $post);

        $post->delete();
    }
}
```

```html
<button wire:click="delete">Удалить пост</button>
```

Теперь, даже если злоумышленник всё ещё может свободно изменять значение `$postId`, при вызове действия `delete` метод `$this->authorize()` выбросит исключение `AuthorizationException`, если пользователь не владеет постом.

Дополнительная литература:

* [Laravel Gates](https://laravel.com/docs/authorization#gates)
* [Laravel Policies](https://laravel.com/docs/authorization#creating-policies)

## Мидлвары

Когда компонент Livewire загружается на страницу, содержащую [Мидлвар-авторизации](https://laravel.com/docs/authorization#via-middleware) на уровне маршрута, например:

```php hl_lines="2"
<?php

Route::livewire('/post/{post}', App\Livewire\UpdatePost::class)
    ->middleware('can:update,post');
```

Livewire гарантирует, что эти мидлвары будут повторно применены к последующим сетевым запросам Livewire. В ядре Livewire это называется «Persistent Middleware» (_постоянные мидлвары_).

Они защищают вас от сценариев, когда правила авторизации или права пользователя изменились после первоначальной загрузки страницы.

Вот более подробный пример такого сценария:

```php hl_lines="2"
<?php

Route::livewire('/post/{post}', App\Livewire\UpdatePost::class)
    ->middleware('can:update,post');
```

```php
<?php

use App\Models\Post;
use Livewire\Component;
use Livewire\Attributes\Validate;

class UpdatePost extends Component
{
    public Post $post;

    #[Validate('required|min:5')]
    public $title = '';

    public $content = '';

    public function mount()
    {
        $this->title = $this->post->title;
        $this->content = $this->post->content;
    }

    public function update()
    {
        $this->post->update([
            'title' => $this->title,
            'content' => $this->content,
        ]);
    }
}
```

Как вы видите, мидлвар `can:update,post` применяется на уровне маршрута. Это означает, что пользователь, у которого нет прав на обновление поста, не может просматривать страницу.

Однако рассмотрим сценарий, в котором пользователь:

* Загружает страницу
* Теряет разрешение на обновление после загрузки страницы
* Пытается обновить пост после потери разрешения

Поскольку Livewire уже успешно загрузил страницу, вы можете спросить себя: «Когда Livewire сделает последующий запрос на обновление поста, будет ли повторно применён мидлвар `can:update,post`? Или неавторизованный пользователь сможет успешно обновить пост?»

Поскольку у Livewire есть внутренние механизмы для повторного применения мидлваров из исходной конечной точки, вы защищены в этом сценарии.

### Настройка постоянных мидлваров

По умолчанию Livewire сохраняет следующие мидлвары между сетевыми запросами:

```php
\Laravel\Sanctum\Http\Middleware\EnsureFrontendRequestsAreStateful::class,
\Laravel\Jetstream\Http\Middleware\AuthenticateSession::class,
\Illuminate\Auth\Middleware\AuthenticateWithBasicAuth::class,
\Illuminate\Routing\Middleware\SubstituteBindings::class,
\App\Http\Middleware\RedirectIfAuthenticated::class,
\Illuminate\Auth\Middleware\Authenticate::class,
\Illuminate\Auth\Middleware\Authorize::class,
```

Если какой-либо из вышеперечисленных мидлваров применяется при первоначальной загрузке страницы, он будет сохранён (повторно применён) для любых будущих сетевых запросов.

Однако, если вы применяете собственный мидлвар из своего приложения при первоначальной загрузке страницы и хотите, чтобы он сохранялся между запросами Livewire, вам нужно будет добавить его в этот список в [сервис-провайдере](https://laravel.com/docs/providers#main-content) вашего приложения следующим образом:

```php hl_lines="15-17"
<?php

namespace App\Providers;

use Illuminate\Support\ServiceProvider;
use Livewire;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Загрузка любых сервисов приложения.
     */
    public function boot(): void
    {
        Livewire::addPersistentMiddleware([
            App\Http\Middleware\EnsureUserHasRole::class,
        ]);
    }
}
```

Если компонент Livewire загружен на странице, использующей мидлвар `EnsureUserHasRole` из вашего приложения, он теперь будет сохраняться и повторно применяться к любым будущим сетевым запросам к этому компоненту Livewire.

!!! warning "Аргументы мидлваров не поддерживаются"
    В настоящее время Livewire не поддерживает аргументы мидлваров для определений постоянных мидлваров.

    ```php
    // Плохо...
    Livewire::addPersistentMiddleware(AuthorizeResource::class.':admin');

    // Хорошо...
    Livewire::addPersistentMiddleware(AuthorizeResource::class);
    ```


### Применение глобальных мидлваров Livewire

В качестве альтернативы, если вы хотите применить определённый мидлвар к каждому сетевому запросу обновления Livewire, вы можете сделать это, зарегистрировав свой собственный маршрут обновления Livewire с любым мидлваром, который вы пожелаете:

```php
<?php

Livewire::setUpdateRoute(function ($handle, $path) {
    return Route::post($path, $handle)
        ->middleware(App\Http\Middleware\LocalizeViewPaths::class);
});
```

Любые AJAX/fetch запросы Livewire, сделанные к серверу, будут использовать вышеуказанную конечную точку и применять мидлвар `LocalizeViewPaths` перед обработкой обновления компонента.

Узнайте больше о [настройке маршрута обновления на странице установки](https://livewire.laravel.com/docs/installation#configuring-livewires-update-endpoint).

## Контрольные суммы снимков

Между каждым запросом Livewire делается снимок (snapshot) компонента Livewire и отправляется в браузер. Этот снимок используется для повторного создания компонента во время следующего обращения к серверу.

[Узнайте больше о снимках Livewire в документации по гидратации.](https://livewire.laravel.com/docs/hydration#the-snapshot)

Поскольку запросы fetch могут быть перехвачены и подделаны в браузере, Livewire генерирует «контрольную сумму» для каждого снимка, которая передается вместе с ним.

Эта контрольная сумма затем используется при следующем сетевом запросе для проверки того, что снимок никак не изменился.

Если Livewire обнаружит несоответствие контрольной суммы, он выбросит исключение `CorruptComponentPayloadException`, и запрос не будет выполнен.

Это защищает от любой формы злонамеренного вмешательства, которое в противном случае могло бы предоставить пользователям возможность выполнять или изменять несвязанный код.
