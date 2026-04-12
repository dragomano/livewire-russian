# Руководство для участников

Приветствуем вас в руководстве для участников Livewire! В этом документе мы рассмотрим, как вы можете внести свой вклад в Livewire: предлагая новые функции, исправляя тесты или устраняя ошибки.

## Локальная настройка Livewire и Alpine

Самый простой способ внести вклад — убедиться, что репозитории Livewire и Alpine настроены на вашей локальной машине. Это позволит вам легко вносить изменения и запускать тесты.

### Форк и клонирование репозиториев

Первым шагом будет форк и клонирование репозиториев. Проще всего это сделать с помощью [GitHub CLI](https://cli.github.com/), но вы также можете сделать это вручную, нажав кнопку «Fork» на [странице репозитория](https://github.com/livewire/livewire).

```shell
# Форк и клонирование Livewire
gh repo fork livewire/livewire --default-branch-only --clone=true -- livewire

# Переход в директорию livewire
cd livewire

# Установка всех зависимостей composer
composer install

# Проверка конфигурации Dusk
vendor/bin/dusk-updater detect --no-interaction
```

Для настройки Alpine убедитесь, что у вас установлен [NPM](https://docs.npmjs.com/downloading-and-installing-node-js-and-npm), а затем выполните следующие команды. Если вы предпочитаете делать форк вручную, посетите [страницу репозитория](https://github.com/alpinejs/alpine).

```shell
# Форк и клонирование Alpine
gh repo fork alpinejs/alpine --default-branch-only --clone=true --remote=false -- alpine

# Переход в директорию alpine
cd alpine

# Установка всех npm-зависимостей
npm install

# Сборка всех пакетов Alpine
npm run build

# Локальная линковка всех пакетов Alpine
cd packages/alpinejs && npm link && cd ../../
cd packages/anchor && npm link && cd ../../
cd packages/collapse && npm link && cd ../../
cd packages/csp && npm link && cd ../../
cd packages/docs && npm link && cd ../../
cd packages/focus && npm link && cd ../../
cd packages/history && npm link && cd ../../
cd packages/intersect && npm link && cd ../../
cd packages/mask && npm link && cd ../../
cd packages/morph && npm link && cd ../../
cd packages/navigate && npm link && cd ../../
cd packages/persist && npm link && cd ../../
cd packages/sort && npm link && cd ../../
cd packages/ui && npm link && cd ../../

# Возврат в директорию livewire
cd ../livewire

# Линковка всех пакетов
npm link alpinejs @alpinejs/anchor @alpinejs/collapse @alpinejs/csp @alpinejs/docs @alpinejs/focus @alpinejs/history @alpinejs/intersect @alpinejs/mask @alpinejs/morph @alpinejs/navigate @alpinejs/persist @alpinejs/sort @alpinejs/ui

# Сборка Livewire
npm run build
```

## Добавление падающего теста

Если вы столкнулись с ошибкой и не знаете, как её решить (особенно учитывая сложность ядра Livewire), вы можете начать с добавления падающего теста. Это позволит кому-то более опытному помочь в поиске и исправлении ошибки. Тем не менее, мы рекомендуем вам также изучить ядро, чтобы лучше понять принципы работы Livewire.

Давайте разберем это по шагам.

#### 1. Определите, куда добавить тест

Ядро Livewire разделено на папки, каждая из которых соответствует определенным функциям. Например:

```shell
src/Features/SupportAccessingParent
src/Features/SupportAttributes
src/Features/SupportAutoInjectedAssets
src/Features/SupportBladeAttributes
src/Features/SupportChecksumErrorDebugging
src/Features/SupportComputed
src/Features/SupportConsoleCommands
src/Features/SupportDataBinding
//...
```

Попробуйте найти функцию, связанную с ошибкой, которую вы обнаружили. Если вы не можете найти подходящую папку или не уверены в выборе, просто выберите одну из них и укажите в пулреквесте, что вам нужна помощь с размещением теста в правильном наборе функций.

#### 2. Определите тип теста

Набор тестов Livewire состоит из двух типов:

1. **Unit-тесты**: фокусируются на PHP-реализации Livewire.
2. **Browser-тесты**: выполняют последовательность шагов в реальном браузере и проверяют результат. Они в основном фокусируются на JavaScript-реализации Livewire.

Если вы не уверены, какой тип выбрать, начните с браузерного теста. Реализуйте шаги, которые вы выполняете в своем приложении и браузере для воспроизведения ошибки.

Unit-тесты следует добавлять в файл `UnitTest.php`, а браузерные тесты — в `BrowserTest.php`. Если один или оба этих файла отсутствуют, вы можете создать их самостоятельно.

**Unit-тест**

```php
<?php

use Tests\TestCase;

class UnitTest extends TestCase
{
    public function test_livewire_can_run_action(): void
    {
       // ...
    }
}
```

**Browser-тест**

```php
<?php

use Tests\BrowserTestCase;

class BrowserTest extends BrowserTestCase
{
    public function test_livewire_can_run_action()
    {
        // ...
    }
}
```

!!! tip "Не знаете, как писать тесты?"
    Вы можете многое узнать, изучая существующие Unit и Browser тесты. Даже копирование существующего теста — отличная отправная точка для написания собственного.

#### 3. Запуск тестов

Перед отправкой пулреквеста убедитесь, что ваш тест проходит. Вы можете сделать это с помощью одной из следующих команд:

```shell
vendor/bin/phpunit --filter "test_can_make_method_a_computed" # Запуск конкретного теста
vendor/bin/phpunit # Запуск всех тестов
```

По умолчанию браузерные тесты запускаются в обычном режиме. Если вы хотите запустить их в фоновом (headless) режиме, создайте файл `.env` в корне репозитория Livewire и добавьте `DUSK_HEADLESS_DISABLED=false`.

#### 4. Подготовка ветки пулреквеста

После того как вы реализовали функцию или падающий тест, пора отправить пулреквест в репозиторий Livewire. Сначала убедитесь, что вы закоммитили изменения в отдельную ветку (не используйте `main`). Для создания новой ветки используйте команду `git`:

```shell
git checkout -b my-feature
```

Вы можете назвать ветку как угодно, но лучше использовать описательное имя, отражающее суть изменений.

Затем закоммитьте изменения в свою ветку. Используйте `git add .` для добавления всех изменений и `git commit -m "Add my feature"` для коммита с описанием.

На данный момент ваша ветка доступна только локально. Чтобы создать пулреквест, нужно отправить ветку в ваш форк репозитория Livewire с помощью `git push`.

```shell
git push origin my-feature

Enumerating objects: 13, done.
Counting objects: 100% (13/13), done.
Delta compression using up to 8 threads
Compressing objects: 100% (6/6), done.

To github.com:Username/livewire.git
 * [new branch]        my-feature -> my-feature
```

#### 5. Отправка пулреквеста

Мы почти у цели! Откройте браузер и перейдите в свой форк репозитория Livewire (`https://github.com/<ваш-логин>/livewire`). Вы увидите уведомление: "**my-feature had recent pushes 1 minute ago**" вместе с кнопкой "**Compare & pull request**". Нажмите кнопку, чтобы открыть форму создания пулреквеста.

В форме укажите заголовок, описывающий ваш PR, и перейдите к описанию. Текстовая область уже содержит шаблон. Постарайтесь ответить на каждый вопрос:

```
Review the contribution guide first at: https://livewire.laravel.com/docs/contribution-guide

1️⃣ Is this something that is wanted/needed? Did you create a discussion about it first?
Yes, you can find the discussion here: https://github.com/livewire/livewire/discussions/999999

2️⃣ Did you create a branch for your fix/feature? (Main branch PR's will be closed)
Yes, the branch is named `my-feature`

3️⃣ Does it contain multiple, unrelated changes? Please separate the PRs out.
No, the changes are only related to my feature.

4️⃣ Does it include tests? (Required)
Yes

5️⃣ Please include a thorough description (including small code snippets if possible) of the improvement and reasons why it's useful.

These changes will improve memory usage. You can see the benchmark results here:

// ...

```

Всё готово? Нажмите **Create pull request** 🚀 Поздравляем! Вы создали свой первый вклад в проект 🎉

Поддерживающие проект (мейнтейнеры) рассмотрят ваш PR и могут оставить отзывы или запросить изменения. Пожалуйста, постарайтесь ответить на отзывы как можно быстрее.

Спасибо за ваш вклад в Livewire!
