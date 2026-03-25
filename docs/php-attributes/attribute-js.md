# Атрибут `Js`

Атрибут `#[Js]` обозначает методы, которые возвращают JavaScript-код для выполнения на стороне клиента. Методы, помеченные `#[Js]`, можно вызывать напрямую из ваших шаблонов без отправки запроса на сервер.

## Базовое использование

Примените атрибут `#[Js]` к методам, которые возвращают JavaScript-выражения:

```php hl_lines="10-17" title="resources/views/components/post/⚡create.blade.php"
<?php

use Livewire\Attributes\Js;
use Livewire\Component;

new class extends Component {
    public $title = '';
    public $content = '';

    #[Js]
    public function resetForm()
    {
        return <<<'JS'
            $wire.title = ''
            $wire.content = ''
        JS;
    }
};
```

```html hl_lines="6"
<form wire:submit="save">
    <input wire:model="title" placeholder="Заголовок">
    <textarea wire:model="content" placeholder="Контент"></textarea>

    <button type="submit">Сохранить</button>
    <button type="button" @click="$wire.resetForm()">Сброс</button>
</form>
```

Когда вызывается `$wire.resetForm()`, JavaScript выполняется непосредственно в браузере — обращения к серверу не происходит.

## Выполнение JavaScript после серверных действий

Если вам нужно выполнить JavaScript **после завершения серверного действия**, используйте вместо этого метод `js()`:

```php hl_lines="13" title="resources/views/components/post/⚡create.blade.php"
<?php

use Livewire\Component;
use App\Models\Post;

new class extends Component {
    public $title = '';

    public function save()
    {
        Post::create(['title' => $this->title]);

        $this->js("alert('Пост успешно сохранён!')");
    }
};
```

Метод `js()` ставит JavaScript в очередь на выполнение при получении ответа от сервера.

## Доступ к $wire

Вы можете получить доступ к объекту компонента `$wire` внутри JavaScript-выражений:

```php
<?php

#[Js]
public function resetForm()
{
    return <<<'JS'
        $wire.title = ''
        $wire.content = ''
    JS;
}
```

## Когда использовать

Используйте `#[Js]`, когда вам нужно:

* Сбросить или очистить поля формы без нагрузки на сервер
* Запустить JavaScript-анимации или переходы
* Обновить состояние на стороне клиента без повторного рендеринга
* Выполнить повторно используемую JavaScript-логику из нескольких мест
* Интегрироваться со сторонними JavaScript-библиотеками

## JavaScript-действия в сравнении с методами #[Js]

Существует важное различие:

* **Методы `#[Js]`** определяются в PHP и возвращают JavaScript-код. Они вызываются через `$wire.methodName()` без выполнения запроса к серверу.
* **JavaScript-действия** (`$js.methodName`) определяются полностью в JavaScript с использованием блоков `@script`.

Оба подхода выполняют JavaScript на клиенте без обращения к серверу. Разница заключается в том, где определен JavaScript-код.

```php title="resources/views/components/⚡example.blade.php"
<?php

use Livewire\Attributes\Js;
use Livewire\Component;

new class extends Component {
    public $count = 0;

    #[Js]
    public function showCount()
    {
        return "alert('Счётчик: {$this->count}')";
    }
};
```

```html
<div>
    <button @click="$wire.showCount()">Показать счётчик (из PHP)</button>
    <button @click="$js.incrementLocal()">Увеличить локально (из JS)</button>
</div>

@script
<script>
    $js('incrementLocal', () => {
        console.log('Запросов к серверу не было')
    })
</script>
@endscript
```

## Узнать больше

Для получения дополнительной информации об интеграции JavaScript в Livewire см.:

* [Документация по JavaScript](/advanced/javascript)
* [Документация по JavaScript-действиям](/essentials/actions#javascript-действия)
