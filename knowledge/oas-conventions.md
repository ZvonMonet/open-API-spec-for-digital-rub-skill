---
title: Соглашения по оформлению OpenAPI Specification
type: reference
scope: версия, структура файла, именование, общие компоненты, ошибки, типы данных, расширения, линтинг
related: [[glossary]], [[business-rules]], [[sensitive-data]]
actualized: 2026-11-19
tags: [oas, conventions, spectral]
---

# Соглашения по оформлению OAS

Единственный источник правил **оформления** спецификации. Термины — в [[glossary]], бизнес-проверки и коды ошибок — в [[business-rules]], классификация и маскирование данных — в [[sensitive-data]].

Идентификаторы правил: `OAS-<раздел>-<номер>`. Правила, которые проверяет линтер, помечены **[lint]** — им соответствует правило в `.spectral.yaml` в корне репозитория.

---

## 1. Версия и файл

| ID | Правило |
|---|---|
| OAS-FILE-01 | Версия стандарта — **OpenAPI 3.0.3** (`openapi: 3.0.3`). Причина: максимальная поддержка генераторами кода и mock-серверами. **[lint]** |
| OAS-FILE-02 | Формат — YAML, отступ 2 пробела, кодировка UTF-8. |
| OAS-FILE-03 | Один файл = один API одной фичи или группы связанных фич. Имя файла: `<продукт>-<область>-api.yaml`, например `koshelek-cr-wallet-opening-api.yaml`. |
| OAS-FILE-04 | Порядок корневых разделов: `openapi`, `info`, `servers`, `security`, `tags`, `paths`, `components`. |
| OAS-FILE-05 | Идентификаторы (пути, поля, схемы, `operationId`) — на английском. Тексты (`summary`, `description`, `title`) — на русском. |
| OAS-FILE-06 | Файл кладётся в `artifacts/<имя>.yaml`; рядом — отчёт проверки `artifacts/<имя>-check.md` по шаблону skill. |
| OAS-FILE-07 | **Только то, что следует из требований.** Каждая операция, схема и ответ прослеживаются до AC, шага сценария Task, бизнес-правила или НФТ; трассировка — в `-check.md`. Компоненты из скелета (§15), которые не используются, удаляются (Spectral: `oas3-unused-component`). |

## 2. Раздел `info`

| ID | Правило |
|---|---|
| OAS-INFO-01 | Обязательны `title`, `version`, `description`. **[lint]** |
| OAS-INFO-02 | `version` — SemVer версии API (`1.0.0`), а не версии стандарта. Новая обязательная часть контракта или удаление поля — смена MAJOR. |
| OAS-INFO-03 | `description` содержит: назначение API, потребителя, внешние системы, ссылки на задачи Jira, перечень допущений (если есть). |
| OAS-INFO-04 | Ключи задач Jira, по которым создана спецификация, — в `x-jira-keys: [SCRUM-2194, ...]`. **[lint]** |

## 3. `servers`

| ID | Правило |
|---|---|
| OAS-SRV-01 | Только `https`. **[lint]** |
| OAS-SRV-02 | Версия API — в базовом URL сервера, а не в каждом пути: `https://api.bank.example/koshelek-cr/v1`. |
| OAS-SRV-03 | Домены — вымышленные (`*.example`), без реальных адресов банка. |

## 4. Пути и операции

