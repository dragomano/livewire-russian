# Директива `wire:intersect`

Директива `wire:intersect` в Livewire позволяет выполнять действие при вхождении элемента в область видимости или выходе из неё. Это полезно для отложенной загрузки контента, запуска аналитики или создания взаимодействий на основе прокрутки.

## Базовое использование

Простейшая форма запускает действие при видимости элемента:

```html
<div wire:intersect="loadMore">
    <!-- Контент загружается при прокрутке в область видимости -->
</div>
```

Когда элемент входит в область видимости, будет вызвано действие `loadMore` на вашем компоненте.

## События входа и выхода

Вы можете указать, запускать ли действие при входе, выходе или обоих событиях:

```html
<!-- Запускается при входе в область видимости (по умолчанию) -->
<div wire:intersect="trackView">...</div>

<!-- Запускается при входе в область видимости (явно) -->
<div wire:intersect:enter="trackView">...</div>

<!-- Запускается при выходе из области видимости -->
<div wire:intersect:leave="pauseVideo">...</div>
```

## Модификаторы видимости

Управляйте тем, какая часть элемента должна быть видна перед запуском:

```html
<!-- Запуск при видимости любой части (по умолчанию) -->
<div wire:intersect="load">...</div>

<!-- Запуск при видимости половины -->
<div wire:intersect.half="load">...</div>

<!-- Запуск при полной видимости -->
<div wire:intersect.full="load">...</div>

<!-- Запуск при пользовательском пороге (0-100) -->
<div wire:intersect.threshold.25="load">...</div>
```

## Отступ

Добавьте отступ вокруг области видимости для запуска действия до/после входа элемента:

```html
<!-- Запуск за 200 пикселей до входа в область видимости -->
<div wire:intersect.margin.200px="loadMore">...</div>

<!-- Использование отступа в процентах -->
<div wire:intersect.margin.10%="loadMore">...</div>

<!-- Разные отступы для каждой стороны (верх, право, низ, лево) -->
<div wire:intersect.margin.10%.25px.25px.25px="loadMore">...</div>
```

## Однократный запуск

Используйте модификатор `.once`, чтобы действие срабатывало только при первом пересечении:

```html
<div wire:intersect.once="trackImpression">
    <!-- Действие срабатывает только один раз, даже если прокрутка происходит несколько раз -->
</div>
```

Это особенно полезно для аналитики или отслеживания, когда вы хотите записать только первый раз, когда пользователь что-то видит.

## Комбинирование модификаторов

Вы можете комбинировать несколько модификаторов для создания точного поведения:

```html
<!-- Загрузка при видимости половины, только один раз, с отступом 100 пикселей -->
<div wire:intersect.once.half.margin.100px="loadSection">
    <!-- ... -->
</div>
```

## Распространённые варианты использования

### Бесконечная прокрутка

```php
<?php

use Livewire\Component;

new class extends Component {
    public $page = 1;
    public $posts = [];

    public function mount()
    {
        $this->loadPosts();
    }

    public function loadPosts()
    {
        $newPosts = Post::latest()
            ->skip(($this->page - 1) * 10)
            ->take(10)
            ->get();

        $this->posts = array_merge($this->posts, $newPosts->toArray());
        $this->page++;
    }
};
?>

<div>
    @foreach ($posts as $post)
        <div>{{ $post['title'] }}</div>
    @endforeach

    <div wire:intersect="loadPosts">
        Загрузка дополнительных постов...
    </div>
</div>
```

### Отложенная загрузка изображений

```php
<?php

use Livewire\Component;

new class extends Component {
    public $imageLoaded = false;

    public function loadImage()
    {
        $this->imageLoaded = true;
    }
};
?>

<div>
    @if ($imageLoaded)
        <img src="/path/to/image.jpg" alt="Продукт">
    @else
        <div wire:intersect.once="loadImage" class="bg-gray-200 h-64">
            <!-- Заполнитель -->
        </div>
    @endif
</div>
```

### Отслеживание видимости

```html
<div wire:intersect:enter.once="trackView" wire:intersect:leave="trackLeave">
    <!-- Отслеживание просмотров и уходов пользователей с этого контента -->
</div>
```

## Сравнение с x-intersect в Alpine

Если вы знакомы с Alpine.js, `wire:intersect` работает аналогично `x-intersect`, но запускает действия Livewire вместо выражений Alpine. Модификаторы и поведение разработаны так, чтобы быть знакомыми пользователям Alpine.

## Справочник

```php
wire:intersect="действие"
wire:intersect:enter="действие"
wire:intersect:leave="действие"
```

### Модификаторы

| Модификатор | Описание |
|-------------|----------|
| `.once` | Запускать действие только при первом пересечении |
| `.half` | Запуск при видимости половины элемента |
| `.full` | Запуск при полной видимости элемента |
| `.threshold.[0-100]` | Запуск при пользовательском пороге видимости в процентах |
| `.margin.[значение]` | Добавить отступ вокруг области видимости (например, `.margin.200px`, `.margin.10%`) |
