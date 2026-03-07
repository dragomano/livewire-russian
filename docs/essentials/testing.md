# Тестирование

Компоненты Livewire легко тестировать. Поскольку под капотом это обычные классы Laravel, их можно тестировать с помощью уже существующих инструментов тестирования Laravel. Однако Livewire предоставляет множество дополнительных утилит, которые делают тестирование компонентов невероятно удобным.

Эта документация проведёт вас через процесс тестирования компонентов Livewire с использованием **Pest** — рекомендуемого фреймворка для тестирования, хотя вы также можете использовать PHPUnit, если вам так удобнее.

## Установка Pest

[Pest](https://pestphp.com/) — это приятный PHP-фреймворк для тестирования с акцентом на простоту. Это рекомендуемый способ тестирования компонентов Livewire в Livewire 4.

Чтобы установить Pest в ваше Laravel-приложение, сначала удалите PHPUnit (если он установлен), а затем подключите Pest:

```shell
composer remove phpunit/phpunit
composer require pestphp/pest --dev --with-all-dependencies
```

Затем инициализируйте Pest в вашем проекте:

```shell
./vendor/bin/pest --init
```

Это создаст в вашем проекте файл конфигурации `tests/Pest.php`.

Более подробные инструкции по установке смотрите в [документации по установке Pest](https://pestphp.com/docs/installation).

## Настройка Pest для компонентов на основе представлений

Если вы пишете тесты рядом со своими компонентами на основе Blade-представлений (однофайловыми или многофайловыми), вам нужно настроить Pest, чтобы он распознавал эти тестовые файлы.

Сначала обновите ваш файл `tests/Pest.php`, добавив директорию `resources/views`:

```php
<?php

pest()->extend(Tests\TestCase::class)
    // ...
    ->in('Feature', '../resources/views');
```

Это указывает Pest использовать ваш базовый класс `TestCase` для тестов, находящихся как в директории `tests/Feature`, так и где угодно внутри `resources/views`.

Далее обновите ваш файл `phpunit.xml`, добавив тестовый набор (test suite) для тестов компонентов:

```xml
<testsuite name="Components">
    <directory suffix=".test.php">resources/views</directory>
</testsuite>
```

Теперь Pest будет распознавать и запускать тесты, расположенные рядом с вашими компонентами, когда вы выполняете команду `./vendor/bin/pest`.

## Создание первого теста

Вы можете сгенерировать файл теста рядом с компонентом, добавив флаг `--test` к команде `make:livewire`:

```shell
php artisan make:livewire post.create --test
```

Для многофайловых компонентов это создаст файл теста по пути `resources/views/components/post/create.test.php`:

```php
<?php

use Livewire\Livewire;

it('renders successfully', function () {
    Livewire::test('post.create')
        ->assertStatus(200);
});
```

Для классовых компонентов это создаст файл теста PHPUnit по пути `tests/Feature/Livewire/Post/CreateTest.php`. Вы можете преобразовать его в синтаксис Pest или продолжать использовать PHPUnit — оба отлично работают с Livewire.

### Тестирование того, что страница содержит компонент

Самый простой тест Livewire, который вы можете написать, — это проверка того, что заданная конечная точка включает и успешно отображает компонент Livewire.

```php
<?php

it('component exists on the page', function () {
    $this->get('/posts/create')
        ->assertSeeLivewire('post.create');
});
```

!!! tip "Smoke-тесты приносят огромную пользу"
    Такие тесты называются «smoke tests» (дымовые тесты) — они гарантируют отсутствие катастрофических проблем в вашем приложении. Хотя они простые, эти тесты дают колоссальную ценность: требуют минимального сопровождения и обеспечивают базовый уровень уверенности в том, что ваши страницы успешно отображаются.

## Тестирование в браузере

Pest v4 включает встроенную поддержку браузерного тестирования на базе Playwright. Это позволяет тестировать ваши компоненты Livewire в реальном браузере, взаимодействуя с ними точно так же, как это делает пользователь.

### Установка браузерного тестирования

Сначала установите плагин Pest для работы с браузером:

```shell
composer require pestphp/pest-plugin-browser --dev
```

Затем установите Playwright через npm:

```shell
npm install playwright@latest
npx playwright install
```

Полную документацию по браузерному тестированию смотрите в [руководстве по браузерному тестированию Pest](https://pestphp.com/docs/browser-testing).

### Написание браузерных тестов

Вместо использования `Livewire::test()` вы можете использовать `Livewire::visit()`, чтобы протестировать ваш компонент в реальном браузере:

```php
<?php

it('can create a new post', function () {
    Livewire::visit('post.create')
        ->type('[wire\:model="title"]', 'My first post')
        ->type('[wire\:model="content"]', 'This is content')
        ->press('Save')
        ->assertSee('Post created successfully');
});
```

Браузерные тесты работают медленнее, чем юнит-тесты, но дают полную уверенность в том, что ваши компоненты работают именно так, как ожидается, в реальной браузерной среде.

Полный список доступных утверждений (assertions) для браузерного тестирования смотрите в [документации по утверждениям браузерного тестирования Pest](https://pestphp.com/docs/browser-testing#content-available-assertions).

!!! info "Когда использовать браузерные тесты"
    Используйте браузерные тесты для критически важных пользовательских сценариев и сложных взаимодействий. Для большинства случаев тестирования компонентов стандартный подход с `Livewire::test()` быстрее и вполне достаточен.

## Тестирование представлений

Livewire предоставляет метод `assertSee()`, чтобы проверить наличие текста в отрендеренном выводе вашего компонента:

```php
<?php

use App\Models\Post;

it('displays posts', function () {
    Post::factory()->create(['title' => 'My first post']);
    Post::factory()->create(['title' => 'My second post']);

    Livewire::test('show-posts')
        ->assertSee('My first post')
        ->assertSee('My second post');
});
```

### Проверка данных представления

Иногда полезно протестировать данные, передаваемые в представление, вместо проверки отрендеренного вывода:

```php
<?php

use App\Models\Post;

it('passes all posts to the view', function () {
    Post::factory()->count(3)->create();

    Livewire::test('show-posts')
        ->assertViewHas('posts', function ($posts) {
            return count($posts) === 3;
        });
});
```

Для простых проверок вы можете передать ожидаемое значение напрямую:

```php
<?php

Livewire::test('show-posts')
    ->assertViewHas('postCount', 3);
```

## Тестирование с аутентификацией

Большинство приложений требуют, чтобы пользователи входили в систему. Вместо того чтобы вручную выполнять аутентификацию в начале каждого теста, используйте метод `actingAs()`:

```php
<?php

use App\Models\User;
use App\Models\Post;

it('user only sees their own posts', function () {
    $user = User::factory()
        ->has(Post::factory()->count(3))
        ->create();

    $stranger = User::factory()
        ->has(Post::factory()->count(2))
        ->create();

    Livewire::actingAs($user)
        ->test('show-posts')
        ->assertViewHas('posts', function ($posts) {
            return count($posts) === 3;
        });
});
```

## Тестирование свойств

Livewire предоставляет утилиты для установки и проверки свойств компонента.

Используйте `set()` для обновления свойств и `assertSet()` для проверки их значений:

```php
<?php

it('can set the title property', function () {
    Livewire::test('post.create')
        ->set('title', 'My Amazing Post')
        ->assertSet('title', 'My Amazing Post');
});
```

### Инициализация свойств

Компоненты часто получают данные от родительских компонентов или параметров маршрута. Передайте эти данные вторым параметром в метод `Livewire::test()`:

```php
<?php

use App\Models\Post;

it('title field is populated when editing', function () {
    $post = Post::factory()->create([
        'title' => 'Existing post title',
    ]);

    Livewire::test('post.edit', ['post' => $post])
        ->assertSet('title', 'Existing post title');
});
```

### Установка параметров URL

Если ваш компонент использует [функцию URL в Livewire](/features/url) для отслеживания состояния в строке запроса, используйте `withQueryParams()`, чтобы смоделировать параметры URL:

```php
<?php

use App\Models\Post;

it('can search posts via url query string', function () {
    Post::factory()->create(['title' => 'Laravel testing']);
    Post::factory()->create(['title' => 'Vue components']);

    Livewire::withQueryParams(['search' => 'Laravel'])
        ->test('search-posts')
        ->assertSee('Laravel testing')
        ->assertDontSee('Vue components');
});
```

### Установка cookies

Используйте `withCookie()` или `withCookies()`, чтобы установить cookies для ваших тестов:

```php
<?php

it('loads discount token from cookie', function () {
    Livewire::withCookies(['discountToken' => 'SUMMER2024'])
        ->test('cart')
        ->assertSet('discountToken', 'SUMMER2024');
});
```

## Вызов действий

Используйте метод `call()`, чтобы запускать действия компонента в ваших тестах:

```php
<?php

use App\Models\Post;

it('can create a post', function () {
    expect(Post::count())->toBe(0);

    Livewire::test('post.create')
        ->set('title', 'My new post')
        ->set('content', 'Post content here')
        ->call('Save');

    expect(Post::count())->toBe(1);
});
```

!!! tip "Ожидания в Pest"
    В примерах выше используются утверждения с синтаксисом `expect()` от Pest. Полный список доступных ожиданий смотрите в [документации по ожиданиям Pest](https://pestphp.com/docs/expectations).

Вы можете передавать параметры в действия:

```php
<?php

Livewire::test('post.show')
    ->call('deletePost', $postId);
```

### Тестирование валидации

Проверяйте, что ошибки валидации были выброшены, с помощью `assertHasErrors()`:

```php
<?php

it('title field is required', function () {
    Livewire::test('post.create')
        ->set('title', '')
        ->call('Save')
        ->assertHasErrors('title');
});
```

Тестирование конкретных правил валидации

```php
<?php

it('title must be at least 3 characters', function () {
    Livewire::test('post.create')
        ->set('title', 'ab')
        ->call('Save')
        ->assertHasErrors(['title' => ['min:3']]);
});
```

### Тестирование авторизации

Убедитесь, что проверки авторизации работают корректно, используя `assertUnauthorized()` и `assertForbidden()`:

```php
<?php

use App\Models\User;
use App\Models\Post;

it('cannot update another users post', function () {
    $user = User::factory()->create();
    $stranger = User::factory()->create();
    $post = Post::factory()->for($stranger)->create();

    Livewire::actingAs($user)
        ->test('post.edit', ['post' => $post])
        ->set('title', 'Hacked!')
        ->call('Save')
        ->assertForbidden();
});
```

### Тестирование перенаправлений

Проверяйте, что действие выполнило перенаправление:

```php
<?php

it('redirects to posts index after creating', function () {
    Livewire::test('post.create')
        ->set('title', 'New post')
        ->set('content', 'Content here')
        ->call('Save')
        ->assertRedirect('/posts');
});
```

Вы также можете проверять перенаправления на именованные маршруты или компоненты страниц:

```php
<?php

->assertRedirect(route('posts.index'));
->assertRedirectToRoute('posts.index');
```

### Тестирование событий

Проверяйте, что события были отправлены из вашего компонента:

```php
<?php

it('dispatches event when post is created', function () {
    Livewire::test('post.create')
        ->set('title', 'New post')
        ->call('Save')
        ->assertDispatched('post-created');
});
```

Тестирование коммуникации событий между компонентами

```php
<?php

it('updates post count when event is dispatched', function () {
    $badge = Livewire::test('post-count-badge')
        ->assertSee('0');

    Livewire::test('post.create')
        ->set('title', 'New post')
        ->call('Save')
        ->assertDispatched('post-created');

    $badge->dispatch('post-created')
        ->assertSee('1');
});
```

Проверка событий с конкретными параметрами:

```php
<?php

it('dispatches notification when deleting post', function () {
    Livewire::test('post.show')
        ->call('delete', postId: 3)
        ->assertDispatched('notify', message: 'Post deleted');
});
```

Для сложных проверок используйте замыкание:

```php
<?php

it('dispatches event with correct data', function () {
    Livewire::test('post.show')
        ->call('delete', postId: 3)
        ->assertDispatched('notify', function ($event, $params) {
            return ($params['message'] ?? '') === 'Post deleted';
        });
});
```

### Тестирование выполнения JavaScript

Проверяйте, что компонент выполнил JavaScript с помощью `$this->js()`:

```php
<?php

it('shows an alert after saving', function () {
    Livewire::test('post.create')
        ->set('title', 'New post')
        ->call('Save')
        ->assertJs("alert('Post saved!')");
});
```

Вы также можете проверить, что JavaScript не выполнялся:

```php
<?php

->assertNoJs();
```

## Использование PHPUnit

Хотя Pest рекомендуется, вы можете использовать и PHPUnit для тестирования компонентов Livewire. Все те же утилиты тестирования работают с синтаксисом PHPUnit.

Вот пример на PHPUnit для сравнения:

```php
<?php

namespace Tests\Feature\Livewire;

use Livewire\Livewire;
use App\Models\Post;
use Tests\TestCase;

class CreatePostTest extends TestCase
{
    public function test_can_create_post()
    {
        $this->assertEquals(0, Post::count());

        Livewire::test('post.create')
            ->set('title', 'My new post')
            ->set('content', 'Post content')
            ->call('Save');

        $this->assertEquals(1, Post::count());
    }

    public function test_title_is_required()
    {
        Livewire::test('post.create')
            ->set('title', '')
            ->call('Save')
            ->assertHasErrors('title');
    }
}
```

Все функции, описанные на этой странице, работают идентично с PHPUnit — просто используйте синтаксис утверждений PHPUnit вместо синтаксиса Pest.

!!! tip "Попробуйте Pest"
    Если вам интересно познакомиться с более элегантным синтаксисом и возможностями Pest, загляните на [pestphp.com](https://pestphp.com/), чтобы узнать больше.

## Все доступные методы тестирования

Ниже приведена полная справочная информация по всем доступным методам тестирования Livewire:

### Методы настройки

| Метод                                                  | Описание                                                                                                      |
|---------------------------------------------------------|------------------------------------------------------------------------------------------------------------------|
| `Livewire::test('post.create')`                      | Тестирование компонента `post.create` |
| `Livewire::test(UpdatePost::class, ['post' => $post])`                      | Тестирование компонента `UpdatePost` с параметрами, передаваемыми в метод `mount()` |
| `Livewire::actingAs($user)`                      | Установка аутентифицированного пользователя для теста |
| `Livewire::withQueryParams(['search' => '...'])`                      | Установка параметров строки запроса URL (например, `?search=...`) |
| `Livewire::withCookie('name', 'value')`                      | Установка cookie для теста |
| `Livewire::withCookies(['color' => 'blue', 'name' => 'Taylor'])`                      | Установка нескольких cookies |
| `Livewire::withHeaders(['X-Header' => 'value'])`                      | Установка пользовательских заголовков |
| `Livewire::withoutLazyLoading()`                      | Отключение отложенной загрузки для всех компонентов в данном тесте |

### Взаимодействие с компонентами

| Метод                                                  | Описание                                                                                                      |
|---------------------------------------------------------|------------------------------------------------------------------------------------------------------------------|
| `set('title', '...')`                      | Установить свойство `title` в указанное значение |
| `set(['title' => '...', 'content' => '...'])`                      | Установить несколько свойств с помощью массива |
| `toggle('sortAsc')`                      | Переключить булево свойство между `true` и `false`  |
| `call('Save')`                      | Вызвать действие/метод `save` |
| `call('remove', $postId)`                      | Вызвать метод с параметрами |
| `refresh()`                      | Запустить повторный рендеринг компонента |
| `dispatch('post-created')`                      | Отправить событие от компонента  |
| `dispatch('post-created', postId: $post->id)`                      | Отправить событие с параметрами |

### Утверждения

| Метод                                                | Описание                                                                                                                                                                          |
|-------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `assertSet('title', '...')`                           | Утверждает, что свойство равно указанному значению                                                                                                                        |
| `assertNotSet('title', '...')`                        | Утверждает, что свойство не равно указанному значению                                                                                                                    |
| `assertCount('posts', 3)`                             | Утверждает, что свойство содержит 3 элемента                                                                                                         |
| `assertSee('...')`                             | Утверждает, что отрендеренный HTML содержит указанный текст                                                                                                           |
| `assertDontSee('...')`                         | Утверждает, что отрендеренный HTML не содержит указанный текст                                                                                                                    |
| `assertSeeHtml('<div>...</div>')`                     | Утверждает, что указанный сырой HTML присутствует в отрендеренном выводе |
| `assertDontSeeHtml('<div>...</div>')`                 | Утверждает, что указанный сырой HTML отсутствует в отрендеренном выводе                                                                                                                         |
| `assertSeeInOrder(['first', 'second'])`                    | Утверждает, что строки появляются в указанном порядке в отрендеренном выводе                                                                                        |
| `assertDispatched('post-created')`                    | Утверждает, что событие было отправлено                                                                                                                     |
| `assertNotDispatched('post-created')`                 | Утверждает, что событие не было отправлено                                                                                                                 |
| `assertHasErrors('title')`                            | Утверждает, что валидация не прошла для указанного свойства                                                                                                                           |
| `assertHasErrors(['title' => ['required', 'min:6']])`   | Утверждает, что провалились конкретные правила валидации                                                                                                            |
| `assertHasNoErrors('title')`                          | Утверждает, что для свойства нет ошибок валидации                                                                                                                  |
| `assertRedirect()`                                    | Утверждает, что было выполнено перенаправление                                                                                                                  |
| `assertRedirect('/posts')`                            | Утверждает перенаправление на конкретный URL                                                                                                                   |
| `assertRedirectToRoute('posts.index')`       | Утверждает перенаправление на именованный маршрут                                                                                                                    |
| `assertNoRedirect()`                                  | Утверждает, что перенаправление не выполнялось                                                                                                                                           |
| `assertViewHas('posts')`                              | Утверждает, что данные были переданы в представление                                                                                                         |
| `assertViewHas('postCount', 3)`                       | Утверждает, что данные представления имеют конкретное значение                                                                                                   |
| `assertViewHas('posts', function ($posts) { ... })`   | Утверждает, что данные представления проходят пользовательскую проверку                                                                         |
| `assertViewIs('livewire.show-posts')`                 | Утверждает, что было отрендерено конкретное представление                                                                                                            |
| `assertJs("alert('...')")`                            | Утверждает, что было выполнено указанное JavaScript-выражение                                                                                                                                       |
| `assertNoJs()`                                        | Утверждает, что JavaScript не выполнялся                                                                                                                                                 |
| `assertFileDownloaded()`                              | Утверждает, что было инициировано скачивание файла                                                                                                                                       |
| `assertFileDownloaded($filename)`                     | Утверждает, что был скачан конкретный файл                                                                                                       |
| `assertUnauthorized()`                                | Утверждает, что было выброшено исключение авторизации (401)                                                                                       |
| `assertForbidden()`                                   | Утверждает, что доступ запрещён (403)                                                                                                                |
| `assertStatus(500)`                                   | Утверждает, что был возвращён указанный статус-код                                                                                                                     |

## Смотрите также

- **[Действия](/essentials/actions)** — Тестирование действий и взаимодействий компонента
- **[Формы](/essentials/forms)** — Тестирование отправки форм и валидации
- **[События](/essentials/events)** — Тестирование отправки и прослушивания событий
- **[Компоненты](/essentials/components)** — Создание тестируемой структуры компонентов
