# Директива `wire:sort`

Livewire предоставляет возможность сортировки перетаскиванием через директиву `wire:sort`. Добавьте её к родительскому элементу и используйте `wire:sort:item` на каждом дочернем элементе, чтобы сделать списки сортируемыми с плавными анимациями из коробки.

## Базовое использование

Чтобы сделать список сортируемым, добавьте `wire:sort` к родительскому элементу с именем метода-обработчика и `wire:sort:item` к каждому дочернему элементу с уникальным идентификатором:

```php
<?php

use Livewire\Component;

new class extends Component {
    public TodoList $list;

    public function handleSort($id, $position)
    {
        $task = $this->list->tasks()->findOrFail($id);

        // Обновляем позицию задачи и переупорядочиваем другие задачи...
    }
};
```

```html
<ul wire:sort="handleSort">
    @foreach ($list->tasks as $task)
        <li wire:key="{{ $task->id }}" wire:sort:item="{{ $task->id }}">
            {{ $task->title }}
        </li>
    @endforeach
</ul>
```

Когда пользователь перетаскивает элемент на новую позицию, Livewire вызовет ваш обработчик с двумя параметрами: идентификатор элемента (из `wire:sort:item`) и новую позицию с отсчётом от нуля.

Вы несёте ответственность за сохранение нового порядка в вашей базе данных.

## Сортировка между группами

Чтобы разрешить перетаскивание элементов между несколькими списками, используйте `wire:sort:group` с одинаковым именем группы для каждого контейнера.

Чтобы определить, в какую группу был помещён элемент, добавьте `wire:sort:group-id` к каждому контейнеру. Его значение будет передано как третий параметр в ваш обработчик:

```php
<?php

use Livewire\Component;
use Livewire\Attributes\Computed;
use App\Models\Card;

new class extends Component {
    public Board $board;

    #[Computed]
    public function columns()
    {
        return $this->board->columns;
    }

    public function handleSort($id, $position, $columnId)
    {
        $card = $this->board->cards()->findOrFail($id);

        // Обновляем позицию карточки и переупорядочиваем другие карточки...
    }
};
```

```html
<div>
    @foreach ($this->columns as $column)
        <ul wire:sort="handleSort" wire:sort:group="cards" wire:sort:group-id="{{ $column->id }}">
            @foreach ($column->cards as $card)
                <li wire:key="{{ $card->id }}" wire:sort:item="{{ $card->id }}">
                    {{ $card->title }}
                </li>
            @endforeach
        </ul>
    @endforeach
</div>
```

Когда элемент перетаскивается в другую группу, срабатывает только обработчик группы назначения.

## Маркеры сортировки

По умолчанию пользователи могут перетаскивать элемент, кликая в любом его месте. Чтобы ограничить перетаскивание определённым маркером, используйте `wire:sort:handle`:

```html
<ul wire:sort="handleSort">
    @foreach ($list->tasks as $task)
        <li wire:key="{{ $task->id }}" wire:sort:item="{{ $task->id }}">
            <div wire:sort:handle>
                <!-- Иконка перетаскивания... -->
            </div>

            {{ $task->title }}
        </li>
    @endforeach
</ul>
```

Теперь пользователи могут инициировать перетаскивание только из элемента-маркера.

## Игнорирование элементов

Чтобы предотвратить запуск операций перетаскивания из определённых областей, используйте `wire:sort:ignore`. Это полезно для кнопок или других интерактивных элементов внутри сортируемых элементов:

```html
<ul wire:sort="handleSort">
    @foreach ($list->tasks as $task)
        <li wire:key="{{ $task->id }}" wire:sort:item="{{ $task->id }}">
            {{ $task->title }}

            <div wire:sort:ignore>
                <button type="button">Редактировать</button>
            </div>
        </li>
    @endforeach
</ul>
```

## Справочник

```php
wire:sort="метод"
wire:sort:item="id"
wire:sort:group="имя"
wire:sort:group-id="идентификатор"
wire:sort:handle
wire:sort:ignore
```

Эта директива не имеет модификаторов.
