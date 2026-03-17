# Загрузка файлов

Livewire предоставляет мощную поддержку загрузки файлов прямо внутри ваших компонентов.

Сначала добавьте трейт `WithFileUploads` в ваш компонент. После того как этот трейт добавлен в компонент, вы можете использовать `wire:model` на полях ввода типа file точно так же, как и с любыми другими типами полей, а Livewire позаботится обо всём остальном.

Вот пример простого компонента, который обрабатывает загрузку фотографии:

```php title="resources/views/components/⚡upload-photo.blade.php"
<?php

use Livewire\Attributes\Validate;
use Livewire\WithFileUploads;
use Livewire\Component;

new class extends Component {
    use WithFileUploads;

    #[Validate('image|max:1024')] // максимум 1MB
    public $photo;

    public function save()
    {
        $this->photo->store(path: 'photos');
    }
};
```

```html
<form wire:submit="save">
    <input type="file" wire:model="photo">

    @error('photo') <span class="error">{{ $message }}</span> @enderror

    <button type="submit">Сохранить фото</button>
</form>
```

!!! warning "Метод `upload` зарезервирован"
    Обратите внимание, что в примере выше используется метод `save` вместо `upload`. Это распространённая ловушка. Слово `upload` зарезервировано Livewire. Вы не можете использовать его как имя метода или свойства в вашем компоненте.

С точки зрения разработчика, работа с полями ввода файлов ничем не отличается от работы с любыми другими типами полей: добавляете `wire:model` к тегу `<input>` — и всё остальное Livewire делает за вас.

Однако под капотом происходит гораздо больше, чтобы загрузка файлов в Livewire работала. Вот краткий обзор того, что происходит, когда пользователь выбирает файл для загрузки:

1. Когда выбирается новый файл, JavaScript Livewire отправляет начальный запрос к компоненту на сервере, чтобы получить временный «подписанный» URL для загрузки.
2. Получив URL, JavaScript выполняет саму загрузку файла по этому подписанному URL, сохраняя файл во временную директорию, определённую Livewire, и возвращает уникальный хеш-ID нового временного файла.
3. После успешной загрузки файла и получения уникального хеш-ID, JavaScript Livewire делает финальный запрос к компоненту на сервере, сообщая ему, что нужно «установить» указанное публичное свойство в новый временный файл.
4. Теперь публичное свойство (в данном случае `$photo`) содержит временно загруженный файл и готово к сохранению или валидации в любой момент.

## Сохранение загруженных файлов

Предыдущий пример демонстрирует самый простой сценарий сохранения: перемещение временно загруженного файла в директорию «photos» на диске файловой системы приложения по умолчанию.

Однако вы можете захотеть настроить имя сохраняемого файла или указать конкретный диск хранения (например, S3).

!!! tip "Оригинальные имена файлов"
    Вы можете получить оригинальное имя загруженного файла, вызвав метод `->getClientOriginalName()` у временной загрузки.

