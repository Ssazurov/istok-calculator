# Обслуживание сайта istok-calculator

Статический сайт на GitHub Pages. Источник — ветка `main`, папка `/` (root).
Сайт: https://ssazurov.github.io/istok-calculator/

## Структура репозитория
```
index.html          — вся страница (HTML + CSS + JS в одном файле)
istok_logo.png       — лого: favicon + шапка сайта (34×34)
istok_logo_big.png   — лого большого разрешения, справочно, в проде не используется
README.md            — публичное описание для посетителей репо
MAINTENANCE.md       — этот файл
```

## Единственное рабочее место
```
\\wsl.localhost\Ubuntu\home\vector\projects\istok-calculator
```
Работать **только** отсюда. Любой другой клон/копия репозитория — источник
конфликтов, и на сайт публикуется не то, что вы правили локально (уже
случалось: force-push восстанавливал актуальную версию).

## Как обновить сайт

1. Внести правки в `index.html` (или заменить картинку).
2. Закоммитить и запушить:

   git add -A
   git commit -m "v1.4: краткое описание изменения"
   git push

3. GitHub Actions сам соберёт и задеплоит Pages (~30–60 сек). Статус:

   gh run list -R Ssazurov/istok-calculator --limit 3

   Ждать строку `completed  success`.
4. Открыть сайт и проверить (см. раздел про кэш ниже).

## Версионирование
В коммитах и, по возможности, в самом UI — держим версию (`v1.3`, `v1.4`...),
проще понять, какая версия реально задеплоена.

## Кэш — почему обновление иногда "не видно"

GitHub Pages отдаётся через CDN (Fastly) с `cache-control: max-age=600` —
до 10 минут файл может отдаваться из кэша по старому URL. Плюс браузер
кэширует картинки агрессивно и самостоятельно.

**Для картинок/статики** — cache-busting через query-параметр версии,
уже сделано для лого:

    <img src="istok_logo.png?v=1.3">

При каждом визуальном изменении лого — увеличивать `?v=`.

**Проверка, что реально отдаёт GitHub (без браузерного кэша):**

    # что реально в репозитории, минуя CDN Pages
    curl -s https://raw.githubusercontent.com/Ssazurov/istok-calculator/main/index.html | md5sum

    # что отдаёт сам сайт через CDN — смотреть last-modified / age / x-cache
    curl -sI https://ssazurov.github.io/istok-calculator/istok_logo.png

Если raw.githubusercontent.com уже отдаёт новое, а сайт — старое: подождать
до 10 минут или добавить/увеличить `?v=` у ресурса.

Если и raw отдаёт старое — пуш не дошёл или ушёл не туда:

    git remote -v                       # должен быть Ssazurov/istok-calculator
    git log origin/main --oneline -5    # сверить с локальным HEAD

## Диагностика деплоя, если сайт лежит (404 / не обновляется)

    gh api repos/Ssazurov/istok-calculator/pages          # статус Pages
    gh run list -R Ssazurov/istok-calculator --limit 5     # последние прогоны
    gh run view <RUN_ID> -R Ssazurov/istok-calculator --log-failed

Частая причина — временный 503 от GitHub Pages API (сбой на их стороне,
не в коде). Лечится повторным запуском:

    gh run rerun <RUN_ID> -R Ssazurov/istok-calculator --failed

Если rerun сам зависает в `queued` — надёжнее пустой коммит, он триггерит
свежий прогон workflow вместо реанимации старого:

    git commit --allow-empty -m "chore: retry deploy" && git push

## Чек-лист перед коммитом

- [ ] Работаю из `\\wsl.localhost\Ubuntu\home\vector\projects\istok-calculator`
- [ ] `git status` чистый / изменения ожидаемые (`git diff` перед `add -A`)
- [ ] Если менял картинку с тем же именем — обновил `?v=` в `index.html`
- [ ] Осмысленное сообщение коммита с версией
- [ ] После пуша проверил `gh run list` → `success`
- [ ] Открыл сайт в приватном окне (без кэша браузера)
