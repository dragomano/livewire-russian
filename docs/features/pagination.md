# Пагинация

Функция пагинации Laravel позволяет запрашивать подмножество данных и предоставляет пользователям возможность перемещаться между *страницами* этих результатов.

Поскольку пагинатор Laravel был разработан для статических приложений, в приложении без Livewire каждый переход по страницам вызывает полный переход браузера на новый URL, содержащий нужную страницу (`?page=2`).

Однако при использовании пагинации внутри компонента Livewire пользователи могут перемещаться между страницами, оставаясь на той же странице. Livewire обработает всё за кулисами, включая обновление строки запроса URL с текущей страницей.

## Базовое использование

Ниже приведён самый простой пример использования пагинации внутри компонента `show-posts` для отображения только десяти постов за раз:

!!! warning "Вы должны использовать трейт `WithPagination`"
    Чтобы воспользоваться возможностями пагинации Livewire, каждый компонент, содержащий пагинацию, должен использовать трейт `Livewire\WithPagination`.

```php title="resources/views/components/⚡show-posts.blade.php"
<?php

use Livewire\Attributes\Computed;
use Livewire\WithPagination;
use Livewire\Component;
use App\Models\Post;

new class extends Component {
    use WithPagination;

    #[Computed]
    public function posts()
    {
        return Post::paginate(10);
    }
};
```

```html
<div>
    <div>
        @foreach ($this->posts as $post)
            <!-- ... -->
        @endforeach
    </div>

    {{ $this->posts->links() }}
</div>
```

Как видите, помимо ограничения количества отображаемых постов с помощью метода `Post::paginate()`, мы также будем использовать `$this->posts->links()` для отображения ссылок навигации по страницам.

