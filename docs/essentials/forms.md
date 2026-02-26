# Формы

Поскольку формы являются основой большинства веб-приложений, Livewire предоставляет множество полезных утилит для их создания. От обработки простых полей ввода до сложных вещей, таких как валидация в реальном времени или загрузка файлов, — у Livewire есть простые и хорошо задокументированные инструменты, которые облегчают вам жизнь и радуют ваших пользователей.

Погружаемся.

## Отправка формы

Начнём с рассмотрения очень простой формы в компоненте `post.create`. В этой форме будут два обычных текстовых поля, кнопка отправки, а также немного кода на бэкенде для управления состоянием формы и её отправкой:

```php title="resources/views/components/post/⚡create.blade.php"
<?php

use Livewire\Component;
use App\Models\Post;

new class extends Component {
    public $title = '';

    public $content = '';

    public function save()
    {
        Post::create(
            $this->only(['title', 'content'])
        );

        session()->flash('status', 'Пост успешно обновлён.');

        return $this->redirect('/posts');
    }
};
?>

<form wire:submit="save">
    <input type="text" wire:model="title">

    <input type="text" wire:model="content">

    <button type="submit">Сохранить</button>
</form>
```

Как вы можете видеть, мы «привязываем» публичные свойства `$title` и `$content` в форме выше с помощью `wire:model`. Это одна из самых часто используемых и мощных возможностей Livewire.

Помимо привязки `$title` и `$content`, мы используем `wire:submit`, чтобы перехватывать событие `submit` при нажатии на кнопку «Сохранить» и вызывать действие `save()`. Это действие сохранит данные формы в базу данных.

После того как новая запись будет создана в базе данных, мы перенаправим пользователя на страницу со списком постов и покажем ему «флеш»-сообщение о том, что новый пост успешно создан.

### Добавление валидации

Чтобы не сохранять неполные или опасные данные, введённые пользователем, большинство форм нуждаются в какой-либо проверке ввода.

Livewire делает валидацию форм максимально простой — достаточно добавить атрибут `#[Validate]` над теми свойствами, которые нужно проверять.

Как только к свойству прикреплён атрибут `#[Validate]`, правило валидации будет применяться к значению этого свойства каждый раз, когда оно обновляется на сервере.

Давайте добавим несколько базовых правил валидации для свойств `$title` и `$content` в нашем компоненте `post.create`:

```php hl_lines="3 8 11 16" title="resources/views/components/post/⚡create.blade.php"
<?php

use Livewire\Attributes\Validate;
use Livewire\Component;
use App\Models\Post;

new class extends Component {
    #[Validate('required')]
    public $title = '';

    #[Validate('required')]
    public $content = '';

    public function save()
    {
        $this->validate();

        Post::create(
            $this->only(['title', 'content'])
        );

        return $this->redirect('/posts');
    }
};
```

Мы также изменим наш Blade-шаблон, чтобы отображать любые ошибки валидации прямо на странице.

```html hl_lines="4 9"
<form wire:submit="save">
    <input type="text" wire:model="title">
    <div>
        @error('title') <span class="error">{{ $message }}</span> @enderror
    </div>

    <input type="text" wire:model="content">
    <div>
        @error('content') <span class="error">{{ $message }}</span> @enderror
    </div>

    <button type="submit">Сохранить</button>
</form>
```

Теперь, если пользователь попытается отправить форму, не заполнив ни одно из полей, он увидит сообщения валидации, которые подскажут, какие именно поля обязательны к заполнению перед сохранением поста.

Livewire предлагает ещё множество возможностей для валидации. Подробную информацию вы найдёте на нашей [специальной странице документации по валидации](/features/validation).

### Выделение объекта формы

Если вы работаете с большой формой и предпочитаете вынести все её свойства, логику валидации и т. д. в отдельный класс, Livewire предоставляет так называемые объекты формы.

