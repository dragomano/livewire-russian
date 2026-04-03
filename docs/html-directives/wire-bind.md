# Директива `wire:bind`

`wire:bind` — это директива, которая динамически привязывает HTML-атрибуты к свойствам компонента или выражениям. В отличие от использования синтаксиса атрибутов Blade, `wire:bind` обновляет атрибут реактивно на клиенте без необходимости полного перерендеринга.

Если вы знакомы с директивой `x-bind` из Alpine, то эти две директивы по сути одинаковы.

## Базовое использование

```html
<input wire:model="message" wire:bind:class="message.length > 240 && 'text-red-500'">
```

По мере ввода пользователем текста `wire:bind:class` реагирует на длину сообщения и применяет класс мгновенно на клиенте.

## Распространённые варианты использования

### Привязка стилей

```html
<div wire:bind:style="{ 'color': textColor, 'font-size': fontSize + 'px' }">
    Стилизованный текст
</div>
```

### Привязка href

```html
<a wire:bind:href="url">Динамическая ссылка</a>
```

### Привязка состояния disabled

```html
<button wire:bind:disabled="isArchived">Удалить</button>
```

### Привязка data-атрибутов

```html
<div wire:bind:data-count="count">...</div>
```

## Справочник

```php
wire:bind:{attribute}="expression"
```

Замените `{attribute}` на любое допустимое имя HTML-атрибута (например, `class`, `style`, `href`, `disabled`, `data-*`).

Эта директива не имеет модификаторов.
