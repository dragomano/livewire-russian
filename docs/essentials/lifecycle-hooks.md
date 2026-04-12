# Хуки жизненного цикла

Livewire предоставляет множество хуков жизненного цикла, которые позволяют выполнять код в определённые моменты существования компонента. Эти хуки дают возможность выполнять действия до или после конкретных событий, таких как инициализация компонента, обновление свойств или отрисовка шаблона.

Вот полный список доступных хуков жизненного цикла компонента:

| Метод хука                        | Описание                                                                             |
|-----------------------------------|--------------------------------------------------------------------------------------|
| `mount()`                         | Вызывается при создании компонента                                                   |
| `hydrate()`                       | Вызывается при повторной гидратации компонента в начале каждого последующего запроса |
| `boot()`                          | Вызывается в начале каждого запроса (как первого, так и последующих)                 |
| `updating()`                      | Вызывается перед обновлением свойства компонента                                     |
| `updated()`                       | Вызывается после обновления свойства                                                 |
| `rendering()`                     | Вызывается перед рендерингом представления компонента                                |
| `rendered()`                      | Вызывается после рендеринга представления компонента                                 |
| `dehydrate()`                     | Вызывается в конце обработки каждого запроса компонента                              |
| `exception($e, $stopPropagation)` | Вызывается при возникновении исключения                                              |

## Mount

В обычном PHP-классе конструктор (`__construct()`) принимает внешние параметры и инициализирует состояние объекта. Однако в Livewire для принятия параметров и начальной инициализации состояния компонента используется метод `mount()`.

Livewire-компоненты не используют `__construct()`, потому что компоненты **пересоздаются** при каждом последующем сетевом запросе, а нам нужно инициализировать компонент только один раз — при его первом создании.

Вот пример использования метода `mount()` для инициализации свойств `name` и `email` в компоненте `profile.edit`:

```php title="resources/views/components/profile/⚡edit.blade.php"
<?php

use Illuminate\Support\Facades\Auth;
use Livewire\Component;

new class extends Component {
    public $name;

    public $email;

    public function mount()
    {
        $this->name = Auth::user()->name;

        $this->email = Auth::user()->email;
    }

    // ...
};
```

Как упоминалось ранее, метод `mount()` получает данные, переданные в компонент, в виде параметров метода:

```php title="resources/views/components/post/⚡edit.blade.php"
<?php

use Livewire\Component;
use App\Models\Post;

new class extends Component {
    public $title;

    public $content;

    public function mount(Post $post)
    {
        $this->title = $post->title;

        $this->content = $post->content;
    }

    // ...
};
```