Livewire использует те же API, что и Laravel для сохранения загруженных файлов, поэтому смело обращайтесь к [документации Laravel по загрузке файлов](https://laravel.com/docs/filesystem#file-uploads). Ниже приведены несколько распространённых сценариев сохранения и примеры:

```php
<?php

public function save()
{
    // Сохраняем файл в директорию "photos" диска файловой системы по умолчанию
    $this->photo->store(path: 'photos');

    // Сохраняем файл в директорию "photos" на настроенном диске "s3"
    $this->photo->store(path: 'photos', options: 's3');

    // Сохраняем файл в директорию "photos" с именем файла "avatar.png"
    $this->photo->storeAs(path: 'photos', name: 'avatar');

    // Сохраняем файл в директорию "photos" на настроенном диске "s3" с именем файла "avatar.png"
    $this->photo->storeAs(path: 'photos', name: 'avatar', options: 's3');

    // Сохраняем файл в директорию "photos" с публичной видимостью ("public") на настроенном диске "s3"
    $this->photo->storePublicly(path: 'photos', options: 's3');

    // Сохраняем файл в директорию "photos" с именем "avatar.png" и публичной видимостью ("public") на настроенном диске "s3"
    $this->photo->storePubliclyAs(path: 'photos', name: 'avatar', options: 's3');
}
```

## Обработка нескольких файлов

Livewire автоматически обрабатывает загрузку нескольких файлов, определяя наличие атрибута `multiple` у тега `<input>`.

Например, ниже приведён компонент со свойством-массивом под названием `$photos`. Добавив атрибут `multiple` к полю ввода файла в форме, Livewire будет автоматически добавлять новые файлы в этот массив:

```php title="resources/views/components/⚡upload-photos.blade.php"
<?php

use Livewire\Attributes\Validate;
use Livewire\WithFileUploads;
use Livewire\Component;

new class extends Component {
    use WithFileUploads;

    #[Validate(['photos.*' => 'image|max:1024'])]
    public $photos = [];

    public function save()
    {
        foreach ($this->photos as $photo) {
            $photo->store(path: 'photos');
        }
    }
};
```

```html
<form wire:submit="save">
    <input type="file" wire:model="photos" multiple>

    @error('photos.*') <span class="error">{{ $message }}</span> @enderror

    <button type="submit">Сохранить фото</button>
</form>
```

## Валидация файлов

Как мы уже обсуждали, валидация загруженных файлов в Livewire выполняется точно так же, как обработка загрузки файлов в обычном контроллере Laravel.

!!! warning "Убедитесь, что S3 правильно настроен"
    Многие правила валидации, связанные с файлами, требуют доступа к самому файлу. При [прямой загрузке в S3](#прямая-загрузка-в-amazon-s3) эти правила валидации не сработают, если объект файла в S3 не является публично доступным.

Подробную информацию о валидации файлов смотрите в [документации Laravel по правилам валидации файлов](https://laravel.com/docs/validation#available-validation-rules).

## Временные URL для предпросмотра

После того как пользователь выбрал файл, обычно следует показать ему предпросмотр этого файла ещё до отправки формы и сохранения файла.

Livewire делает это очень просто с помощью метода `->temporaryUrl()` у загруженных файлов.

!!! info "Временные URL поддерживаются только для изображений"
    По соображениям безопасности временные URL предпросмотра работают только для файлов с MIME-типами изображений.

Давайте рассмотрим пример загрузки файла с предпросмотром изображения:

```php title="resources/views/components/⚡upload-photo.blade.php"
<?php

use Livewire\Attributes\Validate;
use Livewire\WithFileUploads;
use Livewire\Component;

new class extends Component {
    use WithFileUploads;

    #[Validate('image|max:1024')]
    public $photo;

    // ...
};
```

```html hl_lines="2-4"
<form wire:submit="save">
    @if ($photo)
        <img src="{{ $photo->temporaryUrl() }}">
    @endif

    <input type="file" wire:model="photo">

    @error('photo') <span class="error">{{ $message }}</span> @enderror

    <button type="submit">Сохранить фото</button>
</form>
```

Как уже обсуждалось ранее, Livewire сохраняет временные файлы в непубличной директории, поэтому обычно нет простого способа предоставить пользователям временный публичный URL для предпросмотра изображений.

Однако Livewire решает эту проблему, предоставляя временный подписанный URL, который «притворяется» загруженным изображением, чтобы ваша страница могла показать предпросмотр изображения пользователям.

Этот URL защищён от показа файлов из директорий, находящихся выше временной директории. А поскольку он подписан, пользователи не могут злоупотреблять этим URL, чтобы просматривать другие файлы в вашей системе.

!!! tip "Временные подписанные URL для S3"
    Если вы настроили Livewire на использование S3 для хранения временных файлов, вызов метода `->temporaryUrl()` будет генерировать временный подписанный URL напрямую к S3, так что предпросмотр изображений не будет загружаться через сервер вашего Laravel-приложения.

## Тестирование загрузки файлов

Вы можете использовать стандартные вспомогательные методы Laravel для тестирования загрузки файлов.

Ниже приведён полный пример тестирования компонента `UploadPhoto` с помощью Livewire:

```php
<?php

namespace Tests\Feature\Livewire;

use Illuminate\Http\UploadedFile;
use Illuminate\Support\Facades\Storage;
use App\Livewire\UploadPhoto;
use Livewire\Livewire;
use Tests\TestCase;

class UploadPhotoTest extends TestCase
{
    public function test_can_upload_photo()
    {
        Storage::fake('avatars');

        $file = UploadedFile::fake()->image('avatar.png');

        Livewire::test(UploadPhoto::class)
            ->set('photo', $file)
            ->call('upload', 'uploaded-avatar.png');

        Storage::disk('avatars')->assertExists('uploaded-avatar.png');
    }
}
```

Ниже приведён пример компонента `upload-photo`, который требуется для того, чтобы предыдущий тест прошёл успешно:

```php title="resources/views/components/⚡upload-photo.blade.php"
<?php

use Livewire\WithFileUploads;
use Livewire\Component;

new class extends Component {
    use WithFileUploads;

    public $photo;

    public function upload($name)
    {
        $this->photo->storeAs('/', $name, disk: 'avatars');
    }

    // ...
};
```

Подробную информацию о тестировании загрузки файлов смотрите в [документации Laravel по тестированию загрузки файлов](https://laravel.com/docs/http-tests#testing-file-uploads).

## Прямая загрузка в Amazon S3

Как уже обсуждалось ранее, Livewire сохраняет все загруженные файлы во временной директории до тех пор, пока разработчик не сохранит их окончательно.

По умолчанию Livewire использует диск файловой системы, заданный по умолчанию (обычно `local`), и хранит файлы в поддиректории `livewire-tmp/`.

В результате загрузка файлов всегда проходит через сервер вашего приложения, даже если позже вы сохраняете эти файлы в бакет S3.

Если вы хотите обойти сервер приложения и загружать временные файлы Livewire напрямую в бакет S3, установите переменную окружения `LIVEWIRE_TEMPORARY_FILE_UPLOAD_DISK` в вашем файле `.env` в значение `s3` (или в другое пользовательское имя диска, которое использует драйвер `s3`):

```env
LIVEWIRE_TEMPORARY_FILE_UPLOAD_DISK=s3
```

Теперь при загрузке файла пользователем он вообще не будет сохраняться на вашем сервере. Вместо этого файл будет сразу загружен напрямую в ваш бакет S3 в поддиректорию `livewire-tmp/`.

!!! tip "Совет"
    Альтернативно, вы можете опубликовать конфигурационный файл Livewire командой `php artisan livewire:config`, чтобы получить полный контроль над настройками секции `temporary_file_upload`.

Настройка автоматической очистки файлов

Директория временных загрузок Livewire быстро заполняется файлами, поэтому крайне важно настроить автоматическую очистку файлов в S3 старше 24 часов.

Чтобы настроить такое поведение, выполните следующую Artisan-команду в окружении, которое использует бакет S3 для временных загрузок файлов:

```shell
php artisan livewire:configure-s3-upload-cleanup
```

Теперь любые временные файлы старше 24 часов будут автоматически удаляться из S3.

!!! info "Информация"
    Если вы не используете S3 для хранения файлов, Livewire будет самостоятельно заниматься очисткой временных файлов и вам **не нужно** выполнять указанную выше команду.

## Индикаторы загрузки

Хотя `wire:model` для загрузки файлов работает под капотом иначе, чем для остальных типов полей ввода с `wire:model`, интерфейс отображения индикаторов загрузки остаётся тем же самым.

Вы можете показать индикатор загрузки, привязанный именно к загрузке файла, с помощью директивы `wire:loading`:

```html
<input type="file" wire:model="photo">

<div wire:loading wire:target="photo">Загрузка...</div>
```

Или ещё проще — используя автоматический атрибут `data-loading`, который добавляет Livewire:

```html
<div>
    <input type="file" wire:model="photo">

    <div class="not-data-loading:hidden">Загрузка...</div>
</div>
```

Теперь, пока идёт загрузка файла, будет отображаться сообщение «Загрузка...», а после завершения загрузки оно автоматически скроется.

[Подробнее о состояниях загрузки →](/features/loading-states)

## Индикаторы прогресса

Каждая операция загрузки файла в Livewire отправляет специальные JavaScript-события на соответствующий элемент `<input>`, что позволяет вашему собственному JavaScript-коду перехватывать эти события:

Событие                     | Описание
----------------------------|-------------------------------------------------
`livewire-upload-start`     | Отправляется в момент начала загрузки
`livewire-upload-finish`    | Отправляется при успешном завершении загрузки
`livewire-upload-cancel`    | Отправляется, если загрузка была прервана досрочно
`livewire-upload-error`     | Отправляется при ошибке загрузки
`livewire-upload-progress`  | Отправляется периодически во время загрузки, содержит процент прогресса

Ниже приведён пример обёртывания поля загрузки файла Livewire в компонент Alpine.js для отображения полосы прогресса загрузки:

```html
<form wire:submit="save">
    <div
        x-data="{ uploading: false, progress: 0 }"
        x-on:livewire-upload-start="uploading = true"
        x-on:livewire-upload-finish="uploading = false"
        x-on:livewire-upload-cancel="uploading = false"
        x-on:livewire-upload-error="uploading = false"
        x-on:livewire-upload-progress="progress = $event.detail.progress"
    >
        <!-- Поле ввода файла -->
        <input type="file" wire:model="photo">

        <!-- Полоса прогресса -->
        <div x-show="uploading">
            <progress max="100" x-bind:value="progress"></progress>
        </div>
    </div>

    <!-- ... -->
</form>
```

## Отмена загрузки

Если загрузка занимает слишком много времени, пользователь может захотеть её отменить. Эту функциональность можно реализовать с помощью функции `$cancelUpload()` в JavaScript, предоставляемой Livewire.

Вот пример создания кнопки «Отменить загрузку» в компоненте Livewire с использованием `wire:click` для обработки клика:

```html
<form wire:submit="save">
    <!-- Поле ввода файла -->
    <input type="file" wire:model="photo">

    <!-- Кнопка отмены загрузки -->
    <button type="button" wire:click="$cancelUpload('photo')">Отменить загрузку</button>

    <!-- ... -->
</form>
```

Когда нажата кнопка «Отменить загрузку», запрос на загрузку файла будет прерван, а поле ввода файла — очищено. Пользователь теперь может попытаться загрузить другой файл.

Альтернативно, вы можете вызвать метод `cancelUpload(...)` из Alpine.js следующим образом:

```html
<button type="button" x-on:click="$wire.cancelUpload('photo')">Отменить загрузку</button>
```

## JavaScript API загрузки

Интеграция со сторонними библиотеками загрузки файлов часто требует большего контроля, чем простое использование элемента `<input type="file" wire:model="...">`.

Для таких случаев Livewire предоставляет специальные JavaScript-функции.

Эти функции доступны на объекте JavaScript-компонента, который можно удобно получить через объект `$wire` внутри шаблона вашего Livewire-компонента:

```html
<script>
    let file = $wire.el.querySelector('input[type="file"]').files[0]

    // Загрузка одного файла...
    $wire.upload('photo', file, (uploadedFilename) => {
        // Коллбэк успешной загрузки...
    }, () => {
        // Коллбэк ошибки...
    }, (event) => {
        // Коллбэк прогресса...
        // event.detail.progress содержит число от 1 до 100 по мере загрузки
    }, () => {
        // Коллбэк отмены...
    })

    // Загрузка нескольких файлов...
    $wire.uploadMultiple('photos', [file], successCallback, errorCallback, progressCallback, cancelledCallback)

    // Удаление одного файла из уже загруженных нескольких...
    $wire.removeUpload('photos', uploadedFilename, successCallback)

    // Отмена загрузки...
    $wire.cancelUpload('photos')
</script>
```

## Конфигурация

Поскольку Livewire сохраняет все загруженные файлы временно до того, как разработчик их проверит или сохранит, он предполагает некоторое поведение обработки по умолчанию для всех загрузок файлов.

### Глобальная валидация

По умолчанию Livewire валидирует все временные загруженные файлы с помощью следующих правил: `file|max:12288` (должен быть файл размером менее 12 МБ).

Если вы хотите изменить эти правила, сделать это можно в файле конфигурации вашего приложения `config/livewire.php`:

```php
<?php

'temporary_file_upload' => [
    // ...
    'rules' => 'file|mimes:png,jpg,pdf|max:102400', // (максимум 100 МБ, принимаются только PNG, JPEG и PDF)
],
```

### Глобальный мидлвар

Конечная точка временной загрузки файлов по умолчанию оснащена мидлваром для ограничения частоты запросов. Вы можете настроить, какой именно мидлвар будет применяться к этой конечной точке, через следующую опцию конфигурации:

```php
<?php

'temporary_file_upload' => [
    // ...
    'middleware' => 'throttle:5,1', // Разрешить только 5 загрузок на пользователя в минуту
],
```

### Директория временных загрузок

Временные файлы загружаются в директорию `livewire-tmp/` указанного диска. Вы можете изменить эту директорию с помощью следующей опции конфигурации:

```php
<?php

'temporary_file_upload' => [
    // ...
    'directory' => 'tmp',
],
```

## Смотрите также

- **[Формы](/essentials/forms)** — Обработка загрузки файлов в формах
- **[Валидация](/features/validation)** — Проверка загруженных файлов
- **[Состояния загрузки](/features/loading-states)** — Отображение индикаторов прогресса загрузки
- **[wire:model](/html-directives/wire-model)** — Привязка полей ввода файлов к свойствам