Объекты формы позволяют повторно использовать логику формы в разных компонентах и помогают поддерживать чистоту класса компонента, группируя весь код, связанный с формой, в отдельном классе.

Вы можете создать класс формы вручную или воспользоваться удобной artisan-командой:

```shell
php artisan livewire:form PostForm
```

Указанная выше команда создаст файл `app/Livewire/Forms/PostForm.php`.

Давайте перепишем компонент `post.create`, чтобы он использовал класс `PostForm`:

```php
<?php

namespace App\Livewire\Forms;

use Livewire\Attributes\Validate;
use Livewire\Form;

class PostForm extends Form
{
    #[Validate('required|min:5')]
    public $title = '';

    #[Validate('required|min:5')]
    public $content = '';
}
```

```php hl_lines="8 15" title="resources/views/components/post/⚡create.blade.php"
<?php

use App\Livewire\Forms\PostForm;
use Livewire\Component;
use App\Models\Post;

new class extends Component {
    public PostForm $form;

    public function save()
    {
        $this->validate();

        Post::create(
            $this->form->only(['title', 'content'])
        );

        return $this->redirect('/posts');
    }
};
```

```html
<form wire:submit="save">
    <input type="text" wire:model="form.title">
    <div>
        @error('form.title') <span class="error">{{ $message }}</span> @enderror
    </div>

    <input type="text" wire:model="form.content">
    <div>
        @error('form.content') <span class="error">{{ $message }}</span> @enderror
    </div>

    <button type="submit">Сохранить</button>
</form>
```

Если хотите, вы также можете вынести логику создания поста в объект формы вот так:

```php hl_lines="17-22"
<?php

namespace App\Livewire\Forms;

use Livewire\Attributes\Validate;
use App\Models\Post;
use Livewire\Form;

class PostForm extends Form
{
    #[Validate('required|min:5')]
    public $title = '';

    #[Validate('required|min:5')]
    public $content = '';

    public function store()
    {
        $this->validate();

        Post::create($this->only(['title', 'content']));
    }
}
```

Теперь вы можете вызывать `$this->form->store()` прямо из компонента:

```php hl_lines="11" title="resources/views/components/post/⚡create.blade.php"
<?php

use App\Livewire\Forms\PostForm;
use Livewire\Component;

new class extends Component {
    public PostForm $form;

    public function save()
    {
        $this->form->store();

        return $this->redirect('/posts');
    }

    // ...
};
```

Если вы хотите использовать этот объект формы как для создания, так и для обновления записи, его легко адаптировать для обработки обоих сценариев.

Вот как это будет выглядеть, если использовать тот же объект формы в компоненте `post.edit` и заполнить его начальными данными:

```php title="resources/views/components/post/⚡edit.blade.php"
<?php

use App\Livewire\Forms\PostForm;
use Livewire\Component;
use App\Models\Post;

new class extends Component {
    public PostForm $form;

    public function mount(Post $post)
    {
        $this->form->setPost($post);
    }

    public function save()
    {
        $this->form->update();

        return $this->redirect('/posts');
    }
};
```

```php
<?php

namespace App\Livewire\Forms;

use Livewire\Attributes\Validate;
use Livewire\Form;
use App\Models\Post;

class PostForm extends Form
{
    public ?Post $post;

    #[Validate('required|min:5')]
    public $title = '';

    #[Validate('required|min:5')]
    public $content = '';

    public function setPost(Post $post)
    {
        $this->post = $post;

        $this->title = $post->title;

        $this->content = $post->content;
    }

    public function store()
    {
        $this->validate();

        Post::create($this->only(['title', 'content']));
    }

    public function update()
    {
        $this->validate();

        $this->post->update(
            $this->only(['title', 'content'])
        );
    }
}
```

Как вы можете видеть, мы добавили метод `setPost()` в объект `PostForm`, чтобы при необходимости заполнять форму существующими данными, а также сохранять пост внутри объекта формы для последующего использования. Кроме того, мы добавили метод `update()` для обновления уже существующего поста.

