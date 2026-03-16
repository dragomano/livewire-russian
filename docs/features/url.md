# URL

Livewire позволяет сохранять свойства компонента в строке запроса URL. Например, вы можете захотеть, чтобы свойство `$search` вашего компонента отображалось в URL: `https://example.com/users?search=bob`. Это особенно удобно для фильтрации, сортировки и пагинации, поскольку пользователи смогут делиться ссылкой и добавлять страницу в закладки с конкретным состоянием.

## Базовое использование

Ниже приведён компонент `show-users`, который позволяет искать пользователей по имени через простое текстовое поле ввода:

```php title="resources/views/components/⚡show-users.blade.php"
<?php

use Livewire\Attributes\Computed;
use Livewire\Component;
use App\Models\User;

new class extends Component {
    public $search = '';

    #[Computed]
    public function users()
    {
        return User::search($this->search)->get();
    }
};
```

```html
<div>
    <input type="text" wire:model.live="search">

    <ul>
        @foreach ($this->users as $user)
            <li wire:key="{{ $user->id }}">{{ $user->name }}</li>
        @endforeach
    </ul>
</div>
```

Как вы можете видеть, поскольку текстовое поле использует `wire:model.live="search"`, при каждом вводе символа пользователем будут отправляться сетевые запросы для обновления свойства `$search` и отображения отфильтрованного списка пользователей на странице.

Однако если посетитель обновит страницу, значение поиска и результаты будут потеряны.

Чтобы сохранить значение поиска при обновлении страницы (а также дать возможность посетителю поделиться ссылкой), мы можем хранить значение поиска в строке запроса URL, добавив атрибут `#[Url]` над свойством `$search` следующим образом:

```php hl_lines="9" title="resources/views/components/⚡show-users.blade.php"
<?php

use Livewire\Attributes\Computed;
use Livewire\Attributes\Url;
use Livewire\Component;
use App\Models\User;

new class extends Component {
    #[Url]
    public $search = '';

    #[Computed]
    public function users()
    {
        return User::search($this->search)->get();
    }
};
```

Теперь, если пользователь введёт «bob» в поле поиска, адресная строка браузера будет показывать:

```
https://example.com/users?search=bob
```

Если теперь открыть этот URL в новом окне браузера, в поле поиска будет автоматически подставлено «bob», а результаты пользователей будут отфильтрованы соответствующим образом.

## Инициализация свойств из URL

Как вы видели в предыдущем примере, когда свойство помечено атрибутом `#[Url]`, оно не только сохраняет своё обновлённое значение в строку запроса URL, но и считывает любые существующие значения из строки запроса при загрузке страницы.

Например, если пользователь заходит по адресу `https://example.com/users?search=bob`, Livewire автоматически установит начальное значение свойства `$search` равным «bob».

```php
<?php

use Livewire\Attributes\Url;
use Livewire\Component;

class ShowUsers extends Component
{
    #[Url]
    public $search = ''; // Будет установлено в "bob"...

    // ...
}
```

### Свойства, допускающие null

По умолчанию, если страница загружается с пустым значением в строке запроса, например `?search=`, Livewire будет трактовать это значение как пустую строку. В большинстве случаев это ожидаемое поведение, однако иногда требуется, чтобы `?search=` интерпретировалось именно как `null`.

В таких случаях можно использовать nullable-тип следующим образом:

```php hl_lines="9"
<?php

use Livewire\Attributes\Url;
use Livewire\Component;

class ShowUsers extends Component
{
    #[Url]
    public ?string $search;

    // ...
}
```

Поскольку в указанном выше типе присутствует `?`, Livewire увидит `?search=` и установит значение `$search` в `null` вместо пустой строки.

Это работает и в обратную сторону: если в вашем приложении вы установите `$this->search = null`, то в строке запроса URL это будет представлено как `?search=`.

## Использование псевдонима

Livewire даёт вам полный контроль над тем, какое имя будет отображаться в строке запроса URL. Например, у вас может быть свойство `$search`, но вы хотите либо скрыть настоящее имя свойства, либо сократить его до `q`.

Вы можете указать псевдоним для строки запроса, передав параметр `as` в атрибут `#[Url]`:

```php
<?php

use Livewire\Attributes\Url;
use Livewire\Component;

class ShowUsers extends Component
{
    #[Url(as: 'q')]
    public $search = '';

    // ...
}
```