| ID | Правило |
|---|---|
| OAS-PATH-01 | Сегменты путей — kebab-case, ресурсы — существительные во множественном числе: `/wallet-applications`. **[lint]** |
| OAS-PATH-02 | Параметры пути — camelCase в фигурных скобках: `/wallet-applications/{applicationId}`. |
| OAS-PATH-03 | Идентификаторы в путях — только UUID. Номера счетов, телефоны и другие данные уровня C2/C3 в путях и query запрещены (см. [[sensitive-data]], SD-API-02). |
| OAS-PATH-04 | Действие, не сводимое к CRUD, — `POST` на подресурс-глагол: `POST /wallet-applications/{applicationId}/confirm`. |
| OAS-PATH-05 | Без завершающего `/`. **[lint]** |
| OAS-OP-01 | У каждой операции есть `operationId` (camelCase, глагол + сущность: `createWalletApplication`), `summary` (≤ 80 символов), `description`, `tags`. **[lint]** |
| OAS-OP-02 | У каждой операции есть `x-jira-key` — ключ задачи-источника. **[lint]** |
| OAS-OP-03 | Если операция реализует бизнес-правила — `x-business-rules: [BR-...]` с ID из [[business-rules]]. |
| OAS-OP-04 | Если операция вызывает внешние системы — `x-external-systems: [DR_PLATFORM, CORE_BANKING, ANTIFRAUD, ESIA]`. |
| OAS-OP-05 | `description` операции перечисляет: предусловия, что делает сервер, какие внешние системы вызываются, синхронно или асинхронно, ссылки на AC (`AC-01`). |
| OAS-OP-06 | `tags` — по фиче (`Wallet Opening`, `Wallet Top-up`); каждый тег описан в корневом `tags`. **[lint]** |

## 5. Схемы и поля

| ID | Правило |
|---|---|
| OAS-SCH-01 | Имена схем — PascalCase. Запрос — `<Действие><Сущность>Request` (`CreateWalletApplicationRequest`), ресурс — `<Сущность>` (`WalletApplication`), список — `<Сущность>Page`. |
| OAS-SCH-02 | Свойства — camelCase. **[lint]** |
| OAS-SCH-03 | Значения `enum` — UPPER_SNAKE_CASE. **[lint]** |
| OAS-SCH-04 | У каждого объекта явно указан `required`. |
| OAS-SCH-05 | Схемы запросов — `additionalProperties: false`. В схемах ответов не указывается (клиенты должны игнорировать новые поля). |
| OAS-SCH-06 | Поля, которые формирует сервер (`id`, `status`, `createdAt`), — `readOnly: true`. Секреты — `writeOnly: true` (см. [[sensitive-data]]). |
| OAS-SCH-07 | У каждого свойства есть `description` с бизнес-смыслом (не «строка», а «Номер счёта для списания, принадлежащий клиенту»). **[lint]** |
| OAS-SCH-08 | Все переиспользуемые схемы — в `components.schemas`; inline-схемы в операциях — только для примитивов. |
| OAS-SCH-09 | Чувствительные поля размечаются `x-sensitivity` по правилам [[sensitive-data]]. |

## 6. Типы данных

| ID | Тип | Правило | Пример |
|---|---|---|---|
| OAS-TYPE-01 | Идентификатор | `type: string`, `format: uuid`. | `3f6c1e2a-...` |
| OAS-TYPE-02 | Дата | `format: date`, ISO 8601. | `2026-11-19` |
| OAS-TYPE-03 | Дата и время | `format: date-time`, RFC 3339 **с часовым поясом**. | `2026-11-19T14:21:00+03:00` |
| OAS-TYPE-04 | Деньги | Общая схема `Money`: `value` — строка с десятичной суммой (`pattern: '^\d{1,15}(\.\d{1,2})?$'`), `currency` — ISO 4217 (`enum: [RUB]`). Числа с плавающей точкой для сумм запрещены. | `{ value: "1500.00", currency: RUB }` |
| OAS-TYPE-05 | Телефон | E.164, `pattern: '^\+7\d{10}$'`. | `+79000000001` |
| OAS-TYPE-06 | Номер счёта | `type: string`, `pattern: '^\d{20}$'` (строка, не число: ведущие нули). | |
| OAS-TYPE-07 | Коды и номера документов | Всегда строки с `pattern`, не `integer`. | |
| OAS-TYPE-08 | Статус | `enum` из статусной модели [[business-rules]] §8; значения перечисляются полностью. Статус включается **вместе с атрибутами**, которые §8 для него требует (`rejectReason` у `REJECTED`, `errorCode` у `FAILED`), и с описанием условий перехода в `description`. Если условия перехода не заданы ни в Task, ни в контексте, ни в §8 — это открытый вопрос: статус остаётся в `enum` с `x-assumption`, вопрос записывается в отчёт; статус без атрибутов не допускается (ERR-03). | |
| OAS-TYPE-09 | Маскированное значение | Отдельное поле с суффиксом `Masked` (`phoneMasked`), правило — из [[sensitive-data]] §4. | |
| OAS-TYPE-10 | Идентификатор договора (`contractId`) | `type: string`, без `format` и `pattern` (формат задаёт платформа ЦР). | `"DR-CONTRACT-2026-09"` |
| OAS-TYPE-11 | Запрос на сертификат (CSR) | PKCS#10 в кодировке DER, передаётся в Base64: `type: string`, `format: byte`, `maxLength: 8192`; `writeOnly: true`. Содержит только открытый ключ. | `MIIBTzCB/QIBADBG...` |
| OAS-TYPE-12 | Алгоритм ключа ЭП (`keyAlgorithm`) | `enum: [GOST_R_34_10_2012_256, GOST_R_34_10_2012_512]` — ГОСТ Р 34.10-2012 с длиной ключа 256 / 512 бит; хеш — ГОСТ Р 34.11-2012. | `GOST_R_34_10_2012_256` |
| OAS-TYPE-13 | Номер счёта цифрового рубля (`walletNumber`) | `type: string`, `pattern: '^0x[a-fA-F0-9]{40}$'`, **без** `format: uuid` (значение не UUID). Это реквизит платформы ЦР, не идентификатор ресурса API: в путях — `walletId` (`format: uuid`, OAS-TYPE-01, SD-API-03). В ответах по умолчанию — `walletNumberMasked` (`MASK-WALLET`), `pattern: '^0x\*{4}[a-fA-F0-9]{4}$'`. Решение 25.11.2026. | `0x5aAeb6053F3E94C9b9A09f33669435E7Ef1BeAed` → `0x****eAed` |

