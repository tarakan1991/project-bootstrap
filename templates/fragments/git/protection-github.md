## Защита главной ветки

Пока не включено — держимся на договорённости. Включает владелец репозитория. Для приватного репозитория нужен платный план GitHub; для публичного доступно бесплатно.

**1. Настройки репозитория — автовливание и удаление веток:**

```bash
gh api -X PATCH repos/{{OWNER}}/{{REPO}} -F allow_auto_merge=true -F delete_branch_on_merge=true
```

**2. Правило на защищаемой ветке.** Ветки окружений защищаются каждая своим правилом — ниже шаблон для одной; для прод-ветки круг вливающих делаем уже.

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
        "required_approving_review_count": {{0 или 1 — по ответу про ревью}},
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
          { "context": "TODO-имя-проверки" }
        ]
      }
    },
    { "type": "non_fast_forward" }
  ]
}
EOF
```

> Грабля с правками только документации: если проверка настроена игнорировать `*.md` и `docs/**`, то в таком предложении обязательная проверка останется в состоянии ожидания и заблокирует вливание навсегда. Решение — пустая задача с тем же именем проверки на этих путях, которая сразу завершается успехом.

**3. На каждом предложении** включать автовливание — влить само, как только условия выполнены.

После включения обновить этот раздел: договорённость стала техническим ограничением.