Теперь, когда пользователь введёт «bob» в поле поиска, в URL будет отображаться: `https://example.com/users?q=bob` вместо `?search=bob`.

## Исключение определённых значений

По умолчанию Livewire добавляет запись в строку запроса только тогда, когда значение свойства изменилось по сравнению с тем, что было при инициализации. В большинстве случаев это желаемое поведение, однако бывают ситуации, когда требуется более тонкий контроль над тем, какие значения Livewire исключает из строки запроса. В таких случаях можно использовать параметр `except`.

Например, в компоненте ниже начальное значение `$search` изменяется в методе `mount()`. Чтобы гарантировать, что браузер будет исключать параметр `search` из строки запроса только тогда, когда значение `search` — пустая строка, к атрибуту `#[Url]` добавлен параметр `except`:

```php
<?php

use Livewire\Attributes\Url;
use Livewire\Component;

class ShowUsers extends Component
{
    #[Url(except: '')]
    public $search = '';

    public function mount() {
        $this->search = auth()->user()->username;
    }

    // ...
}
```

Без параметра `except` в приведённом выше примере Livewire удалял бы запись `search` из строки запроса каждый раз, когда значение `search` становилось равным начальному значению `auth()->user()->username`. Вместо этого, благодаря использованию `except: ''`, Livewire сохраняет все значения в строке запроса, кроме случая, когда `search` — пустая строка.

## Отображение при загрузке страницы

По умолчанию Livewire добавляет значение в строку запроса только после того, как это значение было изменено на странице. Например, если начальное значение свойства `$search` — пустая строка `""`, то при пустом поле поиска в URL ничего не будет отображаться.

Если вы хотите, чтобы параметр `?search` всегда присутствовал в строке запроса (даже когда значение пустое), можно передать параметр `keep` в атрибут `#[Url]`:

```php
<?php

use Livewire\Attributes\Url;
use Livewire\Component;

class ShowUsers extends Component
{
    #[Url(keep: true)]
    public $search = '';

    // ...
}
```

Теперь при загрузке страницы URL будет изменён на следующий: `https://example.com/users?search=`

## Сохранение в истории браузера

По умолчанию Livewire использует метод [`history.replaceState()`](https://developer.mozilla.org/en-US/docs/Web/API/History/replaceState) для изменения URL вместо [`history.pushState()`](https://developer.mozilla.org/en-US/docs/Web/API/History/pushState). Это означает, что при обновлении строки запроса Livewire **заменяет** текущую запись в истории браузера, а не добавляет новую.

Из-за того, что Livewire «заменяет» текущую запись истории, нажатие кнопки «Назад» в браузере вернёт пользователя на предыдущую страницу, а не к предыдущему значению `?search=`.

Чтобы заставить Livewire использовать `history.pushState` при обновлении URL (т. е. добавлять новые записи в историю), можно передать параметр `history` в атрибут `#[Url]`:

```php
<?php

use Livewire\Attributes\Url;
use Livewire\Component;

class ShowUsers extends Component
{
    #[Url(history: true)]
    public $search = '';

    // ...
}
```

В примере выше, когда пользователь меняет значение поиска с «bob» на «frank», а затем нажимает кнопку «Назад» в браузере, значение поиска (и URL) вернётся к «bob», вместо того чтобы перейти на предыдущую посещенную страницу.

## Использование метода queryString

Строку запроса также можно определять как метод компонента. Это удобно, если некоторые свойства должны иметь динамические параметры (например, зависящие от условий, вычислений или других данных).

```php
<?php

use Livewire\Component;

class ShowUsers extends Component
{
    // ...

    protected function queryString()
    {
        return [
            'search' => [
                'as' => 'q',
            ],
        ];
    }
}
```

## Хуки трейта

Livewire также предоставляет [хуки](/essentials/lifecycle-hooks) специально для работы со строкой запроса.

```php
<?php

trait WithSorting
{
    // ...

    protected function queryStringWithSorting()
    {
        return [
            'sortBy' => ['as' => 'sort'],
            'sortDirection' => ['as' => 'direction'],
        ];
    }
}
```

## Смотрите также

- **[Свойства](/essentials/properties)** — Синхронизация свойств с параметрами URL
- **[Навигация](/features/navigate)** — Сохранение состояния URL при навигации в SPA
- **[Атрибут Url](/php-attributes/attribute-url)** — Привязка свойств к строке запроса URL
- **[Страницы](/essentials/pages)** — Работа с параметрами маршрута