## 7. Заголовки

| ID | Заголовок | Где | Правило |
|---|---|---|---|
| OAS-HDR-01 | `Authorization` | Все запросы | Через `securitySchemes`, не как параметр. |
| OAS-HDR-02 | `X-Request-Id` | Все запросы и ответы | UUID, необязателен в запросе; сервер возвращает его (или сгенерированный) в ответе. Компонент `components.parameters.XRequestId`. |
| OAS-HDR-03 | `Idempotency-Key` | **Каждый** `POST`: создание ресурса, переход статуса (принятие, отмена, подтверждение), денежная операция | UUID, **обязателен**. Компонент `components.parameters.IdempotencyKey`. В `description` параметра: повтор возвращает исходный итоговый ответ; ответы 5xx не сохраняются (BR-IDM-03). **[lint]** |
| OAS-HDR-04 | `Location` | Ответы 201 и 202 | URL созданного ресурса или ресурса статуса. |
| OAS-HDR-05 | `Retry-After` | Ответы 429 и 503 | Секунды. |
| OAS-HDR-06 | `Cache-Control: no-store` | Ответы с данными C2/C3 | Компонент `components.headers.CacheControlNoStore`. |

## 8. Ответы и ошибки

| ID | Правило |
|---|---|
| OAS-ERR-01 | Все ошибки 4xx и 5xx — `application/problem+json` по RFC 9457, схема `Problem`. **[lint]** |
| OAS-ERR-02 | Схема `Problem`: `type` (URI), `title`, `status`, `detail`, `instance`, `code` (бизнес-код из каталога [[business-rules]] §9), `traceId`, `errors[]` (только для 400: `field`, `code`, `message`). |
| OAS-ERR-03 | `detail` не содержит ПДн, значений полей запроса и сведений о мерах ПОД/ФТ ([[sensitive-data]], SD-API-07). |
| OAS-ERR-04 | Каждая операция описывает минимум: успешный ответ, `400`, `401`, `500`; плюс все ответы, следующие из её бизнес-правил. **[lint: 400/401/500]** |
| OAS-ERR-05 | Типовые ответы — через `components.responses`: `BadRequest`, `Unauthorized`, `Forbidden`, `NotFound`, `Conflict`, `UnprocessableEntity`, `TooManyRequests`, `InternalServerError`, `ServiceUnavailable`, `GatewayTimeout`. |
| OAS-ERR-06 | Для `409` и `422` в описании ответа операции перечисляются конкретные `code`, которые она может вернуть. |
| OAS-ERR-07 | Чужой или несуществующий ресурс → `404` (не `403`), чтобы не раскрывать его существование. |

