# Фрагменты — карта сборки

Файлы регламентов собираются из блоков, а не выбираются целиком: комбинаций стратегии, ревью, задач и выката слишком много, чтобы держать готовые варианты.

Правила сборки:

- блоки вставляются **в порядке таблицы**, лишние пропускаются;
- заголовок `#` берётся из `header`, остальные блоки начинаются с `##`;
- `{{...}}` заменяются на ответы интервью;

**Имена в примерах команд подставляются, а не копируются дословно.** Во фрагментах стоят значения по умолчанию; при сборке они заменяются на фактические:

| В шаблоне | Чем заменить |
|---|---|
| `main` | имя общей ветки. При ветках окружений — та ветка, о которой идёт речь в конкретном блоке |
| `origin/main` как база новой ветки | при ветках окружений база — **тестовая** ветка, а не прод |
| `42-краткое-описание` | шаблон имён из настроек интеграции. Задачи не ведутся → без номера: `краткое-описание` |
| `git branch -D` | `-D` нужен только при схлопывании: git не видит схлопнутую ветку как влитую. Не схлопываем → `-d` |

Пропущенная подстановка — самый частый дефект сборки: регламент выглядит правильно, но команды в нём ведут не туда.

- после сборки в файле не остаётся ни `{{`, ни строк вида `<!-- фрагмент ... -->`.

## `docs/workflow/git-workflow.md`

| # | Блок | Когда включается |
|---|---|---|
| 1 | `git/header` | всегда |
| 2 | `git/main-meaning-auto` | `prod=auto` |
| 2 | `git/main-meaning-gated` | `prod=gated` |
| 2 | `git/main-meaning-manual` | `prod=manual` |
| 2 | `git/main-meaning-stable` | `prod=none` |
| 3 | `git/branches-numbered` | `strategy ≠ trunk` И `tasks ≠ none` |
| 3 | `git/branches-plain` | `strategy ≠ trunk` И `tasks=none` |
| 3а | `git/branches-env` | `strategy=env-branches` (дополнительно к 3) |
| 4 | `git/commits-conventional` · `commits-type-verb` · `commits-task-id` · `commits-free` | ровно один, по настройке формата коммитов; язык подставляется отдельным ответом |
| 5 | `git/lifecycle-trunk` | `strategy=trunk` |
| 5 | `git/lifecycle-branch` | `strategy=branch-per-task` |
| 5 | `git/lifecycle-env` | `strategy=env-branches` |
| 6 | `git/accept-none` | `review=none` И `strategy ≠ trunk` |
| 6 | `git/accept-self-check` | `review=self-check` |
| 6 | `git/accept-ship-show-ask` | `review=ship-show-ask` |
| 6 | `git/accept-required` | `review=required` |
| 7 | `git/integration-settings` | `strategy ≠ trunk` |
| 8 | `git/cameo-fix` | `strategy ≠ trunk` |
| 9 | `git/revert-pr` | `strategy ≠ trunk` |
| 9 | `git/revert-direct` | `strategy=trunk` |
| 10 | `git/forking` | репозиторий публичный или есть внешние участники |
| 11 | `git/cheatsheet` | всегда; строки набираются по включённым осям |
| 12 | `git/protection-github` | `strategy ≠ trunk` И место = GitHub |
| 12 | `git/protection-gitlab` | `strategy ≠ trunk` И место = GitLab |

## `docs/workflow/conventions.md`

| # | Блок | Когда включается |
|---|---|---|
| 1 | `conventions/header` | всегда |
| 2 | `conventions/untouchable` | всегда |
| 3 | `conventions/tests-deferred` · `tests-critical` · `tests-strict` | по оси тестов |
| 4 | `conventions/coverage` | `tests ≠ deferred` И порог не задан в репозитории |
| 4 | `conventions/coverage-existing` | порог уже задан в настройках или назван человеком |
| 5 | `conventions/check-command` | `check ≠ none` |
| 6 | `conventions/check-levels` | `check ≠ none` |
| 7 | `conventions/migrations` | всегда — константа |
| 8 | `conventions/code` | всегда |
| 9 | `conventions/logs` | всегда |

## Правила в `AGENTS.md`

Строки раздела «Глобальные правила» набираются по осям — список в `agents-rules.md`.