Для получения дополнительной информации о пагинации в Laravel обратитесь к [исчерпывающей документации по пагинации](https://laravel.com/docs/pagination).

## Отключение отслеживания строки запроса URL

По умолчанию пагинатор Livewire отслеживает текущую страницу в строке запроса URL браузера следующим образом: `?page=2`.

Если вы хотите продолжать использовать утилиту пагинации Livewire, но отключить отслеживание строки запроса, вы можете сделать это с помощью трейта `WithoutUrlPagination`:

```php hl_lines="9"
<?php

use Livewire\WithoutUrlPagination;
use Livewire\WithPagination;
use Livewire\Component;

class ShowPosts extends Component
{
    use WithPagination, WithoutUrlPagination;

    // ...
}
```

Теперь пагинация будет работать как ожидается, но текущая страница не будет отображаться в строке запроса. Это также означает, что текущая страница не будет сохраняться при переходах между страницами.

## Настройка поведения прокрутки

По умолчанию пагинатор Livewire прокручивает страницу к верху после каждого изменения страницы.

Вы можете отключить это поведение, передав `false` параметру `scrollTo` метода `links()` следующим образом:

```php
{{ $posts->links(data: ['scrollTo' => false]) }}
```

В качестве альтернативы вы можете передать любой CSS-селектор параметру `scrollTo`, и Livewire найдёт ближайший элемент, соответствующий этому селектору, и прокрутит страницу к нему после каждой навигации:

```php
{{ $posts->links(data: ['scrollTo' => '#paginated-posts']) }}
```

## Сброс страницы

При сортировке или фильтрации результатов часто требуется сбросить номер страницы обратно на `1`.

По этой причине Livewire предоставляет метод `$this->resetPage()`, позволяющий сбросить номер страницы из любого места в вашем компоненте.

Следующий компонент демонстрирует использование этого метода для сброса страницы после отправки формы поиска:

```php title="resources/views/components/⚡show-posts.blade.php"
<?php

use Livewire\Attributes\Computed;
use Livewire\WithPagination;
use Livewire\Component;
use App\Models\Post;

new class extends Component {
    use WithPagination;

    public $query = '';

    public function search()
    {
        $this->resetPage();
    }

    #[Computed]
    public function posts()
    {
        return Post::where('title', 'like', '%'.$this->query.'%')->paginate(10);
    }
};
```

```html
<div>
    <form wire:submit="search">
        <input type="text" wire:model="query">

        <button type="submit">Искать посты</button>
    </form>

    <div>
        @foreach ($this->posts as $post)
            <!-- ... -->
        @endforeach
    </div>

    {{ $this->posts->links() }}
</div>
```

Теперь, если пользователь находился на странице `5` результатов, а затем дополнительно отфильтровал результаты, нажав «Поиск постов», страница будет сброшена обратно на `1`.

### Доступные методы навигации по страницам

Помимо `$this->resetPage()`, Livewire предоставляет другие полезные методы для программной навигации между страницами из вашего компонента:

| Метод        | Описание                               |
|-----------------|-------------------------------------------|
| `$this->setPage($page)`    | Установить пагинатор на определённый номер страницы |
| `$this->resetPage()`    | Сбросить страницу обратно на 1 |
| `$this->nextPage()`    | Перейти на следующую страницу |
| `$this->previousPage()`    | Перейти на предыдущую страницу |

## Несколько пагинаторов

Поскольку и Laravel, и Livewire используют параметры строки запроса URL для хранения и отслеживания текущего номера страницы, если одна страница содержит несколько пагинаторов, важно присвоить им разные имена.

Чтобы более наглядно продемонстрировать проблему, рассмотрим следующий компонент `show-clients`:

```php title="resources/views/components/⚡show-users.blade.php"
<?php

use Livewire\Attributes\Computed;
use Livewire\WithPagination;
use Livewire\Component;
use App\Models\Client;

new class extends Component {
    use WithPagination;

    #[Computed]
    public function clients()
    {
        return Client::paginate(10);
    }
};
```

Как видите, приведённый выше компонент содержит постраничный набор *клиентов*. Если пользователь перейдёт на страницу `2` этого набора результатов, URL может выглядеть следующим образом:

```
http://application.test/?page=2
```

Предположим, что страница также содержит компонент `show-invoices`, который тоже использует пагинацию. Чтобы независимо отслеживать текущую страницу каждого пагинатора, вам необходимо указать имя для второго пагинатора следующим образом:

```php title="resources/views/components/⚡show-invoices.blade.php"
<?php

use Livewire\Attributes\Computed;
use Livewire\WithPagination;
use Livewire\Component;
use App\Models\Invoice;

new class extends Component {
    use WithPagination;

    #[Computed]
    public function invoices()
    {
        return Invoice::paginate(10, pageName: 'invoices-page');
    }
};
```

Теперь, благодаря параметру `pageName`, который был добавлен к методу `paginate`, когда пользователь перейдёт на страницу `2` *счетов*, URL будет содержать следующее:

```
https://application.test/customers?page=2&invoices-page=2
```

При использовании методов навигации по страницам Livewire для именованного пагинатора вы должны указать имя страницы в качестве дополнительного параметра:

```php
<?php

$this->setPage(2, pageName: 'invoices-page');

$this->resetPage(pageName: 'invoices-page');

$this->nextPage(pageName: 'invoices-page');

$this->previousPage(pageName: 'invoices-page');
```

## Перехват обновлений страницы

Livewire позволяет выполнять код до и после обновления страницы, определив любой из следующих методов внутри вашего компонента:

```php title="resources/views/components/⚡show-posts.blade.php"
<?php

use Livewire\Attributes\Computed;
use Livewire\WithPagination;
use Livewire\Component;
use App\Models\Post;

new class extends Component {
    use WithPagination;

    public function updatingPage($page)
    {
        // Выполняется перед обновлением страницы для этого компонента...
    }

    public function updatedPage($page)
    {
        // Выполняется после обновления страницы для этого компонента...
    }

    #[Computed]
    public function posts()
    {
        return Post::paginate(10);
    }
};
```

### Хуки для именованных пагинаторов

Предыдущие хуки применяются только к пагинатору по умолчанию. Если вы используете именованный пагинатор, вы должны определять методы, используя имя пагинатора.

Например, ниже приведён пример того, как будет выглядеть хук для пагинатора с именем `invoices-page`:

```php
<?php

public function updatingInvoicesPage($page)
{
    //
}
```

### Общие хуки пагинатора

Если вы предпочитаете не указывать имя пагинатора в названии метода хука, вы можете использовать более универсальные альтернативы и просто принимать `$pageName` в качестве второго аргумента метода хука:

```php
<?php

public function updatingPaginators($page, $pageName)
{
    // Выполняется перед обновлением страницы для этого компонента...
}

public function updatedPaginators($page, $pageName)
{
    // Выполняется после обновления страницы для этого компонента...
}
```

## Использование простой темы

Вы можете использовать метод Laravel `simplePaginate()` вместо `paginate()` для повышения скорости и простоты.

При пагинации результатов с использованием этого метода пользователю будут отображаться только навигационные ссылки *следующая* и *предыдущая* вместо отдельных ссылок для каждого номера страницы:

```php
<?php

public function render()
{
    return view('show-posts', [
        'posts' => Post::simplePaginate(10),
    ]);
}
```

Для получения дополнительной информации о простой пагинации обратитесь к [документации Laravel по `simplePaginator`](https://laravel.com/docs/pagination#simple-pagination).

## Использование курсорной пагинации

Livewire также поддерживает использование курсорной пагинации Laravel — более быстрого метода пагинации, полезного при работе с большими наборами данных:

```php
<?php

public function render()
{
    return view('show-posts', [
        'posts' => Post::cursorPaginate(10),
    ]);
}
```

При использовании `cursorPaginate()` вместо `paginate()` или `simplePaginate()`, строка запроса в URL вашего приложения будет хранить закодированный *курсор* вместо стандартного номера страницы. Например:

```
https://example.com/posts?cursor=eyJpZCI6MTUsIl9wb2ludHNUb05leHRJdGVtcyI6dHJ1ZX0
```

Для получения дополнительной информации о курсорной пагинации обратитесь к [документации Laravel по курсорной пагинации](https://laravel.com/docs/pagination#cursor-pagination).

## Использование Bootstrap вместо Tailwind

Если в качестве CSS-фреймворка вашего приложения вы используете [Bootstrap](https://getbootstrap.com/) вместо [Tailwind](https://tailwindcss.com/), вы можете настроить Livewire на использование представлений пагинации в стиле Bootstrap вместо представлений Tailwind по умолчанию.

Для этого задайте значение параметра `pagination_theme` в конфигурационном файле вашего приложения `config/livewire.php`:

```php
'pagination_theme' => 'bootstrap',
```

!!! info "Публикация конфигурационного файла Livewire"
    Перед настройкой темы пагинации необходимо сначала опубликовать конфигурационный файл Livewire в каталог `/config` вашего приложения, выполнив следующую команду:
    ```shell
    php artisan livewire:config
    ```

## Изменение представлений пагинации по умолчанию

Если вы хотите изменить представления пагинации Livewire, чтобы они соответствовали стилю вашего приложения, вы можете сделать это, *опубликовав* их с помощью следующей команды:

```shell
php artisan livewire:publish --pagination
```

После выполнения этой команды следующие четыре файла будут добавлены в каталог `resources/views/vendor/livewire`:

| Имя файла представления | Описание                               |
|-----------------|-------------------------------------------|
| `tailwind.blade.php`    | Стандартная тема пагинации Tailwind |
| `tailwind-simple.blade.php`    | *Простая* тема пагинации Tailwind |
| `bootstrap.blade.php`    | Стандартная тема пагинации Bootstrap |
| `bootstrap-simple.blade.php`    | *Простая* тема пагинации Bootstrap |

После публикации файлов вы получаете над ними полный контроль. При выводе ссылок пагинации с помощью метода `->links()` объекта результатов внутри вашего шаблона, Livewire будет использовать эти файлы вместо своих собственных.

## Использование пользовательских представлений пагинации

Если вы хотите полностью отказаться от встроенных представлений пагинации Livewire, вы можете вывести свои собственные одним из двух способов:

1. Метод `->links()` в вашем представлении Blade
2. Метод `paginationView()` или `paginationSimpleView()` в вашем компоненте

### Через `->links()`

Первый подход заключается в том, чтобы просто передать имя вашего пользовательского представления пагинации Blade напрямую в метод `->links()`:

```php
{{ $posts->links('custom-pagination-links') }}
```

При выводе ссылок пагинации Livewire теперь будет искать представление по пути `resources/views/custom-pagination-links.blade.php`.

### Через `paginationView()` или `paginationSimpleView()`

Второй подход заключается в объявлении метода `paginationView` или `paginationSimpleView` внутри вашего компонента, который возвращает имя представления, которое вы хотели бы использовать:

```php
<?php

public function paginationView()
{
    return 'custom-pagination-links-view';
}

public function paginationSimpleView()
{
    return 'custom-simple-pagination-links-view';
}
```

### Пример представления пагинации

Ниже для справки приведен простой пример представления пагинации Livewire без стилей.

Как видите, вы можете использовать вспомогательные методы навигации по страницам Livewire, такие как `$this->nextPage()`, непосредственно в вашем шаблоне, добавив `wire:click="nextPage"` к кнопкам:

```html
<div>
    @if ($paginator->hasPages())
        <nav role="navigation" aria-label="Pagination Navigation">
            <span>
                @if ($paginator->onFirstPage())
                    <span>Предыдущая</span>
                @else
                    <button wire:click="previousPage" wire:loading.attr="disabled" rel="prev">Предыдущая</button>
                @endif
            </span>

            <span>
                @if ($paginator->onLastPage())
                    <span>Следующая</span>
                @else
                    <button wire:click="nextPage" wire:loading.attr="disabled" rel="next">Следующая</button>
                @endif
            </span>
        </nav>
    @endif
</div>
```

Для чисто визуальных состояний загрузки (например, изменения прозрачности) вместо этого можно использовать автоматический атрибут Livewire `data-loading` вместе с классами Tailwind:

```html
<button wire:click="nextPage" class="data-loading:opacity-50" rel="next">
    Следующая
</button>
```

[Узнайте больше о состояниях загрузки →](/features/loading-states)

## Смотрите также

- **[Параметры URL-запроса](/features/url)** — Синхронизация состояния пагинации с URL
- **[Состояния загрузки](/features/loading-states)** — Отображение обратной связи при смене страниц
- **[Вычисляемые свойства](/features/computed-properties)** — Эффективный запрос данных с пагинацией