Объекты формы не являются обязательными при работе с Livewire, но они предоставляют удобную абстракцию, позволяющую избавить ваши компоненты от повторяющегося шаблонного кода.

### Сброс полей формы

Если вы используете объект формы, после успешной отправки вы, скорее всего, захотите сбросить форму. Это можно сделать с помощью вызова метода `reset()`:

```php hl_lines="25"
<?php

namespace App\Livewire\Forms;

use Livewire\Attributes\Validate;
use App\Models\Post;
use Livewire\Form;

class PostForm extends Form
{
    #[Validate('required|min:5')]
    public $title = '';

    #[Validate('required|min:5')]
    public $content = '';

    // ...

    public function store()
    {
        $this->validate();

        Post::create($this->only(['title', 'content']));

        $this->reset();
    }
}
```

Вы также можете сбросить только определённые свойства, передав их имена в метод `reset()`:

```php
$this->reset('title');

// Или сразу несколько...

$this->reset(['title', 'content']);
```

### Извлечение полей формы

В качестве альтернативы вы можете использовать метод `pull()`, чтобы одновременно получить значения свойств формы и сбросить их за одну операцию.

```php hl_lines="24"
<?php

namespace App\Livewire\Forms;

use Livewire\Attributes\Validate;
use App\Models\Post;
use Livewire\Form;

class PostForm extends Form
{
    #[Validate('required|min:5')]
    public $title = '';

    #[Validate('required|min:5')]
    public $content = '';

    // ...

    public function store()
    {
        $this->validate();

        Post::create(
            $this->pull()
        );
    }
}
```

Вы также можете извлечь только определённые свойства, передав их имена в метод `pull()`:

```php
<?php

// Вернуть значение перед сбросом...
$this->pull('title');

// Вернуть массив ключ-значение свойств перед сбросом...
$this->pull(['title', 'content']);
```

### Использование объектов Rule

Если у вас есть более сложные сценарии валидации, где требуются объекты `Rule` из Laravel, вы можете вместо этого определить метод `rules()`, чтобы объявить правила валидации следующим образом:

```php hl_lines="22"
<?php

namespace App\Livewire\Forms;

use Illuminate\Validation\Rule;
use App\Models\Post;
use Livewire\Form;

class PostForm extends Form
{
    public ?Post $post;

    public $title = '';

    public $content = '';

    protected function rules()
    {
        return [
            'title' => [
                'required',
                Rule::unique('posts')->ignore($this->post),
            ],
            'content' => 'required|min:5',
        ];
    }

    // ...

    public function update()
    {
        $this->validate();

        $this->post->update($this->only(['title', 'content']));

        $this->reset();
    }
}
```

При использовании метода `rules()` вместо атрибута `#[Validate]`, Livewire будет запускать правила валидации **только** тогда, когда вы явно вызываете `$this->validate()`, а не при каждом обновлении свойства.

Если вы используете валидацию в реальном времени или любой другой сценарий, в котором хотите, чтобы Livewire проверял определённые поля после каждого запроса, вы можете использовать атрибут `#[Validate]` **без указания правил** вот так:

```php hl_lines="14"
<?php

namespace App\Livewire\Forms;

use Livewire\Attributes\Validate;
use Illuminate\Validation\Rule;
use App\Models\Post;
use Livewire\Form;

class PostForm extends Form
{
    public ?Post $post;

    #[Validate]
    public $title = '';

    public $content = '';

    protected function rules()
    {
        return [
            'title' => [
                'required',
                Rule::unique('posts')->ignore($this->post),
            ],
            'content' => 'required|min:5',
        ];
    }

    // ...

    public function update()
    {
        $this->validate();

        $this->post->update($this->only(['title', 'content']));

        $this->reset();
    }
}
```

