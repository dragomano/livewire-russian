# Атрибут `Validate`

Атрибут `#[Validate]` связывает правила валидации со свойствами компонента, обеспечивая автоматическую валидацию в реальном времени и чистое объявление правил.

## Базовое использование

Примените атрибут `#[Validate]` к свойствам, которые нуждаются в валидации:

```php hl_lines="8 11" title="resources/views/components/post/⚡create.blade.php"
<?php

use Livewire\Attributes\Validate;
use Livewire\Component;
use App\Models\Post;

new class extends Component {
    #[Validate('required|min:3')]
    public $title = '';

    #[Validate('required|min:3')]
    public $content = '';

    public function save()
    {
        $this->validate();

        Post::create([
            'title' => $this->title,
            'content' => $this->content,
        ]);

        return redirect('/posts');
    }
};
?>

<div>
    <input type="text" wire:model="title">
    @error('title') <span class="error">{{ $message }}</span> @enderror

    <textarea wire:model="content"></textarea>
    @error('content') <span class="error">{{ $message }}</span> @enderror

    <button wire:click="save">Сохранить пост</button>
</div>
```

С `#[Validate]` Livewire автоматически проверяет свойства при каждом обновлении, предоставляя пользователям мгновенную обратную связь.

## Как это работает

Когда вы добавляете `#[Validate]` к свойству:

1. **Автоматическая валидация** — Свойство проверяется каждый раз при обновлении
2. **Обратная связь в реальном времени** — Пользователи мгновенно видят ошибки валидации
3. **Ручная валидация** — Перед сохранением вы всё равно вызываете `$this->validate()`, чтобы убедиться, что все свойства проверены

## Валидация в реальном времени

По умолчанию `#[Validate]` проверяет свойства по мере их обновления:

```php title="resources/views/components/⚡registration.blade.php"
<?php

use Livewire\Attributes\Validate;
use Livewire\Component;

new class extends Component {
    #[Validate('required|email|unique:users,email')]
    public $email = '';

    #[Validate('required|min:8')]
    public $password = '';
};
?>

<div>
    <input type="email" wire:model.live.blur="email">
    @error('email') <span>{{ $message }}</span> @enderror

    <input type="password" wire:model.live.blur="password">
    @error('password') <span>{{ $message }}</span> @enderror
</div>
```

По мере заполнения формы пользователями они получают мгновенную обратную связь о валидации.

## Отключение автопроверки

Чтобы проверка выполнялась только при явном вызове `$this->validate()`, используйте `onUpdate: false`:

```php hl_lines="10 103" title="resources/views/components/post/⚡create.blade.php"
<?php

use Livewire\Attributes\Validate;
use Livewire\Component;
use App\Models\Post;

new class extends Component {
    #[Validate('required|min:3', onUpdate: false)]
    public $title = '';

    #[Validate('required|min:3', onUpdate: false)]
    public $content = '';

    public function save()
    {
        $validated = $this->validate();
        Post::create($validated);
        return redirect('/posts');
    }
};
```

Теперь проверка выполняется только при вызове `save()`, а не при каждом обновлении свойства.

## Пользовательские имена атрибутов

Настройте имя поля в сообщениях валидации:

```php hl_lines="1"
<?php

#[Validate('required', as: 'дата рождения')]
public $dob;
```

Сообщение об ошибке будет «Поле дата рождения обязательно» вместо «Поле dob обязательно».

## Пользовательские сообщения валидации

Переопределите сообщения валидации по умолчанию:

```php hl_lines="1"
<?php

#[Validate('required', message: 'Пожалуйста, укажите заголовок поста')]
public $title;
```

Для нескольких правил используйте несколько атрибутов:

```php
<?php

#[Validate('required', message: 'Пожалуйста, укажите заголовок поста')]
#[Validate('min:3', message: 'Этот заголовок слишком короткий')]
public $title;
```

## Валидация массивов

Проверяйте свойства-массивы и их элементы:

```php title="resources/views/components/⚡task-list.blade.php"
<?php

use Livewire\Attributes\Validate;
use Livewire\Component;

new class extends Component {
    #[Validate([
        'tasks' => 'required|array|min:1',
        'tasks.*' => 'required|string|min:3',
    ])]
    public $tasks = [];

    public function addTask()
    {
        $this->tasks[] = '';
    }
};
```

Это проверяет как сам массив, так и каждую отдельную задачу.

## Ограничения

!!! warning "Объекты правил не поддерживаются"
    PHP-атрибуты не могут напрямую использовать объекты `Rule` Laravel. Для сложных правил, таких как `Rule::exists()`, вместо этого используйте метод `rules()`:

    ```php
    protected function rules()
    {
        return [
            'email' => ['required', 'email', Rule::unique('users')->ignore($this->userId)],
        ];
    }
    ```

## Когда использовать

Используйте `#[Validate]`, когда:

* Создаете формы с обратной связью валидации в реальном времени
* Размещаете правила валидации рядом с определениями свойств
* Создаете простую, читаемую логику валидации
* Реализуете встроенную валидацию для улучшения UX

Используйте метод `rules()`, когда:

* Вам нужны объекты `Rule` Laravel
* Правила зависят от динамических значений
* Вы работаете со сложной условной валидацией
* Вы предпочитаете централизованное определение правил

## Пример: Контактная форма

Вот полная контактная форма с валидацией:

```php title="resources/views/pages/⚡contact.blade.php"
<?php

use Livewire\Attributes\Validate;
use Livewire\Component;
use App\Mail\ContactMessage;
use Illuminate\Support\Facades\Mail;

new class extends Component {
    #[Validate('required|min:2', as: 'имя')]
    public $name = '';

    #[Validate('required|email')]
    public $email = '';

    #[Validate('required')]
    public $subject = '';

    #[Validate('required|min:10', as: 'сообщение')]
    public $message = '';

    public function submit()
    {
        $validated = $this->validate();

        Mail::to('support@example.com')->send(new ContactMessage($validated));

        session()->flash('success', 'Сообщение успешно отправлено!');

        $this->reset();
    }
};
?>

<div>
    @if (session('success'))
        <div class="alert">{{ session('success') }}</div>
    @endif

    <form wire:submit="submit">
        <div>
            <input type="text" wire:model.live.blur="name" placeholder="Ваше имя">
            @error('name') <span class="error">{{ $message }}</span> @enderror
        </div>

        <div>
            <input type="email" wire:model.live.blur="email" placeholder="Ваш email">
            @error('email') <span class="error">{{ $message }}</span> @enderror
        </div>

        <div>
            <input type="text" wire:model.live.blur="subject" placeholder="Тема">
            @error('subject') <span class="error">{{ $message }}</span> @enderror
        </div>

        <div>
            <textarea wire:model.live.blur="message" placeholder="Ваше сообщение"></textarea>
            @error('message') <span class="error">{{ $message }}</span> @enderror
        </div>

        <button type="submit">Отправить сообщение</button>
    </form>
</div>
```

Пользователи получают немедленную обратную связь при заполнении формы с понятными именами полей и полезными сообщениями об ошибках.

## Узнать больше

Для получения исчерпывающей документации по валидации, включая объектные формы, пользовательские правила и тестирование, см. [документацию по валидации](/features/validation).

## Справочник

```php
<?php

#[Validate(
    mixed $rule = null,
    ?string $attribute = null,
    ?string $as = null,
    mixed $message = null,
    bool $onUpdate = true,
    bool $translate = true,
)]
```

| Параметр | Тип | По умолчанию | Описание |
|-----------|------|---------|-------------|
| `$rule` | `mixed` | `null` | Правило(а) валидации для применения |
| `$attribute` | `?string` | `null` | Пользовательское имя атрибута для сообщений об ошибках валидации |
| `$as` | `?string` | `null` | Дружественное имя для отображения в сообщениях об ошибках валидации |
| `$message` | `mixed` | `null` | Пользовательское сообщение(я) об ошибке при сбое валидации |
| `$onUpdate` | `bool` | `true` | Выполнять ли валидацию при обновлении свойства |
| `$translate` | `bool` | `true` | Переводить ли сообщения валидации |
