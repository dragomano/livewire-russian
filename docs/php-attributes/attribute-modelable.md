# Атрибут `Modelable`

Атрибут `#[Modelable]` помечает свойство в дочернем компоненте, к которому можно привязаться из родительского компонента с помощью `wire:model`.

## Базовое использование

Примените атрибут `#[Modelable]` к свойству в дочернем компоненте, чтобы сделать его доступным для привязки:

```php hl_lines="7" title="resources/views/components/⚡todo-input.blade.php"
<?php

use Livewire\Attributes\Modelable;
use Livewire\Component;

new class extends Component {
    #[Modelable]
    public $value = '';
};
?>

<div>
    <input type="text" wire:model="value">
</div>
```

Теперь родительский компонент может привязываться к этому дочернему компоненту так же, как к любому другому элементу ввода:

```php hl_lines="16" title="resources/views/components/⚡todos.blade.php"
<?php

use Livewire\Component;

new class extends Component {
    public $todo = '';

    public function addTodo()
    {
        // Используйте $this->todo здесь...
    }
};
?>

<div>
    <livewire:todo-input wire:model="todo" />

    <button wire:click="addTodo">Добавить задачу</button>
</div>
```

Когда пользователь вводит текст в компоненте `todo-input`, свойство `$todo` родителя автоматически обновляется.

## Как это работает

Без `#[Modelable]` вам пришлось бы вручную обрабатывать двустороннюю связь между родителем и дочерним компонентом:

```php
<?php

// Без #[Modelable] - ручной подход
<livewire:todo-input
    :value="$todo"
    @input="todo = $event.value"
/>
```

Атрибут `#[Modelable]` упрощает это, позволяя `wire:model` работать напрямую с компонентом.

## Создание переиспользуемых компонентов ввода

`#[Modelable]` идеально подходит для создания кастомных компонентов ввода, которые ощущаются как нативные HTML-инпуты:

```php title="resources/views/components/⚡step-one.blade.php"
<?php

use Livewire\Attributes\Modelable;
use Livewire\Component;

new class extends Component {
    #[Modelable]
    public $date = '';
};
?>

<div>
    <input
        type="date"
        wire:model="date"
        class="border rounded px-3 py-2"
    >
</div>
```

```php
<?php

{{-- Использование в родителе --}}
<livewire:date-picker wire:model="startDate" />
<livewire:date-picker wire:model="endDate" />
```

!!! warning "Предупреждение"
    Корневой элемент вашего компонента не может быть элементом управления формой с `wire:model`. Оберните ваш инпут в обёртку, например `<div>`. Livewire внедряет `wire:model` и `x-modelable` в корневой элемент для связи с родителем — второй `wire:model` на том же элементе вызовет конфликт.

## Модификаторы

Родитель может использовать модификаторы `wire:model` для управления временем обновления и сетевыми запросами:

```php
<?php

{{-- Живые обновления при каждом нажатии клавиши --}}
<livewire:todo-input wire:model.live="todo" />

{{-- Обновления с задержкой (debounce) --}}
<livewire:todo-input wire:model.live.debounce.500ms="todo" />

{{-- Ограничение частоты обновлений (throttle) --}}
<livewire:todo-input wire:model.live.throttle.500ms="todo" />
```

!!! note "Модификаторы на основе событий в компонентах"
    Модификаторы на основе событий, такие как `.blur`, `.change` и `.enter`, управляют событиями DOM на конкретных элементах, а не реактивными привязками компонентов. Чтобы управлять временем синхронизации для modelable-компонентов, поместите эти модификаторы на сам элемент ввода внутри дочернего компонента:

    ```php
    <?php

    {{-- Родитель --}}
    <livewire:todo-input wire:model="todo" />

    {{-- Дочерний компонент --}}
    <input wire:model.blur="value" />
    ```

## Пример: Кастомный редактор форматированного текста

Вот более сложный пример компонента редактора форматированного текста:

```php title="resources/views/components/⚡rich-editor.blade.php"
<?php

use Livewire\Attributes\Modelable;
use Livewire\Component;

new class extends Component {
    #[Modelable]
    public $content = '';
};
?>

<div>
    <div
        x-init="
            // Инициализируйте вашу библиотеку редактора здесь
            editor.on('change', () => {
                $wire.content = editor.getContent()
            })
        "
    >
        <!-- UI редактора -->
    </div>
</div>
```

```php
<?php

{{-- Использование --}}
<livewire:rich-editor wire:model="postContent" />
```

## Ограничения

!!! warning "Только одно modelable-свойство на компонент"
    В настоящее время Livewire поддерживает только один атрибут `#[Modelable]` на компонент, поэтому будет привязано только первое из них.

## Когда использовать

Используйте `#[Modelable]`, когда:

* Создаете переиспользуемые компоненты ввода (выбор даты, выбор цвета, редакторы текста)
* Создаете компоненты форм, которые должны работать с `wire:model`
* Оборачиваете сторонние JavaScript-библиотеки в виде компонентов Livewire
* Создаете кастомные инпуты со специальной валидацией или форматированием

## Узнать больше

Для получения дополнительной информации о взаимодействии родительских и дочерних компонентов и привязке данных см. [документацию по вложенным компонентам](/essentials/nesting#привязка-к-данным-дочернего-компонента-с-помощью-wiremodel).
