# Валидация

Цель Livewire — сделать валидацию пользовательского ввода и предоставление обратной связи максимально приятными. Основываясь на возможностях валидации Laravel, Livewire использует ваши существующие знания, одновременно предоставляя дополнительные мощные функции, такие как валидация в реальном времени.

Вот пример компонента `CreatePost`, который демонстрирует базовый процесс валидации в Livewire:

```php hl_lines="16-19"
<?php

namespace App\Livewire;

use Livewire\Component;
use App\Models\Post;

class CreatePost extends Component
{
    public $title = '';

    public $content = '';

    public function save()
    {
        $validated = $this->validate([
            'title' => 'required|min:3',
            'content' => 'required|min:3',
        ]);

        Post::create($validated);

        return redirect()->to('/posts');
    }

    public function render()
    {
        return view('livewire.create-post');
    }
}
```

```html
<form wire:submit="save">
    <input type="text" wire:model="title">
    <div>@error('title') {{ $message }} @enderror</div>

    <textarea wire:model="content"></textarea>
    <div>@error('content') {{ $message }} @enderror</div>

    <button type="submit">Сохранить</button>
</form>
```

Как вы видите, Livewire предоставляет метод `validate()`, который вы можете вызвать для валидации свойств вашего компонента. Он возвращает набор проверенных данных, которые вы затем можете безопасно вставить в базу данных.

На фронтенде вы можете использовать существующие Blade-директивы Laravel для отображения сообщений об ошибках пользователям.

