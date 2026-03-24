# Атрибут `Computed`

Атрибут `#[Computed]` позволяет создавать производные свойства, которые кэшируются во время запроса, обеспечивая преимущество в производительности при многократном обращении к ресурсозатратным операциям.

## Базовое использование

Примените атрибут `#[Computed]` к любому методу, чтобы превратить его в кэшируемое свойство:

```php hl_lines="10" title="resources/views/components/user/⚡show.blade.php"
<?php

use Livewire\Attributes\Computed;
use Livewire\Component;
use App\Models\User;

new class extends Component {
    public $userId;

    #[Computed]
    public function user()
    {
        return User::find($this->userId);
    }

    public function follow()
    {
        Auth::user()->follow($this->user);
    }
};
```

```html
<div>
    <h1>{{ $this->user->name }}</h1>
    <span>{{ $this->user->email }}</span>

    <button wire:click="follow">Подписаться</button>
</div>
```

Доступ к методу `user()` осуществляется как к свойству через `$this->user`. При первом вызове результат кэшируется и повторно используется до конца запроса.

!!! info "Необходимо использовать `$this` в шаблонах"
    В отличие от обычных свойств, к вычисляемым свойствам в шаблоне нужно обращаться через `$this`. Например, `$this->posts` вместо `$posts`.

## Преимущество в производительности

Вычисляемые свойства кэшируют свой результат на время выполнения запроса. Если вы обращаетесь к `$this->posts` несколько раз, базовый метод выполнится только один раз:

```php hl_lines="8" title="resources/views/components/post/⚡index.blade.php"
<?php

use Illuminate\Support\Facades\Auth;
use Livewire\Attributes\Computed;
use Livewire\Component;

new class extends Component {
    #[Computed]
    public function posts()
    {
        return Auth::user()->posts; // Выполняет запрос к базе данных только один раз
    }
};
```

Это позволяет свободно обращаться к производным значениям, не беспокоясь о влиянии на производительность.

## Сброс кэша

Если базовые данные изменяются во время запроса, вы можете очистить кэш с помощью `unset()`:

```php hl_lines="22 title="resources/views/components/post/⚡index.blade.php"
<?php

use Illuminate\Support\Facades\Auth;
use Livewire\Attributes\Computed;
use Livewire\Component;

new class extends Component {
    #[Computed]
    public function posts()
    {
        return Auth::user()->posts;
    }

    public function createPost()
    {
        if ($this->posts->count() > 10) {
            throw new \Exception('Превышено максимальное количество постов');
        }

        Auth::user()->posts()->create(...);

        unset($this->posts); // Очистка кэша
    }
};
```

После создания нового поста `unset($this->posts)` очищает кэш, поэтому при следующем обращении будут получены обновлённые данные.

## Кэширование между запросами

По умолчанию вычисляемые свойства кэшируются только в рамках одного запроса. Чтобы кэшировать их между несколькими запросами, используйте параметр `persist`:

```php hl_lines="1"
<?php

#[Computed(persist: true)]
public function user()
{
    return User::find($this->userId);
}
```

Это закэширует значение на 3600 секунд (1 час). Вы можете настроить длительность:

```php hl_lines="1"
<?php

#[Computed(persist: true, seconds: 7200)] // 2 часа
```

## Кэширование для всех компонентов

Чтобы совместно использовать закэшированные значения во всех экземплярах компонентов в вашем приложении, используйте параметр `cache`:

```php hl_lines="4"
<?php

use Livewire\Attributes\Computed;
use App\Models\Post;

#[Computed(cache: true)]
public function posts()
{
    return Post::all();
}
```

Вы также можете при желании установить собственный ключ кэша:

```php hl_lines="1"
<?php

#[Computed(cache: true, key: 'homepage-posts')]
```

## Когда использовать

Вычисляемые свойства особенно полезны, когда:

* **Условный доступ к ресурсозатратным данным** — запрос к базе данных выполняется только в том случае, если значение действительно используется в шаблоне.
* **Использование встроенных шаблонов** — нет возможности передать данные через `render()`.
* **Отсутствие метода render** — следование соглашению v4 о компонентах в одном файле.
* **Многократный доступ к одному и тому же значению** — автоматическое кэширование предотвращает избыточные запросы.

## Ограничения

!!! warning "Не поддерживается в объектах Form"
    Вычисляемые свойства нельзя использовать в объектах `Livewire\Form`. Попытка доступа к ним через `$form->property` приведет к ошибке.

## Узнать больше

Для получения подробной информации о вычисляемых свойствах, стратегиях кэширования и продвинутых сценариях использования см. [документацию по вычисляемым свойствам](/features/computed-properties).

## Справочник

```php
<?php

#[Computed(
    bool $persist = false,
    int $seconds = 3600,
    bool $cache = false,
    ?string $key = null,
    mixed $tags = null,
)]
```

| Параметр | Тип | По умолчанию | Описание |
|-----------|------|---------|-------------|
| `$persist` | `bool` | `false` | Кэшировать значение между запросами для того же экземпляра компонента |
| `$seconds` | `int` | `3600` | Длительность кэширования значения в секундах |
| `$cache` | `bool` | `false` | Кэшировать значение для всех экземпляров компонентов |
| `$key` | `?string` | `null` | Пользовательский ключ кэша (автоматически генерируется, если не указан) |
| `$tags` | `mixed` | `null` | Теги кэша (требуется драйвер кэша, поддерживающий теги) |
