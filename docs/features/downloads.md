# Скачивание файлов

Скачивание файлов в Livewire работает почти так же, как и в самом Laravel. Как правило, вы можете использовать любую утилиту Laravel для скачивания внутри компонента Livewire, и она будет работать как и ожидается.

Однако под капотом скачивание файлов обрабатывается иначе, чем в стандартном приложении Laravel. При использовании Livewire содержимое файла кодируется в Base64, отправляется на фронтенд и декодируется обратно в бинарный формат для скачивания напрямую с клиента.

## Использование

Инициировать скачивание файла в Livewire так же просто, как вернуть стандартный ответ Laravel для скачивания.

Ниже приведен пример компонента `show-invoice`, который содержит кнопку «Скачать» для скачивания PDF-счета:

```php hl_lines="16-18" title="resources/views/components/⚡show-invoice.blade.php"
<?php

use Livewire\Component;
use App\Models\Invoice;

new class extends Component {
    public Invoice $invoice;

    public function mount(Invoice $invoice)
    {
        $this->invoice = $invoice;
    }

    public function download()
    {
        return response()->download(
            $this->invoice->file_path, 'invoice.pdf'
        );
    }
};
```

```html hl_lines="7"
<div>
    <h1>{{ $invoice->title }}</h1>

    <span>{{ $invoice->date }}</span>
    <span>{{ $invoice->amount }}</span>

    <button type="button" wire:click="download">Скачать</button>
</div>
```

Как и в контроллере Laravel, вы также можете использовать фасад `Storage` для инициации скачивания:

```php
<?php

public function download()
{
    return Storage::disk('invoices')->download('invoice.csv');
}
```

## Потоковое скачивание

Livewire также поддерживает потоковое скачивание; однако на самом деле оно не является потоковым. Скачивание не инициируется до тех пор, пока содержимое файла не будет собрано и доставлено в браузер:

```php
<?php

public function download()
{
    return response()->streamDownload(function () {
        echo '...'; // Прямой вывод содержимого для скачивания...
    }, 'invoice.pdf');
}
```

## Тестирование скачивания файлов

Livewire также предоставляет метод `->assertFileDownloaded()`, позволяющий легко проверить, что файл был скачан с заданным именем:

```php
<?php

use Illuminate\Foundation\Testing\RefreshDatabase;
use Illuminate\Support\Facades\Storage;
use App\Models\Invoice;
use Livewire\Livewire;

uses(RefreshDatabase::class);

it('can download an invoice', function () {
    Storage::fake('invoices');
    Storage::disk('invoices')->put('invoice.pdf', 'Invoice contents');

    $invoice = Invoice::factory()->create([
        'file_path' => Storage::disk('invoices')->path('invoice.pdf'),
    ]);

    Livewire::test('show-invoice', ['invoice' => $invoice])
        ->call('download')
        ->assertFileDownloaded('invoice.pdf');
});
```

Вы также можете использовать метод `->assertNoFileDownloaded()`, чтобы проверить, что рендеринг компонента не вызывает скачивание файла:

```php
<?php

use Illuminate\Foundation\Testing\RefreshDatabase;
use App\Models\Invoice;
use Livewire\Livewire;

uses(RefreshDatabase::class);

it('does not download an invoice until requested', function () {
    $invoice = Invoice::factory()->create();

    Livewire::test('show-invoice', ['invoice' => $invoice])
        ->assertNoFileDownloaded();
});
```