!!! tip "Вы можете использовать внедрение зависимостей со всеми методами-хуками"
    Livewire позволяет разрешать зависимости из [сервис-контейнера Laravel](https://laravel.com/docs/container#automatic-injection), просто указывая типы параметров методов в хуках жизненного цикла.

Метод `mount()` — это важнейшая часть работы с Livewire. Следующая документация содержит дополнительные примеры использования метода `mount()` для решения типичных задач:

* [Инициализация свойств](/essentials/properties#инициализация-свойств)
* [Получение данных от родительских компонентов](/essentials/nesting#передача-пропсов-дочерним-компонентам)
* [Доступ к параметрам маршрута](/essentials/pages#доступ-к-параметрам-маршрута)

## Boot

Как бы ни был полезен `mount()`, он выполняется только один раз за весь жизненный цикл компонента. Иногда же требуется выполнять какую-то логику в начале **каждого** запроса к серверу для данного компонента.

Для таких случаев в Livewire предусмотрен метод `boot()`, в котором можно писать код инициализации компонента, который должен выполняться **каждый раз**, когда класс компонента загружается: и при первой инициализации, и при всех последующих запросах.

Метод `boot()` удобно использовать, например, для инициализации защищённых свойств, которые не сохраняются между запросами. Ниже приведён пример инициализации защищённого свойства в виде модели Eloquent:

```php hl_lines="13-16" title="resources/views/components/post/⚡show.blade.php"
<?php

use Livewire\Attributes\Locked;
use Livewire\Component;
use App\Models\Post;

new class extends Component {
    #[Locked]
    public $postId = 1;

    protected Post $post;

    public function boot()
    {
        $this->post = Post::find($this->postId);
    }

    // ...
};
```

Вы можете использовать этот подход, чтобы полностью контролировать инициализацию свойства компонента в Livewire.

!!! tip "В большинстве случаев лучше использовать вычисляемое свойство"
    Приведённая выше техника очень мощная, однако чаще всего для решения подобных задач лучше использовать [вычисляемые свойства Livewire](/features/computed-properties).

!!! warning "Всегда блокируйте чувствительные публичные свойства"
    Как видно из примера выше, мы используем атрибут `#[Locked]` для свойства `$postId`. В подобных сценариях, когда важно гарантировать, что пользователи на стороне клиента не смогут изменить значение `$postId`, необходимо либо авторизовать значение свойства перед его использованием, либо добавить атрибут `#[Locked]`, чтобы свойство никогда не могло быть изменено.

    Подробнее об этом читайте в [документации по атрибуту Locked](/php-attributes/attribute-locked).

## Update

Пользователи на стороне клиента могут обновлять публичные свойства разными способами, чаще всего — через изменение значения input с директивой `wire:model`.

Livewire предоставляет удобные хуки, которые позволяют перехватывать процесс обновления публичного свойства, чтобы вы могли валидировать или авторизовать значение до того, как оно будет установлено, либо гарантировать, что свойство будет установлено в нужном формате.

Ниже приведён пример использования хука `updating`, чтобы запретить изменение свойства `$postId`.

Стоит отметить, что в реальном приложении для такого случая лучше использовать атрибут [`#[Locked]`](/php-attributes/attribute-locked), как показано в примере выше.

```php title="resources/views/components/post/⚡show.blade.php"
<?php

use Exception;
use Livewire\Component;

new class extends Component {
    public $postId = 1;

    public function updating($property, $value)
    {
        // $property: Имя текущего свойства, которое обновляется
        // $value: Значение, которое вот-вот будет установлено в свойство

        if ($property === 'postId') {
            throw new Exception;
        }
    }

    // ...
};
```

Метод `updating()` выполняется **до** того, как свойство будет обновлено, что позволяет вам перехватывать некорректный ввод и предотвращать обновление свойства. Ниже приведён пример использования метода `updated()`, чтобы гарантировать, что значение свойства остаётся согласованным:

```php title="resources/views/components/user/⚡create.blade.php"
<?php

use Livewire\Component;

new class extends Component {
    public $username = '';

    public $email = '';

    public function updated($property)
    {
        // $property: Имя текущего свойства, которое было обновлено

        if ($property === 'username') {
            $this->username = strtolower($this->username);
        }
    }

    // ...
};
```

Теперь, каждый раз, когда свойство `$username` обновляется на стороне клиента, мы гарантируем, что значение всегда будет в нижнем регистре.

Поскольку при использовании хуков обновления вы часто работаете с конкретным свойством, Livewire позволяет указывать имя свойства прямо в названии метода. Вот тот же пример, что был выше, но переписанный с использованием этой техники:

```php title="resources/views/components/user/⚡create.blade.php"
<?php

use Livewire\Component;

new class extends Component {
    public $username = '';

    public $email = '';

    public function updatedUsername()
    {
        $this->username = strtolower($this->username);
    }

    // ...
};
```

Конечно, вы также можете применить эту технику к хуку `updating`.

### Массивы

Свойства-массивы получают дополнительный аргумент `$key` в эти методы, чтобы указать, какой именно элемент изменяется.

Обратите внимание: когда обновляется сам массив целиком (а не конкретный ключ), аргумент `$key` будет равен `null`.

```php title="resources/views/components/preferences/⚡edit.blade.php"
<?php

use Livewire\Component;

new class extends Component {
    public $preferences = [];

    public function updatedPreferences($value, $key)
    {
        // $value = 'dark'
        // $key   = 'theme'
    }

    // ...
};
```

## Hydrate и Dehydrate

Hydrate и Dehydrate — это менее известные и реже используемые хуки. Однако в определённых сценариях они могут быть очень мощными инструментами.

Термины «dehydrate» (дегидратация) и «hydrate» (гидратация) означают процесс сериализации компонента Livewire в JSON для передачи на клиентскую сторону, а затем обратной десериализации обратно в PHP-объект при следующем запросе.

Мы часто используем термины «hydrate» и «dehydrate» для описания этого процесса в кодовой базе Livewire и в документации. Если вам нужна большая ясность по этим понятиям, вы можете подробнее ознакомиться в [документации по гидратации](/advanced/hydration).

Давайте рассмотрим пример, в котором одновременно используются `mount()`, `hydrate()` и `dehydrate()` для поддержки использования собственного [объекта передачи данных (DTO)](https://ru.wikipedia.org/wiki/DTO) вместо модели Eloquent для хранения данных поста в компоненте:

```php title="resources/views/components/post/⚡show.blade.php"
<?php

use Livewire\Component;

new class extends Component {
    public $post;

    public function mount($title, $content)
    {
        // Выполняется в начале самого первого начального запроса...

        $this->post = new PostDto([
            'title' => $title,
            'content' => $content,
        ]);
    }

    public function hydrate()
    {
        // Выполняется в начале каждого «последующего» запроса...
        // НЕ выполняется при первом запросе (за это отвечает mount)...

        $this->post = new PostDto($this->post);
    }

    public function dehydrate()
    {
        // Выполняется в конце каждого запроса...

        $this->post = $this->post->toArray();
    }

    // ...
};
```

Теперь из действий и других мест внутри вашего компонента вы можете работать с объектом `PostDto` вместо примитивных данных.

Приведённый выше пример в первую очередь демонстрирует возможности и суть хуков `hydrate()` и `dehydrate()`. Однако рекомендуется выполнять такие задачи с помощью [Wireables или Синтезаторов](/essentials/properties#поддержка-пользовательских-типов).

## Render

Если вы хотите подключиться к процессу отрисовки Blade-шаблона компонента, это можно сделать с помощью хуков `rendering()` и `rendered()`:

```php title="resources/views/components/post/⚡index.blade.php"
<?php

use Livewire\Component;
use App\Models\Post;

new class extends Component {
    public function render()
    {
        return $this->view([
            'post' => Post::all(),
        ]);
    }

    public function rendering($view, $data)
    {
        // Выполняется ДО того, как будет отрисован переданный шаблон...
        //
        // $view: Шаблон (вью), который вот-вот будет отрисован
        // $data: Данные, передаваемые в шаблон
    }

    public function rendered($view, $html)
    {
        // Выполняется ПОСЛЕ того, как будет отрисован переданный шаблон...
        //
        // $view: Отрисованный шаблон (вью)
        // $html: Финальный отрисованный HTML
    }

    // ...
};
```

## Exception

Иногда полезно перехватывать и обрабатывать ошибки — например, чтобы настроить собственное сообщение об ошибке или игнорировать определённые типы исключений. Для этого в Livewire существует хук `exception()`, который позволяет именно это: вы можете проверить объект `$e` (исключение) и с помощью параметра `$stopPropagation` «поймать» ошибку и остановить её дальнейшее распространение.

Этот же механизм открывает мощные паттерны, когда нужно прервать дальнейшее выполнение кода (ранний выход). Именно так внутри Livewire работают, например, методы валидации вроде `validate()`.

```php hl_lines="6-9" title="resources/views/components/post/⚡show.blade.php"
<?php

use Livewire\Component;

new class extends Component {
    public function mount()
    {
        $this->post = Post::find($this->postId);
    }

    public function exception($e, $stopPropagation) {
        if ($e instanceof NotFoundException) {
            $this->notify('Пост не найден');
            $stopPropagation();
        }
    }

    // ...
};
```

## Использование хуков внутри трейта

Трейты — это удобный способ повторного использования кода между компонентами или выноса кода из одного компонента в отдельный файл.

Чтобы избежать конфликтов между несколькими трейтами при объявлении методов хуков жизненного цикла, Livewire поддерживает префикс методов хуков в виде _camelCase_-имени текущего трейта, который их объявляет.

Благодаря этому вы можете использовать одни и те же хуки жизненного цикла в разных трейтах, не опасаясь конфликта имён методов.

Ниже приведён пример компонента, который использует трейт с именем `HasPostForm`:

```php title="resources/views/components/post/⚡create.blade.php"
<?php

use Livewire\Component;

new class extends Component {
    use HasPostForm;

    // ...
};
```

Теперь вот сам трейт `HasPostForm`, содержащий все доступные префиксные хуки:

```php
trait HasPostForm
{
    public $title = '';

    public $content = '';

    public function mountHasPostForm()
    {
        // ...
    }

    public function hydrateHasPostForm()
    {
        // ...
    }

    public function bootHasPostForm()
    {
        // ...
    }

    public function updatingHasPostForm()
    {
        // ...
    }

    public function updatedHasPostForm()
    {
        // ...
    }

    public function renderingHasPostForm()
    {
        // ...
    }

    public function renderedHasPostForm()
    {
        // ...
    }

    public function dehydrateHasPostForm()
    {
        // ...
    }

    // ...
}
```

## Использование хуков внутри объекта формы

Объекты форм в Livewire поддерживают хуки обновления свойств. Эти хуки работают аналогично [хукам обновления компонентов](#update), позволяя выполнять действия при изменении свойств внутри объекта формы.

Ниже приведён пример компонента, который использует объект формы `PostForm`:

```php title="resources/views/components/post/⚡create.blade.php"
<?php

use Livewire\Component;

new class extends Component {
    public PostForm $form;

    // ...
};
```

Вот объект формы `PostForm`, содержащий все доступные хуки:

```php
namespace App\Livewire\Forms;

use Livewire\Attributes\Validate;
use Livewire\Form;

class PostForm extends Form
{
    public $title = '';

    public $tags = [];

    public function updating($property, $value)
    {
        // ...
    }

    public function updated($property, $value)
    {
        // ...
    }

    public function updatingTitle($value)
    {
        // ...
    }

    public function updatedTitle($value)
    {
        // ...
    }

    public function updatingTags($value, $key)
    {
        // ...
    }

    public function updatedTags($value, $key)
    {
        // ...
    }

    // ...
}
```

## Смотрите также

- **[Свойства](/essentials/properties)** — Инициализация свойств в методах `mount()` и `boot()`
- **[Компоненты](/essentials/components)** — Понимание, когда выполняются хуки при создании компонента
- **[Страницы](/essentials/pages)** — Использование `mount()` для получения параметров маршрута
- **[Гидратация](/advanced/hydration)** — Понимание хуков `hydrate()` и `dehydrate()`