Теперь, если свойство `$title` обновляется до отправки формы — например, при использовании [`wire:model.live.blur`](/html-directives/wire-model#обновление-по-событию-blur), — валидация для `$title` будет выполняться.

### Отображение индикатора загрузки

По умолчанию Livewire автоматически отключает кнопки отправки и помечает поля ввода как `readonly` во время отправки формы, предотвращая повторную отправку формы пользователем, пока обрабатывается первая.

Однако пользователям может быть сложно заметить это состояние «загрузки» без дополнительных визуальных подсказок в интерфейсе вашего приложения.

Вот пример, как добавить небольшой спиннер загрузки к кнопке «Сохранить» с помощью директивы `wire:loading`, чтобы пользователь понимал, что форма отправляется:

```html
<button type="submit">
    Save

    <div wire:loading>
        <svg>...</svg> <!-- SVG-спиннер загрузки -->
    </div>
</button>
```

В качестве альтернативы вы можете использовать Tailwind CSS вместе с автоматическим атрибутом `data-loading`, который добавляет Livewire, — это позволяет сделать разметку чище и аккуратнее:

```html
<button type="submit">
    <span class="in-data-loading:hidden">Save</span>
    <span class="not-in-data-loading:hidden">
        <svg>...</svg> <!-- SVG-спиннер загрузки -->
    </span>
</button>
```

[Подробнее о состояниях загрузки →](/features/loading-states)

## Поля с живым обновлением

По умолчанию Livewire отправляет сетевой запрос только при отправке формы (или при вызове любого другого [действия](/essentials/actions)), но не в процессе заполнения формы.

Возьмём, к примеру, компонент `post.create`. Если вы хотите, чтобы поле ввода «title» синхронизировалось со свойством `$title` на бэкенде по мере того, как пользователь печатает, можно добавить модификатор `.live` к `wire:model` вот так:

```html
<input type="text" wire:model.live="title">
```

Теперь, когда пользователь печатает в этом поле, на сервер будут отправляться сетевые запросы для обновления свойства `$title`. Это полезно, например, для реализации поиска в реальном времени, когда набор данных фильтруется прямо по мере ввода текста в поле поиска.

## Обновление полей только по событию _blur_

В большинстве случаев `wire:model.live` отлично подходит для обновления полей формы в реальном времени, однако для текстовых полей это может быть слишком ресурсоёмко по сети.

Если вместо отправки запросов при каждом нажатии клавиши вы хотите отправлять запрос только тогда, когда пользователь покидает поле ввода (то есть происходит событие «blur» — потеря фокуса, например, при нажатии Tab или клике вне поля), можно использовать модификатор `.blur`:

```html
<input type="text" wire:model.live.blur="title" >
```

Теперь класс компонента на сервере не будет обновляться до тех пор, пока пользователь не нажмёт Tab или не кликнет за пределы текстового поля.

## Валидация в реальном времени

Иногда требуется показывать ошибки валидации прямо по мере того, как пользователь заполняет форму. Так он сразу узнаёт, что что-то не так, вместо того чтобы ждать, пока вся форма будет заполнена.

Livewire обрабатывает такие случаи автоматически. При использовании `.live` или `.blur` на `wire:model` Livewire будет отправлять сетевые запросы по мере заполнения формы пользователем. Каждый такой запрос запустит соответствующие правила валидации перед обновлением свойства. Если валидация не пройдена, свойство не обновится на сервере, а пользователю будет показана соответствующая ошибка валидации:

```html
<input type="text" wire:model.live.blur="title">

<div>
    @error('title') <span class="error">{{ $message }}</span> @enderror
</div>
```

```php
<?php

#[Validate('required|min:5')]
public $title = '';
```

Теперь, если пользователь введёт всего три символа в поле «title», а затем перейдёт к следующему полю формы (например, кликнув по нему), ему сразу будет показано сообщение об ошибке валидации, указывающее, что для этого поля установлен минимальный размер в пять символов.

Подробную информацию вы найдёте на [странице документации по валидации](/features/validation).

## Автоматическое сохранение формы в реальном времени

Если вы хотите автоматически сохранять форму по мере того, как пользователь её заполняет, вместо того чтобы ждать нажатия кнопки «Отправить», это можно сделать с помощью хука `updated()` в Livewire:

```php hl_lines="23-28" title="resources/views/components/post/⚡edit.blade.php"
<?php

use Livewire\Attributes\Validate;
use Livewire\Component;
use App\Models\Post;

new class extends Component {
    public Post $post;

    #[Validate('required')]
    public $title = '';

    #[Validate('required')]
    public $content = '';

    public function mount(Post $post)
    {
        $this->post = $post;
        $this->title = $post->title;
        $this->content = $post->content;
    }

    public function updated($name, $value)
    {
        $this->post->update([
            $name => $value,
        ]);
    }
};
?>

<form wire:submit>
    <input type="text" wire:model.live.blur="title">
    <div>
        @error('title') <span class="error">{{ $message }}</span> @enderror
    </div>

    <input type="text" wire:model.live.blur="content">
    <div>
        @error('content') <span class="error">{{ $message }}</span> @enderror
    </div>
</form>
```

В приведённом выше примере, когда пользователь завершает работу с полем (нажимает Tab или кликает на следующее поле), отправляется сетевой запрос для обновления соответствующего свойства в компоненте. Сразу после того, как свойство обновлено в классе, вызывается хук `updated()` именно для имени этого свойства и его нового значения.

Мы можем использовать этот хук, чтобы обновлять в базе данных только это конкретное поле.

Кроме того, поскольку к этим свойствам прикреплены атрибуты `#[Validate]`, правила валидации будут выполняться **до** того, как свойство обновится и будет вызван хук `updated()`.

Подробнее о хуке жизненного цикла «updated» и других хуках вы можете узнать на [странице документации по хукам жизненного цикла](/essentials/lifecycle-hooks).

## Отображение индикаторов «грязного» состояния

В сценарии с сохранением в реальном времени, описанном выше, полезно показывать пользователю, когда поле ещё не сохранено в базе данных.

Например, если пользователь зашёл на страницу редактирования поста `post.edit` и начал изменять заголовок в текстовом поле, ему может быть непонятно, когда именно изменения применяются в базе данных — особенно если внизу формы нет кнопки «Сохранить».

Livewire предоставляет директиву `wire:dirty`, которая позволяет переключать видимость элементов или изменять классы, когда значение поля ввода отличается от значения на сервере:

```html
<input type="text" wire:model.live.blur="title" wire:dirty.class="border-yellow">
```

В приведённом выше примере, когда пользователь начинает вводить текст в поле, вокруг поля появится жёлтая рамка. Как только пользователь перейдёт к другому полю (нажмёт Tab или кликнет вне поля), будет отправлен сетевой запрос, рамка исчезнет — это будет сигналом пользователю, что данные сохранены и поле больше не находится в «грязном» (несохранённом) состоянии.

Если вы хотите переключать видимость целого элемента, это можно сделать, используя `wire:dirty` вместе с `wire:target`. Директива `wire:target` указывает, за каким именно свойством (данными) нужно следить на предмет «грязного» состояния. В данном случае — за полем «title»:

```html
<input type="text" wire:model="title">

<div wire:dirty wire:target="title">Не сохранено...</div>
```

## Задержка ввода

При использовании `.live` на текстовом поле ввода вы можете захотеть более тонко контролировать, как часто отправляются сетевые запросы. По умолчанию к полю применяется задержка (debounce) в 250 мс; однако вы можете настроить это значение с помощью модификатора `.debounce`:

```html
<input type="text" wire:model.live.debounce.150ms="title" >
```

Теперь, когда к полю добавлен модификатор `.debounce.150ms`, будет использоваться более короткая задержка в 150 мс при обработке обновлений ввода для этого поля. Другими словами, по мере того как пользователь печатает, сетевой запрос будет отправляться только в том случае, если пользователь прекратит ввод хотя бы на 150 миллисекунд.

## Троттлинг ввода

Как уже было сказано ранее, при использовании задержки (debounce) на поле сетевой запрос не отправляется до тех пор, пока пользователь не прекратит ввод на заданное время. Это означает, что если пользователь продолжает печатать длинное сообщение, запрос не будет отправлен, пока он не закончит.

Иногда такое поведение нежелательно, и вам хотелось бы отправлять запросы по мере ввода текста, а не ждать, когда пользователь закончит или сделает паузу.

В таких случаях вместо `.debounce` можно использовать модификатор `.throttle`, чтобы указать интервал времени, через который будут отправляться сетевые запросы:

```html
<input type="text" wire:model.live.throttle.150ms="title" >
```

В приведённом выше примере, пока пользователь непрерывно печатает в поле «title», сетевой запрос будет отправляться каждые 150 миллисекунд, пока он не закончит ввод.

## Выделение полей ввода в Blade-компоненты

Даже в небольшом компоненте, таком как наш пример `post.create`, мы в итоге дублируем множество шаблонного кода для полей формы: сообщения об ошибках валидации, метки и т. д.

Очень удобно вынести такие повторяющиеся элементы интерфейса в отдельные [Blade-компоненты](https://laravel.com/docs/blade#components), чтобы использовать их повторно по всему приложению.

Например, ниже приведён исходный Blade-шаблон компонента `post.create`. Мы выделим два текстовых поля ввода в отдельные Blade-компоненты:

```html hl_lines="2-5 7-10"
<form wire:submit="save">
    <input type="text" wire:model="title">
    <div>
        @error('title') <span class="error">{{ $message }}</span> @enderror
    </div>

    <input type="text" wire:model="content">
    <div>
        @error('content') <span class="error">{{ $message }}</span> @enderror
    </div>

    <button type="submit">Save</button>
</form>
```

Вот как будет выглядеть шаблон после того, как мы выделили повторно используемый Blade-компонент под названием `<x-input-text>`:

```html hl_lines="2 4"
<form wire:submit="save">
    <x-input-text name="title" wire:model="title" />

    <x-input-text name="content" wire:model="content" />

    <button type="submit">Save</button>
</form>
```

Далее приведён исходный код компонента `x-input-text`:

```html title="resources/views/components/input-text.blade.php"
@props(['name'])

<input type="text" name="{{ $name }}" {{ $attributes }}>

<div>
    @error($name) <span class="error">{{ $message }}</span> @enderror
</div>
```

Как вы можете видеть, мы взяли повторяющийся HTML-код и поместили его внутрь отдельного Blade-компонента.

В основном Blade-компонент содержит только извлечённый HTML из исходного компонента. Однако мы добавили две вещи:

* Директиву `@props`
* Выражение `{{ $attributes }}` на элементе `<input>`

Давайте разберём каждое из этих дополнений:

Указав `name` как «prop» с помощью `@props(['name'])`, мы говорим Blade: если на этом компоненте установлен атрибут с именем "name", возьми его значение и сделай доступным внутри компонента как переменную `$name`.

Для всех остальных атрибутов, у которых нет явного специального назначения, мы используем выражение `{{ $attributes }}`. Оно применяется для «проброса атрибутов» (attribute forwarding), то есть любые HTML-атрибуты, написанные на самом Blade-компоненте, автоматически передаются на внутренний элемент компонента.

Благодаря этому `wire:model="title"`, а также любые дополнительные атрибуты вроде `disabled`, `class="..."` или `required` по-прежнему попадают на настоящий элемент `<input>`.

### Пользовательские элементы управления формы

В предыдущем примере мы «обернули» обычный элемент `<input>` в повторно используемый Blade-компонент, который можно применять так же, как нативный HTML-элемент ввода.

Этот паттерн очень удобен, однако бывают случаи, когда вы хотите создать полностью кастомный компонент ввода с нуля (без использования нативного `<input>` под капотом), но при этом всё равно иметь возможность привязывать его значение к свойствам Livewire через `wire:model`.

Например, представьте, что вы хотите создать компонент `<x-input-counter />` — простой счётчик, реализованный на Alpine.js.

Прежде чем создавать Blade-компонент, давайте сначала посмотрим на простой, чисто Alpine-вариант «счётчика» для примера:

```html
<div x-data="{ count: 0 }">
    <button x-on:click="count--">-</button>

    <span x-text="count"></span>

    <button x-on:click="count++">+</button>
</div>
```

Как вы можете видеть, приведённый выше компонент отображает число вместе с двумя кнопками для увеличения и уменьшения этого значения.

Теперь представим, что мы хотим выделить этот компонент в Blade-компонент под названием `<x-input-counter />`, который мы будем использовать внутри другого компонента примерно так:

```html
<x-input-counter wire:model="quantity" />
```

Создание этого компонента в основном довольно простое. Мы берём HTML-код счётчика и помещаем его в шаблон Blade-компонента, например, `resources/views/components/input-counter.blade.php`.

Однако, чтобы он работал с `wire:model="quantity"` — то есть чтобы вы могли легко привязывать данные из вашего Livewire-компонента к значению `count` внутри этого Alpine-компонента, — потребуется один дополнительный шаг.

Вот исходный код компонента:

```html title="resources/view/components/input-counter.blade.php"
<div x-data="{ count: 0 }" x-modelable="count" {{ $attributes}}>
    <button x-on:click="count--">-</button>

    <span x-text="count"></span>

    <button x-on:click="count++">+</button>
</div>
```

Как вы можете видеть, единственное отличие в этом HTML — это наличие `x-modelable="count"` и `{{ $attributes }}`.

`x-modelable` — это утилита в Alpine, которая указывает Alpine сделать определённый кусок данных доступным для привязки извне. [Более подробную информацию об этой директиве можно найти в документации Alpine.](https://alpinejs.dev/directives/modelable)

`{{ $attributes }}`, как мы уже рассматривали ранее, пробрасывает все атрибуты, переданные в Blade-компонент извне. В данном случае сюда попадёт директива `wire:model`.

Благодаря `{{ $attributes }}`, когда HTML будет отрендерен в браузере, `wire:model="quantity"` окажется рядом с `x-modelable="count"` на корневом элементе `<div>` Alpine-компонента примерно вот так:

```html
<div x-data="{ count: 0 }" x-modelable="count" wire:model="quantity">
```

`x-modelable="count"` указывает Alpine искать любые директивы `x-model` или `wire:model` и использовать данные под именем `count` для привязки к ним.

Поскольку `x-modelable` работает как с `wire:model`, так и с `x-model`, этот Blade-компонент можно использовать взаимозаменяемо как в контексте Livewire, так и в чистом Alpine. Вот пример использования этого Blade-компонента в чисто Alpine-контексте:

```html
<x-input-counter x-model="quantity" />
```

Создание пользовательских элементов ввода в вашем приложении — чрезвычайно мощный инструмент, но он требует более глубокого понимания утилит, которые предоставляют Livewire и Alpine, а также того, как они взаимодействуют друг с другом.

## Смотрите также

- **[Валидация](/features/validation)** — Проверяйте поля формы с обратной связью в реальном времени
- **[wire:model](/html-directives/wire-model)** — Привязывайте поля формы к свойствам компонента
- **[Загрузка файлов](/features/uploads)** — Обрабатывайте загрузку файлов в формах
- **[Действия](/essentials/actions)** — Обрабатывайте отправку форм с помощью действий
- **[Состояния загрузки](/features/loading-states)** — Показывайте индикаторы загрузки во время отправки формы