Для получения дополнительной информации см. [документацию Laravel по отображению ошибок валидации в Blade](https://laravel.com/docs/blade#validation-errors).

## Атрибуты валидации

Если вы предпочитаете располагать правила валидации вашего компонента непосредственно рядом со свойствами, вы можете использовать атрибут Livewire `#[Validate]`.

Связывая правила валидации со свойствами с помощью `#[Validate]`, Livewire будет автоматически запускать правила валидации свойств перед каждым обновлением. Тем не менее, вам всё равно следует запускать `$this->validate()` перед сохранением данных в базу данных, чтобы свойства, которые не были обновлены, также были проверены.

```php hl_lines="9 12"
<?php

use Livewire\Attributes\Validate;
use Livewire\Component;
use App\Models\Post;

class CreatePost extends Component
{
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

        return redirect()->to('/posts');
    }

    // ...
}
```

!!! info "Атрибуты Validate не поддерживают объекты Rule"
    PHP-атрибуты ограничены определенным синтаксисом, таким как простые строки и массивы. Если вы хотите использовать динамический синтаксис, такой как объекты Laravel Rule (`Rule::exists(...)`), вам следует вместо этого [определить метод `rules()`](#определение-метода-rules) в вашем компоненте.

    Узнайте больше в документации по [использованию объектов Laravel Rule с Livewire](#использование-объектов-правил-laravel).

Если вы предпочитаете больше контроля над тем, когда проверяются свойства, вы можете передать параметр `onUpdate: false` атрибуту `#[Validate]`. Это отключит любую автоматическую валидацию и вместо этого предположит, что вы хотите вручную проверять свойства с помощью метода `$this->validate()`:

```php
<?php

use Livewire\Attributes\Validate;
use Livewire\Component;
use App\Models\Post;

class CreatePost extends Component
{
    #[Validate('required|min:3', onUpdate: false)]
    public $title = '';

    #[Validate('required|min:3', onUpdate: false)]
    public $content = '';

    public function save()
    {
        $validated = $this->validate();

        Post::create($validated);

        return redirect()->to('/posts');
    }

    // ...
}
```

### Кастомное имя атрибута

Если вы хотите изменить имя атрибута, подставляемое в сообщение валидации, вы можете сделать это с помощью параметра `as: `:

```php
<?php

use Livewire\Attributes\Validate;

#[Validate('required', as: 'дата рождения')]
public $dob;
```

Когда валидация в приведенном выше фрагменте не удастся, Laravel будет использовать «дата рождения» вместо «dob» в качестве имени поля в сообщении об ошибке. Сгенерированное сообщение будет «Поле дата рождения обязательно для заполнения» (или аналогичное, в зависимости от настроек локализации Laravel) вместо «Поле dob обязательно для заполнения».

### Кастомное сообщение валидации

Чтобы обойти стандартное сообщение валидации Laravel и заменить его собственным, вы можете использовать параметр `message: ` в атрибуте `#[Validate]`:

```php
<?php

use Livewire\Attributes\Validate;

#[Validate('required', message: 'Пожалуйста, укажите заголовок поста')]
public $title;
```

Теперь, когда валидация этого свойства не удастся, сообщение будет «Пожалуйста, укажите заголовок поста» вместо «Поле title обязательно для заполнения».

Если вы хотите добавить разные сообщения для разных правил, вы можете просто указать несколько атрибутов `#[Validate]`:

```php
<?php

#[Validate('required', message: 'Пожалуйста, укажите заголовок поста')]
#[Validate('min:3', message: 'Этот заголовок слишком короткий')]
public $title;
```

### Отказ от локализации

По умолчанию сообщения правил и атрибуты Livewire локализуются с помощью помощника перевода Laravel: `trans()`.

Вы можете отказаться от локализации, передав параметр `translate: false` атрибуту `#[Validate]`:

```php
<?php

#[Validate('required', message: 'Please provide a post title', translate: false)]
public $title;
```

### Кастомный ключ

При применении правил валидации непосредственно к свойству с помощью атрибута `#[Validate]`, Livewire предполагает, что ключом валидации должно быть имя самого свойства. Однако бывают случаи, когда вы можете захотеть настроить ключ валидации.

Например, вы можете захотеть предоставить отдельные правила валидации для свойства-массива и его дочерних элементов. В этом случае вместо передачи правила валидации в качестве первого аргумента атрибуту `#[Validate]`, вы можете передать массив пар ключ-значение:

```php
<?php

#[Validate([
    'todos' => 'required',
    'todos.*' => [
        'required',
        'min:3',
        new Uppercase,
    ],
])]
public $todos = [];
```

Теперь, когда пользователь обновляет `$todos` или вызывается метод `validate()`, будут применены оба этих правила валидации.

## Объекты форм

По мере добавления свойств и правил валидации в компонент Livewire, он может стать слишком перегруженным. Чтобы облегчить эту проблему, а также предоставить удобную абстракцию для повторного использования кода, вы можете использовать *Объекты форм* Livewire для хранения ваших свойств и правил валидации.

Ниже приведен тот же пример `CreatePost`, но теперь свойства и правила вынесены в отдельный объект формы под названием `PostForm`:

```php
<?php

namespace App\Livewire\Forms;

use Livewire\Attributes\Validate;
use Livewire\Form;

class PostForm extends Form
{
    #[Validate('required|min:3')]
    public $title = '';

    #[Validate('required|min:3')]
    public $content = '';
}
```

Теперь `PostForm` можно определить как свойство в компоненте `CreatePost`:

```php
<?php

namespace App\Livewire;

use App\Livewire\Forms\PostForm;
use Livewire\Component;
use App\Models\Post;

class CreatePost extends Component
{
    public PostForm $form;

    public function save()
    {
        Post::create(
            $this->form->all()
        );

        return redirect()->to('/posts');
    }

    // ...
}
```

Как вы видите, вместо того чтобы перечислять каждое свойство по отдельности, мы можем получить значения всех свойств с помощью метода `->all()` объекта формы.

Кроме того, при обращении к именам свойств в шаблоне вы должны добавлять префикс `form.` к каждому экземпляру:

```html
<form wire:submit="save">
    <input type="text" wire:model="form.title">
    <div>@error('form.title') {{ $message }} @enderror</div>

    <textarea wire:model="form.content"></textarea>
    <div>@error('form.content') {{ $message }} @enderror</div>

    <button type="submit">Сохранить</button>
</form>
```

При использовании объектов форм валидация атрибутов `#[Validate]` будет запускаться при каждом обновлении свойства. Однако, если вы отключите это поведение, указав `onUpdate: false` в атрибуте, вы можете вручную запустить валидацию объекта формы с помощью `$this->form->validate()`:

```php
<?php

public function save()
{
    Post::create(
        $this->form->validate()
    );

    return redirect()->to('/posts');
}
```

Объекты форм — это полезная абстракция для большинства крупных наборов данных и множества дополнительных функций, которые делают их еще более мощными. Для получения дополнительной информации ознакомьтесь с подробной [документацией по объектам форм](/essentials/forms#выделение-объекта-формы).

## Валидация в реальном времени

Валидация в реальном времени — это термин, используемый для случаев, когда вы проверяете ввод пользователя по мере заполнения формы, а не ждете отправки формы.

При использовании атрибутов `#[Validate]` непосредственно в свойствах Livewire, каждый раз, когда отправляется сетевой запрос для обновления значения свойства на сервере, будут применяться указанные правила валидации.

Это означает, что для обеспечения валидации в реальном времени для конкретного поля ввода не требуется никакой дополнительной работы на бэкенде. Единственное, что требуется, — это использование `wire:model.live` или `wire:model.live.blur`, чтобы указать Livewire инициировать сетевые запросы по мере заполнения полей.

В приведенном ниже примере к текстовому вводу было добавлено `wire:model.live.blur`. Теперь, когда пользователь печатает в поле, а затем переключается (tab) или кликает в другом месте, будет инициирован сетевой запрос с обновленным значением и запустятся правила валидации:

```html
<form wire:submit="save">
    <input type="text" wire:model.live.blur="title">

    <!-- -->
</form>
```

Если вы используете метод `rules()` для объявления правил валидации свойства вместо атрибута `#[Validate]`, вы все равно можете добавить атрибут `#[Validate]` без параметров, чтобы сохранить поведение валидации в реальном времени:

```php hl_lines="9"
<?php

use Livewire\Attributes\Validate;
use Livewire\Component;
use App\Models\Post;

class CreatePost extends Component
{
    #[Validate]
    public $title = '';

    public $content = '';

    protected function rules()
    {
        return [
            'title' => 'required|min:5',
            'content' => 'required|min:5',
        ];
    }

    public function save()
    {
        $validated = $this->validate();

        Post::create($validated);

        return redirect()->to('/posts');
    }
}
```

В приведённом выше примере, несмотря на то что атрибут `#[Validate]` пуст, он укажет Livewire запускать валидацию полей, указанную в `rules()`, при каждом обновлении свойства.

## Настройка сообщений об ошибках

Из коробки Laravel предоставляет понятные сообщения о валидации, такие как «The title field is required.», если к свойству `$title` привязано правило `required`.

Однако вам может потребоваться настроить язык этих сообщений об ошибках, чтобы они лучше соответствовали вашему приложению и его пользователям.

### Кастомные имена атрибутов

Иногда свойство, которое вы проверяете, имеет имя, которое не подходит для отображения пользователям. Например, если в вашем приложении есть поле базы данных с именем `dob`, которое означает «Дата рождения», вы захотите показать пользователям «Поле дата рождения обязательно для заполнения» вместо «Поле dob обязательно для заполнения».

Livewire позволяет указать альтернативное имя для свойства с помощью параметра `as: `:

```php
<?php

use Livewire\Attributes\Validate;

#[Validate('required', as: 'дата рождения')]
public $dob = '';
```

Теперь, если правило валидации `required` не сработает, в сообщении об ошибке будет указано «Поле дата рождения обязательно для заполнения.» вместо «Поле dob обязательно для заполнения.».

### Кастомные сообщения

Если настройки имени свойства недостаточно, вы можете настроить всё сообщение валидации целиком с помощью параметра `message: `:

```php
<?php

use Livewire\Attributes\Validate;

#[Validate('required', message: 'Пожалуйста, укажите дату вашего рождения.')]
public $dob = '';
```

Если у вас есть несколько правил, для которых нужно настроить сообщения, рекомендуется использовать для каждого из них отдельные атрибуты `#[Validate]`, например так:

```php
<?php

use Livewire\Attributes\Validate;

#[Validate('required', message: 'Пожалуйста, введите заголовок.')]
#[Validate('min:5', message: 'Ваш заголовок слишком короткий.')]
public $title = '';
```

Если вы хотите использовать синтаксис массива в атрибуте `#[Validate]`, вы можете указать кастомные атрибуты и сообщения следующим образом:

```php
<?php

use Livewire\Attributes\Validate;

#[Validate([
    'titles' => 'required',
    'titles.*' => 'required|min:5',
], message: [
    'required' => 'Поле :attribute отсутствует.',
    'titles.required' => 'Поля :attribute отсутствуют.',
    'min' => 'Поле :attribute слишком короткое.',
], attribute: [
    'titles.*' => 'заголовок',
])]
public $titles = [];
```

## Определение метода `rules()`

В качестве альтернативы атрибутам `#[Validate]` в Livewire вы можете определить в компоненте метод `rules()` и вернуть в нём список полей и соответствующих правил валидации. Это удобно, когда вы хотите использовать синтаксисы, которые не поддерживаются в атрибутах PHP, например, объекты правил Laravel типа `Rule::password()`.

Эти правила будут автоматически применяться, когда вы вызываете `$this->validate()` внутри компонента. Также вы можете определить методы `messages()` и `validationAttributes()`.

Пример:

```php hl_lines="13-19 21-27 29-34"
<?php

use Livewire\Component;
use App\Models\Post;
use Illuminate\Validation\Rule;

class CreatePost extends Component
{
    public $title = '';

    public $content = '';

    protected function rules()
    {
        return [
            'title' => Rule::exists('posts', 'title'),
            'content' => 'required|min:3',
        ];
    }

    protected function messages()
    {
        return [
            'content.required' => 'Поле :attribute отсутствует.',
            'content.min' => 'Поле :attribute слишком короткое.',
        ];
    }

    protected function validationAttributes()
    {
        return [
            'content' => 'description',
        ];
    }

    public function save()
    {
        $this->validate();

        Post::create([
            'title' => $this->title,
            'content' => $this->content,
        ]);

        return redirect()->to('/posts');
    }

    // ...
}
```

!!! warning "Метод `rules()` не выполняет валидацию при обновлении данных"
    При определении правил через метод `rules()` Livewire будет использовать эти правила валидации **только** тогда, когда вы явно вызываете `$this->validate()`. Это отличается от стандартных атрибутов `#[Validate]`, которые применяются каждый раз при обновлении поля, например, через `wire:model`. Чтобы правила валидации применялись к свойству при каждом его обновлении, вы всё ещё можете использовать `#[Validate]` без дополнительных параметров.

!!! warning "Не конфликтуйте с механизмами Livewire"
    При использовании средств валидации Livewire в вашем компоненте **не должно** быть свойств или методов с именами `rules`, `messages`, `validationAttributes` или `validationCustomValues`, если вы не настраиваете процесс валидации специально. В противном случае возникнет конфликт с внутренними механизмами Livewire.

## Использование объектов правил Laravel

Объекты правил `Rule` в Laravel — это чрезвычайно мощный способ добавить продвинутое поведение валидации в ваши формы.

Вот пример использования объектов `Rule` вместе с методом `rules()` в Livewire для реализации более сложной валидации:

```php hl_lines="22 36"
<?php

namespace App\Livewire;

use Illuminate\Validation\Rule;
use App\Models\Post;
use Livewire\Form;

class UpdatePost extends Form
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

    public function mount()
    {
        $this->title = $this->post->title;
        $this->content = $this->post->content;
    }

    public function update()
    {
        $this->validate();

        $this->post->update($this->all());

        $this->reset();
    }

    // ...
}
```

## Ручное управление ошибками валидации

Утилиты валидации Livewire покрывают большинство распространённых сценариев валидации, однако бывают случаи, когда вам требуется полный контроль над сообщениями об ошибках валидации прямо внутри компонента.

Ниже приведены все доступные методы для работы с ошибками валидации в компоненте Livewire:

Метод                              | Описание
-----------------------------------|---------------------------------------------------------------
`$this->addError([key], [message])` | Вручную добавить сообщение об ошибке валидации в «мешок ошибок»
`$this->resetValidation([?key])`   | Сбросить ошибки валидации для указанного ключа. Если ключ не передан — сбросить все ошибки
`$this->getErrorBag()`             | Получить объект «мешка ошибок» Laravel, который используется внутри компонента Livewire

!!! info "Использование `$this->addError()` с объектами форм"
    При ручном добавлении ошибок с помощью `$this->addError` внутри объекта формы ключ автоматически будет дополнен префиксом — именем свойства, к которому привязана форма в родительском компоненте. Например, если в компоненте форма присвоена свойству `$data`, то ключ превратится в `data.ключ`.

## Доступ к экземпляру валидатора

Иногда требуется получить доступ к экземпляру валидатора (`Validator`), который Livewire использует внутри метода `validate()`. Для этого предназначен метод `withValidator`. Замыкание, которое вы передаёте, получает уже полностью сконструированный объект валидатора в качестве аргумента. Это позволяет вызвать любые его методы **до** того, как правила валидации будут фактически выполнены.

Ниже приведён пример перехвата внутреннего валидатора Livewire, чтобы вручную проверить условие и добавить дополнительное сообщение об ошибке:

```php
<?php

use Livewire\Attributes\Validate;
use Livewire\Component;
use App\Models\Post;

class CreatePost extends Component
{
    #[Validate('required|min:3')]
    public $title = '';

    #[Validate('required|min:3')]
    public $content = '';

    public function boot()
    {
        $this->withValidator(function ($validator) {
            $validator->after(function ($validator) {
                if (str($this->title)->startsWith('"')) {
                    $validator->errors()->add('title', 'Заголовки не могут начинаться с кавычек');
                }
            });
        });
    }

    public function save()
    {
        Post::create($this->all());

        return redirect()->to('/posts');
    }

    // ...
}
```

## Использование собственных валидаторов

Если вы хотите использовать в Livewire свою собственную систему валидации — это не проблема. Livewire перехватит любое исключение `ValidationException`, выброшенное внутри компонента, и передаст ошибки в представление точно так же, как если бы вы использовали собственный метод `validate()` Livewire.

Ниже приведён пример компонента `CreatePost`, в котором вместо встроенных средств валидации Livewire создаётся и применяется полностью собственный валидатор к свойствам компонента:

```php
<?php

use Illuminate\Support\Facades\Validator;
use Livewire\Component;
use App\Models\Post;

class CreatePost extends Component
{
    public $title = '';

    public $content = '';

    public function save()
    {
        $validated = Validator::make(
            // Данные для валидации...
            ['title' => $this->title, 'content' => $this->content],

            // Правила валидации, которые нужно применить...
            ['title' => 'required|min:3', 'content' => 'required|min:3'],

            // Пользовательские сообщения об ошибках валидации...
            ['required' => 'Поле :attribute обязательно для заполнения'],
            )->validate();

        Post::create($validated);

        return redirect()->to('/posts');
    }

    // ...
}
```

## Тестирование валидации

Livewire предоставляет удобные утилиты для тестирования сценариев валидации, такие как метод `assertHasErrors()`.

Ниже приведён базовый тестовый случай, который проверяет, что ошибки валидации возникают, если для свойства `title` не задано значение:

```php
<?php

namespace Tests\Feature\Livewire;

use App\Livewire\CreatePost;
use Livewire\Livewire;
use Tests\TestCase;

class CreatePostTest extends TestCase
{
    public function test_cant_create_post_without_title()
    {
        Livewire::test(CreatePost::class)
            ->set('content', 'Sample content...')
            ->call('save')
            ->assertHasErrors('title');
    }
}
```

В дополнение к проверке наличия ошибок, метод `assertHasErrors` позволяет также уточнить утверждение до конкретных правил, передавая правила, против которых нужно выполнить проверку, в качестве второго аргумента метода:

```php
<?php

public function test_cant_create_post_with_title_shorter_than_3_characters()
{
    Livewire::test(CreatePost::class)
        ->set('title', 'Sa')
        ->set('content', 'Sample content...')
        ->call('save')
        ->assertHasErrors(['title' => ['min:3']]);
}
```

Вы также можете проверить наличие ошибок валидации сразу для нескольких свойств:

```php
<?php

public function test_cant_create_post_without_title_and_content()
{
    Livewire::test(CreatePost::class)
        ->call('save')
        ->assertHasErrors(['title', 'content']);
}
```

Для получения дополнительной информации о других утилитах тестирования, предоставляемых Livewire, ознакомьтесь с [документацией по тестированию](/essentials/testing).

## Доступ к ошибкам в JavaScript

Livewire предоставляет магическое свойство `$errors` для доступа к ошибкам валидации на стороне клиента:

```html
<form wire:submit="save">
    <input type="email" wire:model="email">

    <div wire:show="$errors.has('email')">
        <span wire:text="$errors.first('email')"></span>
    </div>

    <button type="submit">Сохранить</button>
</form>
```

### Доступные методы

- `$errors.has('field')` - Проверить, есть ли ошибки у поля
- `$errors.first('field')` - Получить первое сообщение об ошибке для поля
- `$errors.get('field')` - Получить все сообщения об ошибках для поля
- `$errors.all()` - Получить все ошибки для всех полей
- `$errors.clear()` - Очистить все ошибки
- `$errors.clear('field')` - Очистить ошибки для конкретного поля

При использовании Alpine.js доступ к `$errors` осуществляется через `$wire.$errors`.

## Устаревший атрибут `[#Rule]`

Когда Livewire v3 только вышел, для атрибутов валидации использовался термин `Rule` вместо `Validate` (`#[Rule]`).

Из-за конфликта имён с объектами правил Laravel (`Rule objects`) это было изменено на `#[Validate]`. В Livewire v3 поддерживаются оба варианта, однако рекомендуется заменить все вхождения `#[Rule]` на `#[Validate]`, чтобы использовать актуальный синтаксис.

## Смотрите также

- **[Формы](/essentials/forms)** — Валидация полей формы с обратной связью в реальном времени
- **[Свойства](/essentials/properties)** — Валидация значений свойств перед сохранением
- **[Атрибут Validate](/php-attributes/attribute-validate)** — Использование #[Validate] для валидации свойств
- **[Действия](/essentials/actions)** — Валидация данных в методах-действиях
