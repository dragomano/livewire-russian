# Атрибут `Renderless`

Атрибут `#[Renderless]` пропускает фазу рендеринга жизненного цикла Livewire при вызове действия, повышая производительность для действий, которые не изменяют представление компонента.

## Базовое использование

Примените атрибут `#[Renderless]` к любому методу действия, которому не нужно повторно рендерить компонент:

```php hl_lines="15" title="resources/views/components/post/⚡show.blade.php"
<?php

use Livewire\Attributes\Renderless;
use Livewire\Component;
use App\Models\Post;

new class extends Component {
    public Post $post;

    public function mount(Post $post)
    {
        $this->post = $post;
    }

    #[Renderless]
    public function incrementViewCount()
    {
        $this->post->incrementViewCount();
    }
};
```

```html
<div>
    <h1>{{ $post->title }}</h1>
    <p>{{ $post->content }}</p>

    <div wire:intersect="incrementViewCount"></div>
</div>
```

В приведённом выше примере используется `wire:intersect` для вызова `incrementViewCount()`, когда пользователь прокручивает страницу до конца. Поскольку применен атрибут `#[Renderless]`, количество просмотров регистрируется, но шаблон не рендерится повторно — ни одна часть страницы не затрагивается.

## Когда использовать

Используйте `#[Renderless]`, когда действие:

* Выполняет только операции на бэкенде (логирование, аналитика, отслеживание)
* Не изменяет свойства, влияющие на отображаемое представление
* Должно запускаться часто, не вызывая ненужных повторных рендерингов

Общие случаи использования включают:

* Отслеживание взаимодействия с пользователем (клики, прокрутка, время на странице)
* Отправка событий аналитики
* Обновление счётчиков или метрик
* Выполнение фоновых операций

## Альтернативные подходы

### Использование skipRender()

Если вам нужно пропустить рендеринг по условию или вы предпочитаете не использовать атрибуты, вы можете вызвать `skipRender()` непосредственно в вашем действии:

```php hl_lines="13" title="resources/views/components/post/⚡show.blade.php"
<?php

use Livewire\Component;
use App\Models\Post;

new class extends Component {
    public Post $post;

    public function incrementViewCount()
    {
        $this->post->incrementViewCount();

        $this->skipRender();
    }
};
```

### Использование модификатора .renderless

Вы также можете пропустить рендеринг прямо из элемента, используя модификатор `.renderless`:

```html
<button type="button" wire:click.renderless="incrementViewCount">
    Отследить просмотр
</button>
```

Этот подход полезен для разовых случаев, когда вы не хотите добавлять атрибут к методу.

## Узнать больше

Для получения дополнительной информации о действиях и оптимизации производительности см. [документацию по действиям](/essentials/actions#пропуск-повторного-рендеринга).