Использование HTTP-кодов:

| Код | Когда |
|---|---|
| 200 | Успех с телом (чтение, синхронное действие). |
| 201 | Ресурс создан синхронно; есть `Location`. |
| 202 | Принято, обработка асинхронная; тело — ресурс со статусом, есть `Location`. |
| 204 | Успех без тела. |
| 400 | Синтаксис или формат запроса (`VALIDATION_ERROR`). |
| 401 / 403 / 404 | Аутентификация / права / ресурс не найден. |
| 409 | Конфликт состояния: ресурс уже есть, недопустимый шаг для текущего статуса. |
| 422 | Запрос корректен, но нарушено бизнес-правило. |
| 429 | Превышен лимит частоты. |
| 500 | Внутренняя ошибка. |
| 502 / 503 / 504 | Ошибка, недоступность, таймаут внешней системы. |

## 9. Асинхронные операции

| ID | Правило |
|---|---|
| OAS-ASYNC-01 | Если результат зависит от внешней системы, которая может отвечать дольше НФТ по времени ответа, операция возвращает `202` с ресурсом в промежуточном статусе и `Location`. |
| OAS-ASYNC-04 | Если часть решения получена синхронно до ответа (например, отказ антифрода), `202` может вернуть ресурс сразу в итоговом статусе (`REJECTED`). Это описывается в `description` операции и примером. |
| OAS-ASYNC-02 | Клиент узнаёт результат опросом `GET` ресурса. Рекомендуемый интервал опроса — в `description`. |
| OAS-ASYNC-03 | Уведомления сервер-сервер — через `callbacks`. События «тонкие»: id, тип, статус, время (SD-API-12). |

## 10. Списки

| ID | Правило |
|---|---|
| OAS-PAGE-01 | Пагинация курсорная: query `limit` (1–100, по умолчанию 20), `cursor`. |
| OAS-PAGE-02 | Ответ: `items[]`, `nextCursor` (отсутствует на последней странице). |
| OAS-PAGE-03 | Фильтры по датам — `dateFrom`, `dateTo` (`format: date`); сортировка — `sort` (`-createdAt`). |

## 11. Безопасность

| ID | Правило |
|---|---|
| OAS-SEC-01 | Клиентские операции — `bearerAuth` (`type: http`, `scheme: bearer`, `bearerFormat: JWT`), задаётся глобально в корневом `security`. **[lint]** |
| OAS-SEC-02 | Операции бэк-офиса — отдельная схема `backofficeAuth` (`oauth2`, authorization code) со scope `backoffice:read`, `backoffice:read-pii`, `backoffice:write`; указывается на уровне операции. |
| OAS-SEC-03 | Взаимодействие с внешними системами (платформа ЦР, АБС, антифрод) в клиентской OAS не описывается как `security`; его особенности (mTLS, ГОСТ, подпись) — в `info.description` или в OAS адаптера. |

## 12. Примеры

| ID | Правило |
|---|---|
| OAS-EX-01 | У каждой операции — минимум один пример успешного ответа и один пример ошибки. |
| OAS-EX-02 | Для запросов с вариантами (`oneOf`) — пример на каждый вариант. |
| OAS-EX-03 | Только синтетические данные ([[sensitive-data]], SD-API-13): телефоны `+7900000xxxx`, вымышленные ФИО, счета вида `40817810000000000001`. |
| OAS-EX-04 | Примеры используют `examples` (именованные) в `content`, а не `example` в схеме, если вариантов больше одного. |

## 13. Расширения x-*

| Расширение | Где | Значение | Определено в |
|---|---|---|---|
| `x-jira-keys` | `info` | Список ключей задач-источников. | Здесь |
| `x-jira-key` | Операция | Ключ задачи-источника операции. | Здесь |
| `x-business-rules` | Операция, схема, свойство | ID правил из [[business-rules]]. | Здесь |
| `x-external-systems` | Операция | `DR_PLATFORM`, `CORE_BANKING`, `ANTIFRAUD`, `ESIA`. | Здесь |
| `x-acceptance-criteria` | Операция | ID критериев приёмки из Story (`AC-01`). | Здесь |
| `x-assumption` | Любой объект | Текст допущения, сделанного при нехватке данных. | Здесь |
| `x-sensitivity` | Свойство | `level`, `category`, `masking`, `log`. | [[sensitive-data]] §9 |

