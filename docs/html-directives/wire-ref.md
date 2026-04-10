# Директива `wire:ref`

Ссылки (refs) в Livewire предоставляют способ именовать, а затем находить отдельный элемент или компонент внутри Livewire.

Они полезны для отправки событий или потоковой передачи контента определённому элементу.

Они являются аккуратной альтернативой, хотя концептуально похожи, использованию классов или идентификаторов для выбора целевых элементов.

Вот список вариантов использования:

- Отправка события определённому компоненту
- Обращение к элементу с помощью `$refs`
- Потоковая передача контента определённому элементу

Давайте рассмотрим каждый из них.

## Отправка событий

Ссылки — отличный способ обратиться к определённым дочерним компонентам в системе событий Livewire.

Рассмотрим следующий модальный компонент Livewire, который прослушивает событие _close_:

```php
<?php

new class extends Livewire\Component {
    public bool $isOpen = false;

    // ...

    #[On('close')]
    public function close()
    {
        $this->isOpen = false;
    }
};
?>

<div wire:show="isOpen">
    {{ $slot }}
</div>
```

Добавив `wire:ref` к тегу компонента, вы теперь можете отправить событие _close_ напрямую к нему, используя параметр `ref:`:

```php
<?php

new class extends Livewire\Component {
    public function save()
    {
        //

        $this->dispatch('close')->to(ref: 'modal');
    }
};
?>

<div>
    <!-- ... -->

    <livewire:modal wire:ref="modal">
        <!-- ... -->

        <button wire:click="save">Сохранить</button>
    </livewire:modal>
</div>
```

## Обращение к элементам DOM

Когда вы добавляете `wire:ref` к HTML-элементу, вы можете получить к нему доступ через магическое свойство `$refs`.

Рассмотрим счётчик символов, который обновляется в реальном времени:

```html
<div>
    <textarea wire:model="message" wire:ref="message"></textarea>

    Символов: <span wire:ref="count">0</span>

    <!-- ... -->
</div>

<script>
    this.$refs.message.addEventListener('input', (e) => {
        this.$refs.count.textContent = e.target.value.length
    })
</script>
```

## Обращение к `$wire`

Если вы хотите получить доступ к `$wire` для компонента со ссылкой, вы можете сделать это через свойство `.$wire` элемента:

```html
<div>
    <!-- ... -->

    <livewire:modal wire:ref="modal">
        <!-- ... -->

        <button wire:click="save()">Сохранить</button>
    </livewire:modal>
</div>

<script>
    this.$intercept('save', ({ onFinish }) => {
        onFinish(() => {
            this.$refs.modal.$wire.close()
        })
    })
</script>
```

## Потоковая передача контента

Livewire поддерживает потоковую передачу контента напрямую к элементам внутри компонента с помощью CSS-селекторов, однако `wire:ref` является более удобным и обнаруживаемым подходом.

Рассмотрим следующий компонент, который передаёт ответ напрямую от LLM по мере его генерации:

```php
<?php

new class extends Livewire\Component {
    public $question = '';

    public function ask()
    {
        Ai::ask($this->question, function ($chunk) {
            $this->stream($chunk)->to(ref: 'answer');
        });

        $this->reset('question');
    }
};
?>

<div>
    <input type="text" wire:model="question">

    <button wire:click="ask"></button>

    <h2>Ответ:</h2>

    <p wire:ref="answer"></p>
</div>
```

## Динамические ссылки

Ссылки отлично работают в циклах и других динамических контекстах.

Вот пример с несколькими экземплярами модальных окон:

```html
@foreach($users as $index => $user)
    <livewire:modal
        wire:key="{{ $user->id }}"
        wire:ref="{{ 'user-modal-' . $user->id }}"
    >
        <!-- ... -->
    </livewire>
@endforeach
```

## Область действия

Ссылки ограничены текущим компонентом. Это означает, что вы можете обращаться к любому элементу внутри компонента, но не к элементам в других компонентах на странице.

Если несколько элементов имеют одинаковое имя ссылки внутри компонента, будет использован первый встреченный.

## Справочник

```php
wire:ref="имя"
```

Эта директива не имеет модификаторов.
