# Островки

Островки позволяют создавать изолированные области внутри компонента Livewire, которые обновляются независимо друг от друга. Когда действие происходит внутри островка, перерисовывается **только этот островок** — а не весь компонент целиком.

Это даёт вам преимущества производительности от разбиения компонентов на более мелкие части без дополнительных затрат на создание отдельных дочерних компонентов, передачу пропсов или организацию взаимодействия между компонентами.

## Базовое использование

Чтобы создать островок, оберните любую часть вашего Blade-шаблона директивой `@island`:

```php title="resources/views/components/⚡dashboard.blade.php"
<?php

use Livewire\Attributes\Computed;
use Livewire\Component;
use App\Models\Revenue;

new class extends Component {
    #[Computed]
    public function revenue()
    {
        // Дорогостоящие вычисления или запросы...
        return Revenue::yearToDate();
    }
};
?>

<div>
    @island
        <div>
            Revenue: {{ $this->revenue }}

            <button type="button" wire:click="$refresh">Обновить</button>
        </div>
    @endisland

    <div>
        <!-- Другой контент... -->
    </div>
</div>
```

Когда нажимается кнопка «Обновить», перерисовывается только островок, содержащий вычисление дохода. Остальная часть компонента остаётся нетронутой.

Поскольку дорогостоящее вычисление находится внутри вычисляемого свойства — которое выполняется по требованию — оно будет вызвано только при перерисовке островка, а не при обновлении других частей страницы. Однако, так как островки по умолчанию загружаются вместе со страницей, свойство `revenue` всё равно будет вычислено во время начальной загрузки страницы.

## Отложенная загрузка

Иногда у вас есть дорогостоящие вычисления или медленные вызовы API, которые не должны блокировать начальную загрузку страницы. Вы можете отложить начальную отрисовку островка до завершения загрузки страницы, используя параметр `lazy`:

```php title="resources/views/components/⚡dashboard.blade.php"
<?php

use Livewire\Attributes\Computed;
use Livewire\Component;
use App\Models\Revenue;

new class extends Component {
    #[Computed]
    public function revenue()
    {
        // Дорогостоящие вычисления или запросы...
        return Revenue::yearToDate();
    }
};
?>

<div>
    @island(lazy: true)
        <div>
            Revenue: {{ $this->revenue }}

            <button type="button" wire:click="$refresh">Обновить</button>
        </div>
    @endisland

    <div>
        <!-- Другой контент... -->
    </div>
</div>
```

Островок изначально отобразит состояние загрузки, а затем в отдельном запросе получит и отобразит своё содержимое.

### Сравнение ленивой и отложенной загрузки

По умолчанию `lazy` использует intersection observer, чтобы запустить загрузку, когда островок становится видимым в области просмотра. Если вы хотите, чтобы островок загружался сразу после загрузки страницы (независимо от видимости), используйте вместо этого `defer`:

```html
{{-- Загружается при прокрутке в видимую область --}}
@island(lazy: true)
    <!-- ... -->
@endisland

{{-- Загружается сразу после загрузки страницы --}}
@island(defer: true)
    <!-- ... -->
@endisland
```

### Пользовательские состояния загрузки

Вы можете настроить, что будет отображаться, пока ленивый островок загружается, используя директиву `@placeholder`:

```html
@island(lazy: true)
    @placeholder
        <!-- Индикатор загрузки -->
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

## Именованные островки

Чтобы запустить островок из другого места в вашем компоненте, дайте ему имя и ссылайтесь на него с помощью `wire:island`:

```html
<div>
    @island(name: 'revenue')
        <div>
            Доход: {{ $this->revenue }}
        </div>
    @endisland

    <button type="button" wire:click="$refresh" wire:island="revenue">
        Обновить доход
    </button>