## 14. Линтинг (Spectral)

| ID | Правило |
|---|---|
| OAS-LINT-01 | Перед выдачей результата спецификация проверяется командой `npx @stoplight/spectral-cli@6 lint <файл> --ruleset .spectral.yaml` (ruleset проверен на Spectral 6.17). |
| OAS-LINT-02 | Ошибок (`error`) быть не должно. Предупреждения (`warn`) допустимы, если каждое объяснено в отчёте. |
| OAS-LINT-03 | Ruleset `.spectral.yaml` расширяет `spectral:oas` и добавляет правила, помеченные в этом файле **[lint]**. Изменение соглашения сопровождается изменением ruleset. |

---

## 15. Скелет файла

```yaml
openapi: 3.0.3
info:
  title: КошелёкЦР — открытие кошелька цифрового рубля
  version: 1.0.0
  description: |
    Назначение, потребитель, внешние системы, допущения.
  x-jira-keys: [SCRUM-2194]
servers:
  - url: https://api.bank.example/koshelek-cr/v1
    description: Пример окружения
security:
  - bearerAuth: []
tags:
  - name: Wallet Opening
    description: Открытие кошелька цифрового рубля
paths: {}
components:
  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
  parameters:
    XRequestId:
      name: X-Request-Id
      in: header
      required: false
      description: Сквозной идентификатор запроса для трассировки.
      schema: { type: string, format: uuid }
    IdempotencyKey:
      name: Idempotency-Key
      in: header
      required: true
      description: Ключ идемпотентности; повтор с тем же ключом возвращает исходный результат.
      schema: { type: string, format: uuid }
  headers:
    CacheControlNoStore:
      description: Ответ содержит конфиденциальные данные и не кэшируется.
      schema: { type: string, enum: [no-store] }
    Location:
      description: URL созданного ресурса или ресурса статуса.
      schema: { type: string, format: uri }
    RetryAfter:
      description: Через сколько секунд можно повторить запрос.
      schema: { type: integer, minimum: 1 }
  schemas:
    Money:
      type: object
      required: [value, currency]
      properties:
        value:
          type: string
          pattern: '^\d{1,15}(\.\d{1,2})?$'
          description: Сумма в рублях с точностью до копеек.
        currency:
          type: string
          enum: [RUB]
          description: Код валюты по ISO 4217.
    Problem:
      type: object
      required: [type, title, status, code]
      properties:
        type: { type: string, format: uri, description: URI типа ошибки. }
        title: { type: string, description: Краткое описание типа ошибки. }
        status: { type: integer, description: HTTP-код ответа. }
        detail: { type: string, description: Пояснение без ПДн и без сведений о мерах ПОД/ФТ. }
        instance: { type: string, description: Идентификатор конкретного случая ошибки. }
        code: { type: string, description: Бизнес-код ошибки из каталога. }
        traceId: { type: string, description: Идентификатор для поиска в журналах. }
        errors:
          type: array
          description: Ошибки валидации полей (только для 400).
          items:
            type: object
            required: [field, code]
            properties:
              field: { type: string, description: Путь к полю в запросе. }
              code: { type: string, description: Код ошибки валидации. }
              message: { type: string, description: Пояснение. }
  responses:
    BadRequest:
      description: Некорректный запрос.
      content:
        application/problem+json:
          schema: { $ref: '#/components/schemas/Problem' }
    Unauthorized:
      description: Требуется аутентификация.
      content:
        application/problem+json:
          schema: { $ref: '#/components/schemas/Problem' }
    InternalServerError:
      description: Внутренняя ошибка сервера.
      content:
        application/problem+json:
          schema: { $ref: '#/components/schemas/Problem' }
    # Forbidden, NotFound, Conflict, UnprocessableEntity, TooManyRequests,
    # ServiceUnavailable, GatewayTimeout — по тому же образцу (OAS-ERR-05)
```
