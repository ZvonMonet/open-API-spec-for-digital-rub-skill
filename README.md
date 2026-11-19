# oas-from-jira

Учебный проект: skill для ИИ-агента, который по задаче и пользовательской истории из Jira готовит **OpenAPI Specification** и предлагает её через Pull Request.

Продукт для примера — вымышленный модуль **КошелёкЦР** (кошелёк цифрового рубля для небольшого банка). Все данные в репозитории синтетические.

- Инструкции для агента — [AGENTS.md](AGENTS.md)
- Skill — [skills/oas-from-jira/SKILL.md](skills/oas-from-jira/SKILL.md)
- Знания — [knowledge/](knowledge/)
- Результаты — [artifacts/](artifacts/)
- Журнал ошибок агента — [ai-log.md](ai-log.md)

Проверка артефакта:

```bash
npx @stoplight/spectral-cli@6 lint artifacts/<имя>.yaml --ruleset .spectral.yaml
```