</div>
```

Директива `wire:island` работает вместе с директивами действий, такими как `wire:click`, `wire:submit` и т. д., чтобы ограничить область их обновлений конкретным островком.

Когда у вас несколько островков с одинаковым именем, они связываются между собой и всегда будут отрисовываться как группа:

```html
@island(name: 'revenue')
    <div class="sidebar">
        Доход: {{ $this->revenue }}
    </div>
@endisland

@island(name: 'revenue')
    <div class="header">
        Доход: {{ $this->revenue }}
    </div>
@endisland

<button type="button" wire:click="$refresh" wire:island="revenue">
    Обновить доход
</button>
```

Оба островка будут обновляться вместе всякий раз, когда один из них будет вызван.

## Режимы append и prepend

Вместо полной замены содержимого островки могут добавлять новый контент в конец (append) или в начало (prepend). Это идеально подходит для пагинации, бесконечной прокрутки или лент в реальном времени:

```php title="resources/views/components/⚡activity-feed.blade.php"
<?php

use Livewire\Attributes\Computed;
use Livewire\Component;
use App\Models\Activity;

new class extends Component {
    public $page = 1;

    public function loadMore()
    {
        $this->page++;
    }

    #[Computed]
    public function activities()
    {
        return Activity::latest()
            ->forPage($this->page, 10)
            ->get();
    }
};
?>

<div>
    @island(name: 'feed')
        @foreach ($this->activities as $activity)
            <x-activity-item wire:key="{{ $activity->id }}" :activity="$activity" />
        @endforeach
    @endisland

    <button type="button" wire:click="loadMore" wire:island.append="feed">
        Загрузить ещё
    </button>
</div>
```

Доступные режимы:

- `wire:island.append` — Добавить в конец
- `wire:island.prepend` — Добавить в начало

## Вложенные островки

Островки могут быть вложены друг в друга. Когда внешний островок перерисовывается, внутренние островки по умолчанию пропускаются:

```html
@island(name: 'revenue')
    <div>
        Общий доход: {{ $this->revenue }}

        @island(name: 'breakdown')
            <div>
                Разбивка по месяцам: {{ $this->monthlyBreakdown }}

                <button type="button" wire:click="$refresh">
                    Обновить разбивку
                </button>
            </div>
        @endisland

        <button type="button" wire:click="$refresh">
            Обновить доход
        </button>
    </div>
@endisland
```

Нажатие на «Обновить доход» обновляет только внешний островок, в то время как «Обновить разбивку» обновляет только внутренний островок.

## Всегда отрисовывать с родителем

По умолчанию, когда компонент перерисовывается, островки пропускаются. Используйте параметр `always`, чтобы заставить островок обновляться каждый раз, когда обновляется родительский компонент:

```html
<div>
    @island(always: true)
        <div>
            Доход: {{ $this->revenue }}

            <button type="button" wire:click="$refresh">Обновить доход</button>
        </div>
    @endisland

    <button type="button" wire:click="$refresh">Обновить</button>
</div>
```

С параметром `always: true` островок будет перерисовываться каждый раз, когда обновляется любая часть компонента. Это полезно для критически важных данных, которые всегда должны оставаться синхронизированными с состоянием компонента.

Это также работает для вложенных островков — внутренний островок с `always: true` будет обновляться всякий раз, когда обновляется его родительский островок.

## Пропуск начальной отрисовки

Параметр `skip` предотвращает начальную отрисовку островка — идеально для контента, который загружается по требованию:

```html
@island(skip: true)
    @placeholder
        <button type="button" wire:click="$refresh">
            Загрузить детали дохода
        </button>
    @endplaceholder

    <div>
        Доход: {{ $this->revenue }}

        <button type="button" wire:click="$refresh">Обновить</button>
    </div>
@endisland
```

Содержимое placeholder будет отображаться изначально. Когда островок будет вызван, он отрисуется и заменит placeholder.

## Опрос островка

Вы можете использовать `wire:poll` внутри островка, чтобы обновлять только этот островок с заданным интервалом:

```html
@island(skip: true)
    <div wire:poll.3s>
        Доход: {{ $this->revenue }}

        <button type="button" wire:click="$refresh">Обновить</button>
    </div>
