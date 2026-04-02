# Атрибут `Url`

Атрибут `#[Url]` сохраняет значение свойства в строке запроса URL, позволяя пользователям делиться ссылками и добавлять в закладки определённые состояния страницы.

## Базовое использование

Примените атрибут `#[Url]` к любому свойству, которое должно сохраняться в URL:

```php hl_lines="9" title="resources/views/components/user/⚡index.blade.php"
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
?>

<div>
    <input type="text" wire:model.live="search" placeholder="Поиск пользователей...">

    <ul>
        @foreach ($this->users as $user)
            <li wire:key="{{ $user->id }}">{{ $user->name }}</li>
        @endforeach
    </ul>
</div>
```

Когда пользователь вводит в поле поиска «bob», URL обновляется до `https://example.com/users?search=bob`. Если он поделится этой ссылкой или обновит страницу, поисковое значение сохранится.

## Как это работает

Атрибут `#[Url]` выполняет две вещи:

1. **Запись в URL** — При изменении свойства обновляет строку запроса
2. **Чтение из URL** — При загрузке страницы инициализирует свойство из строки запроса

Это создает общедоступное, сохраняемое в закладках состояние для вашего компонента.

## URL против Сессии

И `#[Url]`, и `#[Session]` сохраняют значения свойств, но с разными компромиссами:

| Функция | `#[Url]` | `#[Session]` |
|---------|----------|--------------|
| Сохраняется при обновлении | ✅ | ✅ |
| Сохраняется при обмене URL | ✅ | ❌ |
| Сохраняет URL чистым | ❌ | ✅ |
| Видимо пользователю | ✅ | ❌ |
| Состояние доступно для обмена | ✅ | ❌ |

Используйте `#[Url]`, когда хотите, чтобы пользователи могли делиться состоянием или добавлять его в закладки. Используйте `#[Session]`, когда состояние должно быть приватным.

## Использование псевдонима

Сократите или замаскируйте имена свойств в URL с помощью параметра `as`:

```php hl_lines="7" title="resources/views/components/user/⚡index.blade.php"
<?php

use Livewire\Attributes\Url;
use Livewire\Component;

new class extends Component {
    #[Url(as: 'q')]
    public $search = '';
};
```

URL будет отображаться как `?q=bob` вместо `?search=bob`.

## Исключение значений

По умолчанию Livewire добавляет параметры запроса только тогда, когда значения отличаются от их начального значения. Используйте `except`, чтобы настроить это:

```php hl_lines="7" title="resources/views/components/user/⚡index.blade.php"
<?php

use Livewire\Attributes\Url;
use Livewire\Component;

new class extends Component {
    #[Url(except: '')]
    public $search = '';

    public function mount()
    {
        $this->search = auth()->user()->username;
    }
};
```

Теперь Livewire будет исключать `search` из URL, только если это пустая строка, а не когда оно равно начальному имени пользователя.

## Всегда показывать в URL

Чтобы всегда включать параметр в URL, даже если он пуст, используйте `keep`:

```php hl_lines="1"
#[Url(keep: true)]
public $search = '';
```

URL всегда будет отображать `?search=`, даже если значение пустое.

## Свойства, допускающие `null`

Используйте подсказки типов, допускающих `null`, чтобы пустые параметры запроса трактовались как `null`, а не как пустые строки:

```php hl_lines="8" title="resources/views/components/user/⚡index.blade.php"
<?php

use Livewire\Attributes\Url;
use Livewire\Component;

new class extends Component {
    #[Url]
    public ?string $search;
};
```

Теперь `?search=` установит `$search` в `null`, а не в `''`.

## История браузера

По умолчанию Livewire использует `history.replaceState()` для изменения URL без добавления записей в историю браузера. Чтобы добавить записи в историю (чтобы кнопка «Назад» восстанавливала предыдущие значения запроса), используйте `history`:

```php hl_lines="1"
<?php

#[Url(history: true)]
public $search = '';
```

Теперь нажатие кнопки «Назад» в браузере восстановит предыдущие поисковые значения вместо перехода на предыдущую страницу.

## Когда использовать

Используйте `#[Url]`, когда:

* Создаете интерфейсы поиска или фильтрации
* Реализуете пагинацию
* Создаете общедоступные представления (положения на карте, выбранные фильтры и т. д.)
* Позволяете пользователям добавлять в закладки определённые состояния
* Поддерживаете навигацию вперёд/назад по состояниям браузера

## Пример: Фильтрация товаров

Вот практический пример фильтрации товаров с несколькими параметрами URL:

```php title="resources/views/pages/⚡products.blade.php"
<?php

use Livewire\Attributes\Computed;
use Livewire\Attributes\Url;
use Livewire\Component;
use App\Models\Product;

new class extends Component {
    #[Url(as: 'q')]
    public $search = '';

    #[Url]
    public $category = 'all';

    #[Url]
    public $minPrice = 0;

    #[Url]
    public $maxPrice = 1000;

    #[Url]
    public $sort = 'name';

    #[Computed]
    public function products()
    {
        return Product::query()
            ->when($this->search, fn($q) => $q->search($this->search))
            ->when($this->category !== 'all', fn($q) => $q->where('category', $this->category))
            ->whereBetween('price', [$this->minPrice, $this->maxPrice])
            ->orderBy($this->sort)
            ->paginate(20);
    }
};
?>

<div>
    <input type="text" wire:model.live="search" placeholder="Поиск товаров...">

    <select wire:model.live="category">
        <option value="all">Все категории</option>
        <option value="electronics">Электроника</option>
        <option value="clothing">Одежда</option>
    </select>

    <input type="range" wire:model.live="minPrice" min="0" max="1000">
    <input type="range" wire:model.live="maxPrice" min="0" max="1000">

    <select wire:model.live="sort">
        <option value="name">Название</option>
        <option value="price">Цена</option>
        <option value="created_at">Новизна</option>
    </select>

    @foreach($this->products as $product)
        <div wire:key="{{ $product->id }}">{{ $product->name }} - ${{ $product->price }}</div>
    @endforeach
</div>
```

Пользователи могут делиться такими URL:

```
https://example.com/products?q=laptop&category=electronics&minPrice=500&maxPrice=1500&sort=price
```

## Рекомендации по SEO

Параметры запроса индексируются поисковыми системами и включаются в аналитику:

* **Хорошо для SEO** — Каждая уникальная комбинация запроса создает уникальный URL, который может быть проиндексирован
* **Отслеживание аналитики** — Отслеживайте, какие фильтры и поиски используют пользователи
* **Сохранение в социальных сетях** — Параметры запроса сохраняются при обмене ссылками

## Узнать больше

Для получения дополнительной информации о параметрах запроса URL, включая метод `queryString()` и хуки трейтов, см. [документацию по параметрам запроса URL](/features/url).

## Справочник

```php
<?php

#[Url(
    ?string $as = null,
    bool $history = false,
    bool $keep = false,
    mixed $except = null,
    mixed $nullable = null,
)]
```

| Параметр | Тип | По умолчанию | Описание |
|-----------|------|---------|-------------|
| `$as` | `?string` | `null` | Пользовательское имя для параметра запроса в URL |
| `$history` | `bool` | `false` | Помещать изменения URL в историю браузера (включает кнопку «Назад») |
| `$keep` | `bool` | `false` | Сохранять параметр запроса при переходе с этой страницы |
| `$except` | `mixed` | `null` | Значение(я), которое нужно исключить из URL |
| `$nullable` | `mixed` | `null` | Значение, используемое, когда параметр запроса отсутствует в URL |
