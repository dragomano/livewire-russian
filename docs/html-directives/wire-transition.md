# Директива `wire:transition`

`wire:transition` обеспечивает плавные анимации при появлении, исчезновении или изменении элементов с использованием нативного [View Transitions API](https://developer.mozilla.org/en-US/docs/Web/API/View_Transitions_API) браузера.

В отличие от библиотек анимации на основе JavaScript, View Transitions аппаратно ускоряются и обрабатываются нативно браузером, что обеспечивает более плавные анимации с меньшими накладными расходами.

## Базовое использование

Добавьте `wire:transition` к любому элементу, который может быть добавлен, удалён или изменён во время обновления Livewire:

```php
<?php

class ShowPost extends Component
{
    public Post $post;

    public $showComments = false;
}
```

```html hl_lines="5"
<div>
    <button wire:click="$toggle('showComments')">Показать/скрыть комментарии</button>

    @if ($showComments)
        <div wire:transition>
            @foreach ($post->comments as $comment)
                <div>{{ $comment->body }}</div>
            @endforeach
        </div>
    @endif
</div>
```

При появлении или исчезновении комментариев браузер будет плавно применять эффект перекрёстного затухания вместо резкого показа или скрытия.

## Именованные переходы

По умолчанию Livewire присваивает элементам с `wire:transition` имя view-transition-name `match-element`. Вы можете предоставить собственное имя для включения более продвинутых эффектов перехода:

```html
<div wire:transition="sidebar">...</div>
```

Это устанавливает CSS-свойство `view-transition-name` элемента в `sidebar`, на которое можно настроить пользовательские анимации через CSS.

## Настройка анимаций с помощью CSS

View Transitions полностью управляются через CSS. Вы можете настроить анимацию, обратившись к псевдоэлементам view-transition:

```css
/* Настройка перехода для конкретного элемента */
::view-transition-old(sidebar) {
    animation: 300ms ease-out both slide-out;
}

::view-transition-new(sidebar) {
    animation: 300ms ease-in both slide-in;
}

@keyframes slide-out {
    to { transform: translateX(-100%); }
}

@keyframes slide-in {
    from { transform: translateX(100%); }
}
```

View Transitions API предоставляет три псевдоэлемента, которые можно стилизовать:

- `::view-transition-old(name)` — Снимок уходящего элемента
- `::view-transition-new(name)` — Снимок входящего элемента
- `::view-transition-group(name)` — Контейнер для обоих снимков

## Типы переходов

Для более сложных сценариев, таких как пошаговые мастера, где вам нужны разные анимации в зависимости от направления, вы можете использовать типы переходов. Это позволяет анимировать «вперёд» и «назад» по-разному.

Используйте метод `$this->transition()` для установки типа перехода:

```php
<?php

class Wizard extends Component
{
    public $step = 1;

    public function goToStep($step)
    {
        $this->transition(type: $step > $this->step ? 'forward' : 'backward');

        $this->step = $step;
    }
}
```

Затем настройте тип в вашем CSS с помощью селектора `:active-view-transition-type()`:

```css
html:active-view-transition-type(forward) {
    &::view-transition-old(content) {
        animation: 300ms ease-out both slide-out-left;
    }
    &::view-transition-new(content) {
        animation: 300ms ease-in both slide-in-right;
    }
}

html:active-view-transition-type(backward) {
    &::view-transition-old(content) {
        animation: 300ms ease-out both slide-out-right;
    }
    &::view-transition-new(content) {
        animation: 300ms ease-in both slide-in-left;
    }
}

@keyframes slide-out-left {
    from { transform: translateX(0); opacity: 1; }
    to { transform: translateX(-100%); opacity: 0; }
}

@keyframes slide-in-right {
    from { transform: translateX(100%); opacity: 0; }
    to { transform: translateX(0); opacity: 1; }
}

@keyframes slide-out-right {
    from { transform: translateX(0); opacity: 1; }
    to { transform: translateX(100%); opacity: 0; }
}

@keyframes slide-in-left {
    from { transform: translateX(-100%); opacity: 0; }
    to { transform: translateX(0); opacity: 1; }
}
```

Для методов, которые всегда переходят в одном направлении, вы можете использовать атрибут `#[Transition]`:

```php
<?php

use Livewire\Attributes\Transition;

class Wizard extends Component
{
    public $step = 1;

    #[Transition(type: 'forward')]
    public function next()
    {
        $this->step++;
    }

    #[Transition(type: 'backward')]
    public function previous()
    {
        $this->step--;
    }
}
```

## Пропуск переходов

Иногда вам может потребоваться отключить переходы для определённых действий — например, кнопка «сброс», которая должна мгновенно переходить к первому шагу без анимации.

Используйте `$this->skipTransition()` для отключения переходов для текущего запроса:

```php
<?php

public function reset()
{
    $this->skipTransition();

    $this->step = 1;
}
```

Или используйте атрибут `#[Transition]` с `skip: true`:

```php
<?php

use Livewire\Attributes\Transition;

#[Transition(skip: true)]
public function reset()
{
    $this->step = 1;
}
```

## Учёт предпочтений по уменьшению движения

Livewire автоматически учитывает настройку пользователя [prefers-reduced-motion](https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/At-rules/@media/prefers-reduced-motion#user_preferences). При её включении переходы отключаются, чтобы избежать дискомфорта у пользователей, чувствительных к движению.

## Поддержка браузерами

View Transitions поддерживаются в Chrome 111+, Edge 111+ и Safari 18+. В браузерах, которые не поддерживают View Transitions, элементы будут появляться и исчезать без анимации — функциональность продолжает работать, просто без визуального перехода.

!!! warning "Ограниченная поддержка в Firefox"
    Firefox 144+ поддерживает базовые view transitions, но не поддерживает типы переходов.

[Поддержка браузерами на caniuse.com →](https://caniuse.com/view-transitions)

## Смотрите также

- **[wire:show](/html-directives/wire-show)** — Переключение видимости с помощью CSS display
- **[Состояния загрузки](/features/loading-states)** — Отображение индикаторов загрузки во время запросов
- **[Переходы Alpine](https://alpinejs.dragomano.ru/directives/transition)** — Для более сложных анимаций

## Справочник

```php
wire:transition="имя"
```

| Выражение | Описание |
|------------|-------------|
| (нет) | Использует `match-element` как view-transition-name |
| `"имя"` | Использует переданную строку как view-transition-name |

Эта директива не имеет модификаторов.
