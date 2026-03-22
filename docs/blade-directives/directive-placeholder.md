# Директива `@placeholder`

Директива `@placeholder` отображает пользовательский контент во время загрузки ленивых или отложенных компонентов и островков.

## Базовое использование с ленивыми компонентами

Для однофайловых и многофайловых компонентов используйте `@placeholder`, чтобы указать, что должно отображаться во время загрузки:

```php title="resources/views/components/⚡revenue.blade.php"
<?php

use Livewire\Component;
use App\Models\Transaction;

new class extends Component {
    public $amount;

    public function mount()
    {
        // Медленный запрос к базе данных...
        $this->amount = Transaction::monthToDate()->sum('amount');
    }
};
?>

@placeholder
    <div>
        <!-- Индикатор загрузки -->
        <svg class="animate-spin h-5 w-5">...</svg>
    </div>
@endplaceholder

<div>
    Доход за этот месяц: {{ $amount }}
</div>
```

При рендеринге с помощью `<livewire:revenue lazy />` плейсхолдер будет отображаться до тех пор, пока компонент не загрузится.

!!! tip "Только для компонентов на основе представлений"
    Директива `@@placeholder` работает для однофайловых и многофайловых компонентов. Для компонентов на основе классов используйте метод `placeholder()`.

!!! warning "Соответствие типов корневых элементов"
    Плейсхолдер и компонент должны иметь один и тот же тип корневого элемента. Если ваш плейсхолдер использует `<div>`, ваш компонент также должен использовать `<div>`.

## Использование с островками

Используйте `@placeholder` внутри ленивых островков для настройки состояний загрузки:

```html
@island(lazy: true)
    @placeholder
        <div class="animate-pulse">
            <div class="h-32 bg-gray-200 rounded"></div>
        </div>
    @endplaceholder

    <div>
        Доход: {{ $this->revenue }}

        <button type="button" wire:click="$refresh">Обновить</button>
    </div>
@endisland
```

Плейсхолдер появляется во время загрузки островка, а затем заменяется фактическим содержимым.

## Скелетные плейсхолдеры

Плейсхолдеры идеально подходят для создания скелетных загрузчиков (skeleton loaders), соответствующих макету вашего контента:

```html
@placeholder
    <div class="space-y-4">
        <div class="h-4 bg-gray-200 rounded w-3/4"></div>
        <div class="h-4 bg-gray-200 rounded"></div>
        <div class="h-4 bg-gray-200 rounded w-5/6"></div>
    </div>
@endplaceholder

<div>
    <!-- Фактический контент -->
    <h2>{{ $post->title }}</h2>
    <p>{{ $post->content }}</p>
</div>
```

## Узнать больше

Для получения информации о ленивой загрузке компонентов см. [документацию по ленивой загрузке](/features/lazy).

Для получения информации о состояниях загрузки островков см. [документацию по островкам](/features/islands).
