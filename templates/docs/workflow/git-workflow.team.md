# Рабочий процесс: git-репозиторий

> Механика работы с гитом: ветки, коммиты, PR, rebase. Режим «команда»: PR-only + обязательное ревью второго участника.
> Конвенции кода и инварианты («что нельзя трогать») → `conventions.md`. Управление задачами → `project-workflow.md`.

## ⚠️ Главное: что означает мерж в `main`

<!-- ВАРИАНТ A — мерж в main запускает автодеплой на прод. Оставить этот блок, удалить вариант B. -->
Мерж в `main` запускает автодеплой на прод. **Каждый мерж = выкат в продакшен.** Из этого следует:

- мержим **только** зелёный CI и **только** после апрува ревьюера;
- после мержа результат дополнительно проверяется **на проде**;
- PR **не** закрывает задачу автоматически (без `Closes #N`) — закрываем после проверки на проде.

<!-- ВАРИАНТ B — автодеплоя нет (или прода пока нет). Оставить этот блок, удалить вариант A. -->
`main` всегда в рабочем состоянии: собирается, проверка зелёная. Мержим только зелёный CI и только после апрува ревьюера. Деплой — отдельное осознанное действие.

`main` — **домашняя база**. Между задачами ты стоишь на свежем `main`; рабочие ветки одноразовые и короткоживущие. Не «живёшь» на фиче-ветке неделями — иначе локальный `main` устаревает и возвращаться после мержа некуда (типичная грабля).

## Ветки

Формат `{номер-задачи}-описание`, например `42-добавить-регистрацию`. Номер берётся из задачи в трекере — «нет задачи — нет работы» (исключение — тривиальные `docs`/`chore`, см. «Быстрая правка не по теме»). Если трекер — GitHub, ветку удобно создавать кнопкой `Create a branch` на странице issue.

## Коммиты

Русский язык, формат `тип: глагол + что делает`. Типы: `feat`, `fix`, `chore`, `docs`, `refactor`. Без точки в конце, не длиннее 72 символов. Примеры:
```
feat: добавляет обработчик регистрации
fix: исправляет ошибку валидации API-ключа
docs: обновляет AGENTS.md после рефакторинга
```

## Жизненный цикл ветки

```bash
# СТАРТ — всегда от свежего main
git checkout main && git pull
git checkout -b 42-краткое-описание

# ... работаем, коммитим ...

# ПЕРЕД PR — подтянуть main под себя
git fetch origin && git rebase origin/main
# rebase переписал историю → обычный push отлетит, нужен force-with-lease
git push --force-with-lease origin 42-краткое-описание

# PR: base main, в описании упоминаем задачу → запросить ревью
# апрув + зелёный CI → Merge (squash)

# ВЕРНУТЬСЯ ДОМОЙ
git checkout main && git pull
git branch -D 42-краткое-описание      # squash-мерж не виден как merge → -D, не -d
```

## Приём PR

- **PR обязателен**, мерж — после **апрува второго участника** и зелёного CI.
- Ревьюер проверяет по Definition of Done (`project-workflow.md`): критерий готовности, тесты по политике, доки/ADR обновлены, если затронута архитектура.
- Автор не мержит без апрува; ревьюер не аппрувит без обновлённых доков (правило из `docs/README.md`).
- **Метод мержа — squash** (репозиторий настроить на squash-only): один коммит на задачу, линейная история `main`.
- **Прямой push в `main` не используем** (только PR) — и закрываем технически через ruleset (см. конец файла).

## Быстрая правка не по теме (cameo fix)

Понадобилось внести мелкую правку, пока сидишь в незаконченной фиче? **Не** коммить её в текущую фичу и **не** ответвляйся от фичи. Ответвляйся **от свежего `main`**:

```bash
git stash push -- путь/к/файлу        # если уже что-то наредактировал в фиче
git fetch origin
git checkout -b docs-краткое-описание origin/main
git stash pop
# правка → коммит → PR → squash-merge
git checkout 42-краткое-описание       # вернулись в свою фичу
```

Тривиальные `docs`/`chore` можно вести без отдельной задачи — исключение из «нет задачи — нет работы».

## Откат

Сломавший мерж откатывается **тоже через PR**:

