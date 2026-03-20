# Синтезаторы

Поскольку компоненты Livewire дегидратируются (сериализуются) в JSON, а затем гидратируются (десериализуются) обратно в PHP-объекты между запросами, их свойства должны быть пригодны для сериализации в JSON.

По умолчанию PHP легко сериализует большинство примитивных значений в JSON. Однако, чтобы компоненты Livewire могли поддерживать более сложные типы свойств (такие как модели, коллекции, экземпляры Carbon и Stringable), требуется более мощная система.

Поэтому Livewire предоставляет точку расширения под названием «Синтезаторы», которая позволяет пользователям поддерживать любые собственные типы свойств по своему усмотрению.

!!! tip "Сначала разберитесь с гидратацией"
    Перед использованием синтезаторов полезно полностью понять систему гидратации Livewire. Вы можете узнать больше, прочитав [документацию по гидратации](/advanced/hydration).

## Понимание синтезаторов

Прежде чем изучать создание пользовательских синтезаторов, давайте сначала посмотрим на внутренний синтезатор, который Livewire использует для поддержки [Laravel Stringables](https://laravel.com/docs/strings).

Предположим, ваше приложение содержит следующий компонент `CreatePost`:

```php
<?php

class CreatePost extends Component
{
    public $title = '';
}
```

Между запросами Livewire может сериализовать состояние этого компонента в объект JSON, подобный следующему:

```js
state: { title: '' },
```

Теперь рассмотрим более сложный пример, когда значение свойства `$title` является объектом `stringable` вместо обычной строки:

```php
<?php

class CreatePost extends Component
{
    public $title = '';

    public function mount()
    {
        $this->title = str($this->title);
    }
}
```

Дегидратированный JSON, представляющий состояние этого компонента, теперь содержит [кортеж метаданных](/advanced/hydration#глубоко-вложенные-кортежи) вместо простой пустой строки:

```js
state: { title: ['', { s: 'str' }] },
```

Livewire теперь может использовать этот кортеж для обратной гидратации свойства `$title` в `stringable` при следующем запросе.

Теперь, когда вы увидели внешние эффекты работы синтезаторов, вот реальный исходный код внутреннего синтезатора Livewire для `stringable`:

```php
<?php

use Illuminate\Support\Stringable;

class StringableSynth extends Synth
{
    public static $key = 'str';

    public static function match($target)
    {
        return $target instanceof Stringable;
    }

    public function dehydrate($target)
    {
        return [$target->__toString(), []];
    }

    public function hydrate($value)
    {
        return str($value);
    }
}
```

Давайте разберем это по частям.

Первое — это свойство `$key`:

```php
<?php

public static $key = 'str';
```

Каждый синтезатор должен содержать статическое свойство `$key`, которое Livewire использует для преобразования [кортежа метаданных](/advanced/hydration#глубоко-вложенные-кортежи), такого как `['', { s: 'str' }]`, обратно в `stringable`. Как вы можете заметить, каждый кортеж метаданных имеет ключ `s`, ссылающийся на этот ключ.

И наоборот, когда Livewire выполняет дегидратацию свойства, он использует статическую функцию синтезатора `match()`, чтобы определить, подходит ли данный конкретный синтезатор для дегидратации текущего свойства (`$target` — это текущее значение свойства):

```php
<?php

public static function match($target)
{
    return $target instanceof Stringable;
}
```

Если `match()` возвращает `true`, будет использован метод `dehydrate()`, который принимает PHP-значение свойства в качестве входных данных и возвращает JSON-совместимый [кортеж метаданных](/advanced/hydration#глубоко-вложенные-кортежи):

```php
<?php

public function dehydrate($target)
{
    return [$target->__toString(), []];
}
```

Затем, в начале следующего запроса, после того как этот синтезатор был найден по ключу `{ s: 'str' }` в кортеже, будет вызван метод `hydrate()`, которому будет передано необработанное JSON-представление свойства с ожиданием, что он вернет полное PHP-совместимое значение для присвоения свойству.

```php
<?php

public function hydrate($value)
{
    return str($value);
}
```

## Регистрация пользовательского синтезатора

Чтобы продемонстрировать, как вы можете написать собственный синтезатор для поддержки пользовательского свойства, мы будем использовать следующий компонент `UpdateProperty` в качестве примера:

```php
<?php

class UpdateProperty extends Component
{
    public Address $address;

    public function mount()
    {
        $this->address = new Address();
    }
}
```

Вот исходный код класса `Address`:

```php
<?php

namespace App\Dtos\Address;

class Address
{
    public $street = '';
    public $city = '';
    public $state = '';
    public $zip = '';
}
```

Для поддержки свойств типа `Address` мы можем использовать следующий синтезатор:

```php
<?php

use App\Dtos\Address;

class AddressSynth extends Synth
{
    public static $key = 'address';

    public static function match($target)
    {
        return $target instanceof Address;
    }

    public function dehydrate($target)
    {
        return [[
            'street' => $target->street,
            'city' => $target->city,
            'state' => $target->state,
            'zip' => $target->zip,
        ], []];
    }

    public function hydrate($value)
    {
        $instance = new Address;

        $instance->street = $value['street'];
        $instance->city = $value['city'];
        $instance->state = $value['state'];
        $instance->zip = $value['zip'];

        return $instance;
    }
}
```

Чтобы сделать его доступным глобально в вашем приложении, вы можете использовать метод Livewire `propertySynthesizer` для регистрации синтезатора в методе `boot` вашего сервис-провайдера:

```php
<?php

class AppServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Livewire::propertySynthesizer(AddressSynth::class);
    }
}
```

## Поддержка привязки данных

Используя пример `UpdateProperty` выше, вполне вероятно, что вы захотите поддерживать привязку `wire:model` напрямую к свойствам объекта `Address`. Синтезаторы позволяют поддерживать это с помощью методов `get()` и `set()`:

```php hl_lines="36-44"
<?php

use App\Dtos\Address;

class AddressSynth extends Synth
{
    public static $key = 'address';

    public static function match($target)
    {
        return $target instanceof Address;
    }

    public function dehydrate($target)
    {
        return [[
            'street' => $target->street,
            'city' => $target->city,
            'state' => $target->state,
            'zip' => $target->zip,
        ], []];
    }

    public function hydrate($value)
    {
        $instance = new Address;

        $instance->street = $value['street'];
        $instance->city = $value['city'];
        $instance->state = $value['state'];
        $instance->zip = $value['zip'];

        return $instance;
    }

    public function get(&$target, $key)
    {
        return $target->{$key};
    }

    public function set(&$target, $key, $value)
    {
        $target->{$key} = $value;
    }
}
```
