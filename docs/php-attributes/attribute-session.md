# Атрибут `Session`

Атрибут `#[Session]` сохраняет значение свойства в сессии пользователя, поддерживая его между обновлениями страницы и навигацией.

## Базовое использование

Примените атрибут `#[Session]` к любому свойству, которое должно сохраняться в сессии:

```php hl_lines="9" title="resources/views/components/post/⚡index.blade.php"
<?php

use Livewire\Attributes\Session;
use Livewire\Attributes\Computed;
use Livewire\Component;
use App\Models\Post;

new class extends Component {
    #[Session]
    public $search = '';

    #[Computed]
    public function posts()
    {
        return $this->search === ''
            ? Post::all()
            : Post::where('title', 'like', "%{$this->search}%")->get();
    }
};
?>

<div>
    <input type="text" wire:model.live="search" placeholder="Поиск постов...">

    @foreach($this->posts as $post)
        <div wire:key="{{ $post->id }}">{{ $post->title }}</div>
    @endforeach
</div>
```

После того как пользователь введет поисковый запрос, он может обновить страницу или уйти с нее и вернуться — поисковое значение будет сохранено.

## Как это работает

При каждом изменении свойства Livewire сохраняет его новое значение в сессии пользователя. Когда компонент загружается, Livewire извлекает значение из сессии и инициализирует им свойство.

Это позволяет сохранять состояние интерфейса для пользователя без изменения URL.

## Сессия против URL

И `#[Session]`, и `#[Url]` сохраняют значения свойств, но с разными компромиссами:

| Функция | `#[Session]` | `#[Url]` |
|---------|-------------|----------|
| Сохраняется при обновлении | ✅ | ✅ |
| Сохраняется при обмене URL | ❌ | ✅ |
| Сохраняет URL чистым | ✅ | ❌ |
| Видимо пользователю | ❌ | ✅ |
| Состояние доступно для обмена | ❌ | ✅ |

Используйте `#[Session]`, когда вам нужна устойчивость без загромождения URL или когда состояние не должно быть доступно для обмена.

## Пользовательские ключи сессии

По умолчанию Livewire генерирует ключи сессии, используя имена компонента и свойства. Вы можете настроить это:

```php hl_lines="7" title="resources/views/components/post/⚡index.blade.php"
<?php

use Livewire\Attributes\Session;
use Livewire\Component;

new class extends Component {
    #[Session(key: 'post_search')]
    public $search = '';
};
```

Свойство будет храниться в сессии с использованием ключа `post_search`.

## Динамические ключи сессии

Вы можете генерировать ключи динамически, используя другие свойства:

```php hl_lines="10" title="resources/views/components/post/⚡index.blade.php"
<?php

use Livewire\Attributes\Session;
use Livewire\Component;
use App\Models\Author;

new class extends Component {
    public Author $author;

    #[Session(key: 'search-{author.id}')]
    public $search = '';
};
```

Если `$author->id` равен `4`, ключ сессии станет `search-4`. Это позволяет использовать разные значения сессии для каждого автора.

## Когда использовать

Используйте `#[Session]`, когда:

* Сохраняете предпочтения пользователя (тема, язык, состояние боковой панели)
* Поддерживаете состояние фильтра/поиска при навигации по страницам
* Храните данные формы, чтобы предотвратить их потерю при обновлении
* Сохраняете состояние пользовательского интерфейса приватным для пользователя
* Избегаете загромождения URL параметрами запроса

## Пример: Фильтры дашборда

Вот практический пример сохранения фильтров дашборда:

```php title="resources/views/pages/⚡dashboard.blade.php"
<?php

use Livewire\Attributes\Session;
use Livewire\Attributes\Computed;
use Livewire\Component;
use App\Models\Transaction;

new class extends Component {
    #[Session]
    public $dateRange = '30days';

    #[Session]
    public $category = 'all';

    #[Session]
    public $sortBy = 'date';

    #[Computed]
    public function transactions()
    {
        return Transaction::query()
            ->when($this->dateRange === '30days', fn($q) => $q->where('created_at', '>=', now()->subDays(30)))
            ->when($this->category !== 'all', fn($q) => $q->where('category', $this->category))
            ->orderBy($this->sortBy)
            ->get();
    }
};
?>

<div>
    <select wire:model.live="dateRange">
        <option value="7days">Последние 7 дней</option>
        <option value="30days">Последние 30 дней</option>
        <option value="year">Этот год</option>
    </select>

    <select wire:model.live="category">
        <option value="all">Все категории</option>
        <option value="income">Доход</option>
        <option value="expense">Расход</option>
    </select>

    <select wire:model.live="sortBy">
        <option value="date">Дата</option>
        <option value="amount">Сумма</option>
    </select>

    @foreach($this->transactions as $transaction)
        <div wire:key="{{ $transaction->id }}">{{ $transaction->description }}</div>
    @endforeach
</div>
```

Пользователи могут устанавливать предпочтительные фильтры, и они будут сохраняться в течение сессий, обновлений страниц и переходов.

## Соображения производительности

!!! warning "Не храните большие объемы данных"
    Сессии Laravel загружаются в память при каждом запросе. Хранение слишком большого количества данных в сессии пользователя может замедлить работу всего приложения для этого пользователя. Избегайте хранения больших коллекций или объектов.

**Хорошие примеры использования:**

* Простые значения (строки, числа, логические значения)
* Небольшие массивы (параметры фильтров, предпочтения)
* ID моделей (а не целиком модели)

**Плохие примеры использования:**

* Большие коллекции
* Полные модели Eloquent
* Бинарные данные или содержимое файлов

!!! tip "Альтернатива: Устойчивость через URL"
    Если вы хотите, чтобы состояние было доступно для обмена через URL или сохраняемо в закладках, рассмотрите возможность использования [атрибута `#[Url]`](/features/url) вместо `#[Session]`. Параметры URL сохраняют состояние в адресной строке, в то время как свойства сессии сохраняют URL чистым.

## Справочник

```php
<?php

#[Session(
    ?string $key = null,
)]
```

| Параметр | Тип | По умолчанию | Описание |
|-----------|------|---------|-------------|
| `$key` | `?string` | `null` | Пользовательский ключ сессии (генерируется автоматически, если не указан) |
