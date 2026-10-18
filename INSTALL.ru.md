# Установка на Windows

Проверено с DeepSeek Harness Desktop **0.2.1-rc.2** и Session Manager
**0.5.0**. Для перевода ядра наличие Session Manager не обязательно.
Другие версии требуют повторной проверки совместимости.

## 1. Скачать и проверить архив

Откройте [релиз 1.1.0](https://github.com/hokageantoha-spec/dsh-russian-complete/releases/tag/v1.1.0)
и скачайте `dsh-russian-complete-1.1.0.tgz` вместе с
`dsh-russian-complete-1.1.0.tgz.sha256` в одну папку.

В PowerShell укажите путь к скачанному файлу и проверьте SHA-256:

```powershell
$archive = Join-Path $env:USERPROFILE 'Downloads\dsh-russian-complete-1.1.0.tgz'
$expected = ((Get-Content "$archive.sha256" -Raw).Trim() -split '\s+')[0]
$actual = (Get-FileHash -LiteralPath $archive -Algorithm SHA256).Hash
if ($actual -ne $expected) { throw 'Контрольная сумма не совпала. Не устанавливайте архив.' }
```

## 2. Сохранить профиль и закрыть DSH

По умолчанию профиль находится в `%USERPROFILE%\.dsh\profiles\desktop`.
Если используется другой профиль, укажите его фактический путь.

Полностью завершите DSH через **Application → Exit** и сохраните копию
каталога профиля вместе с `node_modules`, `package.json`, lock-файлами и
конфигурацией Cordis:

```powershell
$profileDir = Join-Path $env:USERPROFILE '.dsh\profiles\desktop'
if (-not (Test-Path -LiteralPath (Join-Path $profileDir 'package.json'))) { throw 'Проверьте путь профиля' }
$backupDir = Join-Path $env:USERPROFILE ('dsh-desktop-backup-' + (Get-Date -Format 'yyyyMMdd-HHmmss'))
Copy-Item -LiteralPath $profileDir -Destination $backupDir -Recurse -Force
```

Не включайте одновременно несколько русских пакетов локализации: они
могут конфликтовать при регистрации одного пространства имён для `ru`.

## 3. Установить архив через пакетный менеджер профиля

Используйте pnpm той версии, которой создан профиль. Для стандартной
установки DSH 0.2.1-rc.2 bundled pnpm находится по следующему пути; команда
ниже использует установленный в системе Node.js:

```powershell
$pnpm = Join-Path $env:LOCALAPPDATA 'Programs\DeepSeek Harness\resources\runtime\primary-runtime\dependencies\pnpm\bin\pnpm.mjs'
if (-not (Test-Path -LiteralPath $pnpm)) { throw 'Укажите фактический путь bundled pnpm' }
node $pnpm --dir $profileDir add $archive --ignore-scripts
if ($LASTEXITCODE -ne 0) { throw 'Установка завершилась ошибкой. Восстановите резервную копию профиля.' }
```

Если pnpm подходящей версии уже доступен в PATH, эквивалентная команда:

```powershell
pnpm --dir $profileDir add $archive --ignore-scripts
```

Сам русификатор не требует скачивания дополнительных зависимостей.
При обработке профиля pnpm может обращаться к источникам других уже
закреплённых зависимостей.

## 4. Подключить пакет к профилю

При первой установке откройте `package.json` профиля и **добавьте**
`"dsh-russian-complete"` в существующий массив `dsh.profile.bundles`.
Сохраните остальные элементы. Пример фрагмента:

```json
{
  "dsh": {
    "profile": {
      "bundles": [
        "@deepseek-ai/dsh-base",
        "@deepseek-ai/dsh-web-app",
        "dsh-russian-complete"
      ]
    }
  }
}
```

Это фрагмент для пояснения: не заменяйте им весь файл профиля. При
обновлении с 1.0.0 запись уже должна существовать; повторно добавлять её
не нужно.

## 5. Проверить загрузку

Откройте DSH. В разделе **Плагины** пакет должен быть включён, версия —
**1.1.0**, компонент — **«Работает»**. Затем выберите
**Настройки → Общие → Язык → Русский**. Предпочтение языка сохраняется
DSH в `cordis.patch.yml` активного профиля.

Если установлен Session Manager 0.5.0, в настройках появится русская
страница **«Управление сессиями»**. Английские названия некоторых
экспериментальных карточек и нативного меню остаются ограничением API.

## Если нужно восстановить прежнюю версию

Полностью закройте DSH. Сохраните текущий каталог профиля под другим
именем и верните полную резервную копию на исходный путь. Она содержит
реальные файлы прежнего плагина и конфигурацию; повторное скачивание
пакета для такого восстановления не требуется. Затем снова откройте DSH.

## Сборка из исходников

Для сборки и тестов нужен Node.js 24. Сборка готового клиентского бандла:

```powershell
npm run build
npm run check
npm test
npm run report
npm run package
```

Тесты совместимости требуют установленного целевого DSH и Session Manager.
Готовый архив появится в `dist/`. Для обычной установки используйте файлы
готового релиза; запуск сборки пользователю не требуется.