@endisland
```

Опрос привязан к островку — только этот островок будет обновляться каждые 3 секунды, а не весь компонент целиком.

## Запуск островков из JavaScript

Директива `wire:island` работает только вместе с директивами действий Livewire, такими как `wire:click`. Чтобы ограничить действие островком из Alpine или JavaScript, используйте `$wire.$island()`:

```html
<button type="button" x-on:click="$wire.$island('feed').loadMore()">
    Загрузить ещё
</button>
```

Это эквивалентно `wire:click="loadMore" wire:island="feed"`, но даёт вам гибкость выражений Alpine и логики JavaScript.

Режимы `append` и `prepend` поддерживаются через параметр `options`:

```html
<button type="button" x-on:click="$wire.$island('feed', { mode: 'append' }).loadMore()">
    Загрузить ещё
</button>
```

Любой метод `$wire` работает с `$island()`, включая `$refresh()`, `$set()` и `$toggle()`:

```html
<button type="button" x-on:click="$wire.$island('revenue').$refresh()">
    Обновить доход
</button>
```

## Важные замечания

Хотя островки обеспечивают мощную изоляцию, имейте в виду:

**Область видимости данных**: Островки имеют доступ к свойствам и методам компонента, но **не** имеют доступа к переменным шаблона, определённым вне островка. Любые переменные `@php` или переменные цикла из родительского шаблона **недоступны** внутри островка:

```html
@php
    $localVariable = 'Это будет недоступно на островке';
@endphp

@island
    {{-- ❌ Это вызовет ошибку — $localVariable недоступна --}}
    {{ $localVariable }}

    {{-- ✅ Свойства компонента работают нормально --}}
    {{ $this->revenue }}
@endisland
```

**Островки нельзя использовать в циклах или условных конструкциях**: Поскольку островки не имеют доступа к переменным цикла или контексту условий, их **нельзя** использовать внутри `@foreach`, `@if` или других управляющих структур:

```html
{{-- ❌ Это не сработает --}}
@foreach ($items as $item)
    @island
        {{ $item->name }}
    @endisland
@endforeach

{{-- ❌ Это тоже не сработает --}}
@if ($showRevenue)
    @island
        Доход: {{ $this->revenue }}
    @endisland
@endif

{{-- ✅ Вместо этого поместите цикл/условие внутрь островка --}}
@island
    @if ($this->showRevenue)
        Доход: {{ $this->revenue }}
    @endif

    @foreach ($this->items as $item)
        {{ $item->name }}
    @endforeach
@endisland
```

**Синхронизация состояния**: Хотя запросы островков выполняются параллельно, и островки, и корневой компонент могут изменять одно и то же состояние компонента. Если одновременно в полёте несколько запросов, состояние может разойтись — победит последнее вернувшееся состояние.

**Когда использовать островки**: Островки наиболее полезны для:

- Дорогостоящих вычислений, которые не должны блокировать начальную загрузку страницы
- Независимых областей со своими собственными взаимодействиями
- Обновлений в реальном времени, затрагивающих только отдельные части интерфейса
- Узких мест производительности в больших компонентах

Островки не нужны для статического контента, тесно связанных частей интерфейса или простых компонентов, которые и без того отрисовываются быстро.

## Смотрите также

- **[Вложенность](/essentials/nesting)** — Альтернативный подход с использованием дочерних компонентов
- **[Отложенная загрузка](/features/lazy)** — Отсрочка загрузки дорогостоящего контента
- **[Вычисляемые свойства](/features/computed-properties)** — Оптимизация производительности островков с помощью мемоизации
- **[@island](/blade-directives/directive-island)** — Создание изолированных регионов обновления
