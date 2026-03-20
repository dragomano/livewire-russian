# Пакеты

Чтобы включить компоненты Livewire в пакет Laravel, вам нужно зарегистрировать их в сервис-провайдере вашего пакета.

## Однофайловые и многофайловые компоненты

Для однофайловых (SFC) и многофайловых (MFC) компонентов используйте метод `addNamespace` в методе `boot()` вашего сервис-провайдера:

```php
<?php

use Livewire\Livewire;

public function boot(): void
{
    Livewire::addNamespace(
        namespace: 'mypackage',
        viewPath: __DIR__ . '/../resources/views/livewire',
    );
}
```

Это регистрирует все SFC и MFC компоненты в директории `resources/views/livewire` вашего пакета под пространством имён `mypackage`.

**Использование:**

```php
<livewire:mypackage::counter />
<livewire:mypackage::users.table />
```

## Компоненты на основе классов

Для компонентов на основе классов вам потребуется указать дополнительные параметры и зарегистрировать ваши представления в Laravel:

```php
<?php

use Livewire\Livewire;

public function boot(): void
{
    Livewire::addNamespace(
        namespace: 'mypackage',
        classNamespace: 'MyVendor\\MyPackage\\Livewire',
        classPath: __DIR__ . '/Livewire',
        classViewPath: __DIR__ . '/../resources/views/livewire',
    );

    $this->loadViewsFrom(__DIR__ . '/../resources/views', 'my-package');
}
```

Метод `render()` вашего компонента должен ссылаться на представление, используя синтаксис пространства имён пакета Laravel:

```php
<?php

public function render()
{
    return view('my-package::livewire.counter');
}
```

**Использование:**

```php
<livewire:mypackage::counter />
```

## Именование файлов

Префикс в виде эмодзи ⚡, используемый в именах файлов компонентов Livewire, может вызвать проблемы с Composer при публикации пакетов. При разработке пакетов избегайте использования эмодзи молнии в именах файлов компонентов — используйте `counter.blade.php` вместо `⚡counter.blade.php`.
