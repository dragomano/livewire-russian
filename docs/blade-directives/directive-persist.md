# Директива `@persist`

Директива `@persist` сохраняет элементы при навигации по страницам с использованием `wire:navigate`, поддерживая их состояние и избегая повторной инициализации.

## Базовое использование

Оберните элемент в `@persist` и укажите уникальное имя, чтобы сохранить его при переходах между страницами:

```html
@persist('player')
    <audio src="{{ $episode->file }}" controls></audio>
@endpersist
```

При переходе на новую страницу, которая также содержит сохранённый элемент с тем же именем, Livewire повторно использует существующий DOM-элемент вместо создания нового. Для аудиоплеера это означает, что воспроизведение продолжится без прерывания.

!!! tip "Требуется `wire:navigate`"
    Директива `@persist` работает только тогда, когда навигация осуществляется с помощью функции Livewire `wire:navigate`. Стандартная загрузка страниц не сохраняет элементы.

## Распространённые варианты использования

**Аудио/видеоплееры**

```html
@persist('podcast-player')
    <audio src="{{ $episode->audio_url }}" controls></audio>
@endpersist
```

**Виджеты чата**

```html
@persist('support-chat')
    <div id="chat-widget">
        <!-- Интерфейс чата... -->
    </div>
@endpersist
```

**Сторонние виджеты**

```html
@persist('analytics-widget')
    <div id="analytics-dashboard">
        <!-- Сложный виджет, инициализация которого обходится дорого... -->
    </div>
@endpersist
```

## Размещение в макетах

Сохраняемые элементы обычно следует размещать вне компонентов Livewire, чаще всего в основном макете:

```html title="resources/views/layouts/app.blade.php"
<!DOCTYPE html>
<html lang="{{ str_replace('_', '-', app()->getLocale()) }}">
    <head>
        <meta charset="utf-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">

        <title>{{ $title ?? config('app.name') }}</title>

        @vite(['resources/css/app.css', 'resources/js/app.js'])

        @livewireStyles
    </head>
    <body>
        <main>
            {{ $slot }}
        </main>

        @persist('player')
            <audio src="{{ $episode->file }}" controls></audio>
        @endpersist

        @livewireScripts
    </body>
</html>
```

## Сохранение позиции прокрутки

Для прокручиваемых сохраняемых элементов добавьте `wire:navigate:scroll`, чтобы сохранить позицию прокрутки:

```html
@persist('scrollable-list')
    <div class="overflow-y-scroll" wire:navigate:scroll>
        <!-- Прокручиваемый контент... -->
    </div>
@endpersist
```

## Подсветка активных ссылок

Внутри сохраняемых элементов используйте `wire:current` вместо серверных условий для выделения активных ссылок:

```html
@persist('navigation')
    <nav>
        <a href="/dashboard" wire:navigate wire:current="font-bold">Дашборд</a>
        <a href="/posts" wire:navigate wire:current="font-bold">Посты</a>
        <a href="/users" wire:navigate wire:current="font-bold">Пользователи</a>
    </nav>
@endpersist
```

[Узнать больше о `wire:current` →](/html-directives/wire-current)

## Как это работает

При навигации с помощью `wire:navigate`:

1. Livewire ищет элементы с совпадающими именами `@persist` на обеих страницах.
2. Если они найдены, существующий элемент перемещается в DOM новой страницы.
3. Состояние элемента, прослушиватели событий и данные Alpine сохраняются.

[Узнать больше о навигации →](/features/navigate)

## Справочник

```html
@persist(string $key)
    <!-- Контент -->
@endpersist
```

| Параметр | Тип | По умолчанию | Описание |
|-----------|------|---------|-------------|
| `$key` | `string` | *обязательно* | Уникальное имя, идентифицирующее элемент для сохранения при навигации по страницам |
