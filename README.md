# oas-from-jira

Учебный проект: skill для ИИ-агента, который по задаче и пользовательской истории из Jira готовит **OpenAPI Specification** и **sequence-диаграмму PlantUML** с кодами ошибок и ретраями и предлагает их через Pull Request.

Продукт для примера — вымышленный модуль **КошелёкЦР** (кошелёк цифрового рубля для небольшого банка). Все данные в репозитории синтетические.

- Инструкции для агента — [AGENTS.md](AGENTS.md)
- Skill — [skills/oas-from-jira/SKILL.md](skills/oas-from-jira/SKILL.md)
- Знания — [knowledge/](knowledge/)
- Результаты — [artifacts/](artifacts/)
- Журнал ошибок агента — [ai-log.md](ai-log.md)

Проверка артефактов:

```bash
npx @stoplight/spectral-cli@6 lint artifacts/<имя>-api.yaml --ruleset .spectral.yaml
java -jar plantuml.jar -checkonly artifacts/<имя>-sequence.puml
```

`plantuml.jar` — из Maven Central: `https://repo1.maven.org/maven2/net/sourceforge/plantuml/plantuml/<версия>/plantuml-<версия>.jar`. Код `.puml` можно вставить в любой редактор PlantUML без доступа к сети.