```bash
git checkout main && git pull
git checkout -b revert-<что-откатываем>
git revert <sha коммита-мержа>         # squash-мерж = один коммит, откатывается целиком
git push origin revert-<что-откатываем>
# PR → апрув (для отката — допустимо ускоренное ревью) → squash-merge
```

Не правь `main` напрямую даже в пожар. Единственное исключение — сломан сам CI и мерж невозможен: чинит владелец/админ, осознанно и руками.

## Правила (шпаргалка)

- Старт ветки — всегда от свежего `main`; между задачами стоишь на `main`.
- Перед PR — `git rebase origin/main`. Пуш rebase-нутой ветки — только `--force-with-lease`, **никогда** `--force` (затрёшь чужое).
- Длинные ветки периодически ребейзим; обязательно — перед PR.
- `merge` между ветками руками не используем — только `rebase`.
- <!-- ВАРИАНТ A: -->PR упоминает задачу, но БЕЗ `Closes #N` — закрываем после проверки на проде.<!-- ВАРИАНТ B: -->PR упоминает задачу; автозакрытие (`Closes #N`) допустимо, если проверка происходит до мержа.
- Метод мержа — **squash**; локальную ветку чистим `git branch -D`; залежи — `git fetch --prune` + удалить `[gone]`.
- Прямой push в `main` — закрыт ruleset'ом; до его включения — договорённость.
- Тривиальные `docs`/`chore` — можно без задачи; всё остальное — задача обязательна.
- Времянки → `scratch/`, логи → `logs/` (см. `conventions.md`).
- Перед PR — полная проверка одной командой (`conventions.md`); docs-only изменения CI не гоняют.

## Защита ветки + auto-merge

<!-- ВАРИАНТ GitHub — оставить этот блок, удалить блок GitLab ниже. -->
Для команды ruleset включаем при первой возможности (для приватного репо нужен платный план GitHub; для публичного — free). Выполняет владелец/админ:

**1. Repo settings — auto-merge и авто-удаление веток:**
```bash
gh api -X PATCH repos/{{OWNER}}/{{REPO}} -F allow_auto_merge=true -F delete_branch_on_merge=true
```

**2. Ruleset на `main` — PR + 1 апрув + зелёный CI:**
```bash
gh api -X POST repos/{{OWNER}}/{{REPO}}/rulesets --input - <<'EOF'
{
  "name": "main protection",
  "target": "branch",
  "enforcement": "active",
  "conditions": { "ref_name": { "include": ["~DEFAULT_BRANCH"], "exclude": [] } },
  "rules": [
    { "type": "pull_request",
      "parameters": {
        "required_approving_review_count": 1,
        "dismiss_stale_reviews_on_push": false,
        "require_code_owner_review": false,
        "require_last_push_approval": false,
        "required_review_thread_resolution": false
      }
    },
    { "type": "required_status_checks",
      "parameters": {
        "strict_required_status_checks_policy": false,
        "required_status_checks": [
          { "context": "TODO-имя-CI-check" }
        ]
      }
    },
    { "type": "non_fast_forward" }
  ]
}
EOF
```

> Нюанс docs-only: CI-workflow стоит на `paths-ignore` для `*.md`/`docs/**`, поэтому required-check в docs-only PR останется pending и заблокирует мерж. Решение — job-пустышка с тем же именем check'а на docs-only путях (`paths: ['**/*.md', 'docs/**']`), которая сразу завершается успехом.

**3. На каждом PR** — «Enable auto-merge (squash)»: PR вольётся сам после апрува и зелёного CI.

<!-- ВАРИАНТ GitLab — оставить этот блок, удалить блок GitHub выше. -->
Protected branches в GitLab бесплатны (включая self-hosted CE) — включаем сразу, выполняет владелец/админ:

1. **Settings → Merge requests:** «Squash commits when merging» = **Require**; «Pipelines must succeed» = on; «Skipped pipelines are considered successful» = on — иначе docs-only MR (пайплайн не запускается) не сможет влиться.
2. **Settings → Repository → Protected branches:** `main` — Allowed to push and merge: **No one** (push) / **Maintainers** (merge). Прямой push закрыт физически.
3. **Обязательные апрувы** (Merge request approvals, Approvals required = 1) — платная фича (Premium/Ultimate). Есть Premium → включить. На Free/CE технически не enforce'ится — держимся на договорённости: **не мержим без апрува в комментариях MR**.
4. На каждом MR — **«Merge when pipeline succeeds»** после получения апрува.
