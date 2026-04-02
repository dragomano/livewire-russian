# Атрибут `Transition`

Атрибут `#[Transition]` настраивает поведение перехода представления для методов действия, позволяя устанавливать типы переходов или полностью их отключать.

## Базовое использование

Примените атрибут `#[Transition]` к методам действия, которые должны запускать определённые анимации перехода:

```php hl_lines="10 16"
<?php

use Livewire\Attributes\Transition;
use Livewire\Component;

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

```html
<div>
    <div wire:transition="content">
        Шаг {{ $step }}
    </div>

    <button wire:click="previous">Назад</button>
    <button wire:click="next">Вперёд</button>
</div>
```

Тип перехода можно указать в CSS, используя селектор `:active-view-transition-type()`:

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

## Отключение переходов

Используйте `skip: true` для отключения переходов для определённых действий:

```php
<?php

#[Transition(skip: true)]
public function reset()
{
    $this->step = 1;
}
```

Это полезно для таких действий, как «сброс» или «отмена», которые должны обновляться мгновенно, без анимации.

## Параметры

| Параметр | Тип | Описание |
|-----------|------|-------------|
| `type` | `string` | Имя типа перехода (например, `'forward'`, `'backward'`) |
| `skip` | `bool` | Установите значение `true`, чтобы отключить переходы для этого действия |

## Альтернативные подходы

### Использование transition()

Для динамических типов переходов, зависящих от логики времени выполнения, используйте вместо этого метод `transition()`:

```php
<?php

public function goToStep($step)
{
    $this->transition(type: $step > $this->step ? 'forward' : 'backward');

    $this->step = $step;
}
```

### Использование skipTransition()

Вы также можете императивно отключить переходы:

```php
<?php

public function reset()
{
    $this->skipTransition();

    $this->step = 1;
}
```

## Узнать больше

Для получения дополнительной информации о переходах представления см. [документацию по `wire:transition`](/html-directives/wire-transition).
