# Отчёт проверки: koshelek-cr-wallet-opening

| Параметр | Значение |
|---|---|
| OAS | `artifacts/koshelek-cr-wallet-opening-api.yaml` (OpenAPI 3.0.3, API 1.1.0) |
| Sequence-диаграмма | `artifacts/koshelek-cr-wallet-opening-sequence.puml` |
| Task | SCRUM-2194 «КошелёкЦР - открытие кошелька цифрового рубля» |
| Story | SCRUM-2196 (связь *relates to*) — единственная связанная история |
| Продукт / контекст | `knowledge/context/koshelek-cr/` |
| Подготовлено | Агентом (skill `oas-from-jira`), 2026-11-23. Черновик для проверки человеком. Артефакты подготовлены заново по текущим описаниям задач в Jira, а не правкой предыдущей версии. Исправлено человеком 2026-11-25 (`fix(human)`): ответы на открытые вопросы 1–7, ERR-02, ERR-03 (`ai-log.md`). |

## 1. Источники

| Источник | Как использован |
|---|---|
| SCRUM-2194 (Jira, Task) | Акторы, границы («Входит» / «Не входит»), предусловия, основной сценарий (шаги 1–13 с разметкой `[UI]` / `[API]` / `[EXT]`), альтернативы A1–A10, исключения E1–E4, внешние системы, ключевые данные, перечень бизнес-правил, особенности НФТ, подтверждённые решения |
| SCRUM-2196 (Jira, Story) | AC-01, AC-01a, AC-01b, AC-02 … AC-21; подтверждённые решения (срок заявки 24 ч, пароль на устройстве, ИП в том же приложении) |
| `knowledge/context/koshelek-cr/functional-requirements.md` | FR-GEN-01…06, FR-OPEN-01…13 |
| `knowledge/context/koshelek-cr/non-functional-requirements.md` | NFR-PERF-01, 02, 03, 05; NFR-AV-02, 03, 04, 05; NFR-RATE-01, 03; NFR-SEC-01, 03, 04; NFR-RET-01; NFR-OBS-01, 02; NFR-CMP-01, 02, 03 |
| `knowledge/context/koshelek-cr/integrations.md` | Политика повтора `RP-EXP`, таймауты; INT-ABS-01, INT-ESIA-01, 02, INT-DRP-01, 02, 03, 04, 08, INT-AF-01, 03; таблицы соответствия ошибок |
| `knowledge/business-rules.md` | §1.1, §1.4 (BR-IDM-01…03), §1.5, §2 (BR-AF-01…03, 06, BR-AML-01), §3 (BR-DR-OPEN-01…15), §8.1, §8.2, §9 |
| `knowledge/sensitive-data.md` | §3 (каталог полей), §4 (MASK-WALLET, MASK-HASH), §5 (SD-API-01…07, 10, 13…16), §9 (нотация `x-sensitivity`) |
| `knowledge/oas-conventions.md` | Полностью: структура, именование, заголовки, ошибки, асинхронность, расширения, скелет |

## 2. Результаты проверок

### 2.1 Spectral

```
npx @stoplight/spectral-cli@6 lint artifacts/koshelek-cr-wallet-opening-api.yaml --ruleset .spectral.yaml
Spectral 6.17.1: No results with a severity of 'hint' or higher found!
0 ошибок, 0 предупреждений
```

Ошибок и предупреждений не было ни на первом прогоне, ни после доработки (добавлено допущение A-06), ни после правок `fix(human)` 25.11.2026.

### 2.2 PlantUML

```
java -jar plantuml.jar -checkonly artifacts/koshelek-cr-wallet-opening-sequence.puml
PlantUML 1.2024.8 (jar из Maven Central), код возврата 0
```

- Рендер просмотрен: да (`-DPLANTUML_LIMIT_SIZE=16384`, PNG 1669 × 8046 px, просмотрен частями). Подписи не обрезаны, пустых веток нет. Сообщения пронумерованы `[01]`–`[89]`.
- Сверка бизнес-кодов OAS ↔ диаграмма (команда из SKILL.md, шаг 9): пустой вывод.
- Статусы из OAS в метках `[S]`: все 11 значений `WalletApplicationStatus` (`CREATED`, `ESIA_PENDING`, `CONTRACT_PENDING`, `KEYS_PENDING`, `PROCESSING`, `ON_REVIEW`, `COMPLETED`, `REJECTED`, `CANCELLED`, `EXPIRED`, `FAILED`). Каждое значение `rejectReason` — в ветке `REJECTED`, каждое значение `errorCode` — в ветке `FAILED`.
- Операции OAS на диаграмме: все 8, с тем же методом, путём и `operationId`.

## 3. Операции

| # | Метод и путь | operationId | Шаг Task | Сообщения на диаграмме |
|---|---|---|---|---|
| 1 | `POST /wallet-applications` | `createWalletApplication` | 2 | `[02]`–`[16]` |
| 2 | `POST /wallet-applications/{applicationId}/esia-authorization` | `startEsiaAuthorization` | 3 | `[17]`–`[25]` |
| 3 | `GET /wallet-applications/{applicationId}/contract` | `getWalletApplicationContract` | 6 | `[32]`–`[39]` |
| 4 | `POST /wallet-applications/{applicationId}/contract-acceptance` | `acceptWalletApplicationContract` | 8 | `[41]`–`[52]` |
| 5 | `POST /wallet-applications/{applicationId}/certificate-requests` | `submitCertificateRequest` | 10 | `[54]`–`[66]` |
| 6 | `GET /wallet-applications/{applicationId}` | `getWalletApplication` | 11 | `[72]`–`[75]` |
| 7 | `GET /wallets/{walletId}` | `getWallet` | 13 | `[77]`–`[82]` |
| 8 | `POST /wallet-applications/{applicationId}/cancel` | `cancelWalletApplication` | A6 (отказ клиента) | `[83]`–`[89]` |

Шаги `[UI]` 1, 4, 7, 9, 12 операций не дают (на диаграмме — сообщения клиента `[01]`, `[26]`, `[40]`, `[53]`, `[76]`). Шаг 5 и асинхронные результаты — служебные сообщения без клиентских операций.

## 4. Матрица покрытия AC

| AC | Элемент OAS | Элемент диаграммы | Покрыт |
|---|---|---|---|
| AC-01 | `createWalletApplication` → `201`, `WalletApplication` (`applicationId`, `ownerType`, `expiresAt`), `status: CREATED` | Шаг 2, ветка «успех», `[16]` | Да |
| AC-01a | `createWalletApplication` → `201`, `ownerType: SOLE_PROPRIETOR`; проверка дублей по типу | `[16]` (подпись «кошелёк другого типа не мешает») | Да |
| AC-01b | `createWalletApplication` → `422 CLIENT_NOT_ELIGIBLE`, `reason: NOT_SOLE_PROPRIETOR` | `[13]` | Да |
| AC-02 | `createWalletApplication` → `422 CLIENT_NOT_ELIGIBLE`, `reason: AGE / CITIZENSHIP` | `[12]` | Да |
| AC-03 | `createWalletApplication` → `409 WALLET_ALREADY_EXISTS` + `ownerType` (`WalletApplicationProblem`) | `[14]` | Да |
| AC-04 | `createWalletApplication` → `409 APPLICATION_IN_PROGRESS` + `applicationId` | `[15]` | Да |
| AC-05 | `IdempotencyKey`, `201` при повторе с тем же ключом и телом | `[05]` | Да |
| AC-06 | `startEsiaAuthorization` → `200 EsiaAuthorization.authorizationUrl`, `status: ESIA_PENDING` | `[25]` | Да |
| AC-07 | `getWalletApplication` → `REJECTED`, `rejectReason: ESIA_CONSENT_DENIED / ESIA_ACCOUNT_NOT_CONFIRMED` | Шаг 4, ветки после `[27]` и `[29]` | Да |
| AC-08 | `getWalletApplication` → `REJECTED`, `rejectReason: WALLET_ALREADY_EXISTS` | Шаг 5, ветка после `[31]` | Да |
| AC-09 | `getWalletApplicationContract` → `200 WalletContract` | `[39]` | Да |
| AC-10 | `acceptWalletApplicationContract` → `200`, `status: KEYS_PENDING`; фиксация версии и времени — в `description` | `[50]` | Да (факт фиксации — серверный, в ответ не выводится) |
| AC-11 | `acceptWalletApplicationContract` → `409 CONTRACT_VERSION_OUTDATED`; версия сверяется с ответом INT-DRP-02 в том же запросе | `[45]`, `[46]`, `[49]` | Да (исправлено, ERR-02) |
| AC-12 | `cancelWalletApplication` → `200`, `status: CANCELLED` | `[87]` | Да |
| AC-13 | `submitCertificateRequest` → `202`, `Location`, `status: PROCESSING`; `csr` `writeOnly`, закрытого ключа нет | `[66]`; note к шагу 9 | Да |
| AC-14 | `submitCertificateRequest` → `202` с `ON_REVIEW` / `REJECTED` (`OPERATION_DECLINED_BY_SECURITY`); `getWalletApplication` — переходы из `ON_REVIEW` | `[62]`, `[63]`, `[67]` и ветки после него | Да (`202` с `REJECTED` подтверждён — BR-AF-02, OAS-ASYNC-04) |
| AC-15 | `getWalletApplication` → `COMPLETED`, `walletId` | Асинхронная обработка, ветка «счёт открыт» после `[68]`; `[75]` | Да |
| AC-16 | `getWallet` → `200 Wallet` (`ownerType`, `status: ACTIVE`, `walletNumberMasked`, `balance` 0.00 RUB, `openedAt`) | `[82]` | Да |
| AC-17 | `503 ESIA_UNAVAILABLE` (`startEsiaAuthorization`), `503 PLATFORM_UNAVAILABLE` (`getWalletApplicationContract`, `acceptWalletApplicationContract`, `submitCertificateRequest`, `getWallet`), `Retry-After`, `504 UPSTREAM_TIMEOUT` (кроме `submitCertificateRequest`) | `[23]`, `[37]`, `[47]`, `[65]`, `[81]` | Да |
| AC-18 | `404 RESOURCE_NOT_FOUND` у всех операций с `applicationId` | `[18]`, `[33]`, `[42]`, `[55]`, `[74]`, `[84]` | Да |
| AC-19 | `expiresAt`, статус `EXPIRED`; `409 INVALID_APPLICATION_STATE` для шагов по истёкшей заявке | Раздел «Истечение срока заявки»; `[19]`, `[34]`, `[44]`, `[57]`, `[86]` | Да |
| AC-20 | `Idempotency-Key` у `acceptWalletApplicationContract` и `cancelWalletApplication`; повтор → сохранённый `200` | `[51]`–`[52]`, `[88]`–`[89]` | Да |
| AC-21 | `submitCertificateRequest` → `503 SERVICE_UNAVAILABLE`, `Retry-After`; заявка в `KEYS_PENDING`; повтор — новая проверка (BR-IDM-03) | `[60]`, `[61]` | Да |

## 5. Трассировка элементов

### 5.1 Операции, ответы, заголовки

| Элемент OAS | Источник |
|---|---|
| `createWalletApplication` | Шаг 2 `[API]`, FR-OPEN-01, 02; AC-01…05 |
| — `201` + `Location` | AC-01, AC-01a; OAS-HDR-04 |
| — `403 IDENTIFICATION_REQUIRED`, `CLIENT_BLOCKED` | Предусловия Task (BR-GEN-03, 04); таблица ошибок АБС в `integrations.md` |
| — `409 WALLET_ALREADY_EXISTS`, `APPLICATION_IN_PROGRESS` | A2, A3; AC-03, AC-04; BR-DR-OPEN-04, 05; §8.1 |
| — `422 CLIENT_NOT_ELIGIBLE`, `IDEMPOTENCY_KEY_REUSED` | A1, A1a; AC-01b, AC-02; BR-DR-OPEN-01, 15; BR-IDM-02 |
| — `429` + `Retry-After` | NFR-RATE-01 |
| — `503 CORE_BANKING_UNAVAILABLE` + `Retry-After`, `504 UPSTREAM_TIMEOUT` | Шаг 2 `[EXT:CORE_BANKING]`; `integrations.md` (АБС, политика повтора) |
| `startEsiaAuthorization` | Шаг 3 `[API]`, FR-OPEN-03; AC-06; имя операции — из `integrations.md` и NFR-PERF-05 |
| — `409 INVALID_APPLICATION_STATE` | §8.1 (шаг не в своём статусе); AC-19 |
| — `503 ESIA_UNAVAILABLE`, `504 UPSTREAM_TIMEOUT` | E1; AC-17; `integrations.md` (ЕСИА) |
| — повторный вызов в `ESIA_PENDING` | E2; `integrations.md` (ЕСИА: «повторный вызов в `ESIA_PENDING` возвращает новую ссылку») |
| `getWalletApplicationContract` | Шаг 6 `[API]`, FR-OPEN-05; AC-09; `503 PLATFORM_UNAVAILABLE` / `504` — E1, AC-17 |
| `acceptWalletApplicationContract` | Шаг 8 `[API]`, FR-OPEN-06; AC-10, AC-11, AC-20; `409 CONTRACT_VERSION_OUTDATED` — A7, `integrations.md` (платформа) |
| — вызов INT-DRP-02, `503 PLATFORM_UNAVAILABLE` / `504 UPSTREAM_TIMEOUT` | AC-11 (актуальная версия), FR-OPEN-06, INT-DRP-02 (решение 25.11.2026, вариант А); AC-17, E1 |
| `submitCertificateRequest` | Шаг 10 `[API]`, FR-OPEN-07…09; AC-13, AC-14, AC-21 |
| — `202` + `Location` | NFR-PERF-02, OAS-ASYNC-01; AC-13 |
| — `503 SERVICE_UNAVAILABLE` | E3; AC-21; BR-AF-06 |
| — `503 PLATFORM_UNAVAILABLE` | E1; AC-17; `integrations.md` — недоставленный запрос INT-DRP-03 (`504` не используется: неподтверждённый приём уходит в асинхронную обработку) |
| `getWalletApplication` | Шаг 11 `[API]`, FR-OPEN-10 (в т. ч. `FAILED` с `errorCode`); AC-07, 08, 14, 15, 18, 19; `429` — NFR-RATE-03; интервал опроса — NFR-PERF-03 |
| `getWallet` | Шаг 13 `[API]`, FR-OPEN-13; AC-16; `503` / `504` — INT-DRP-04, FR-GEN-05 (см. противоречие 2) |
| — `Cache-Control: no-store` | SD-API-10 (баланс и номер кошелька — C2) |
| `cancelWalletApplication` | A6; AC-12, AC-20; FR-OPEN-11; §8.1 |
| `Idempotency-Key` у всех 5 `POST` | BR-IDM-01, OAS-HDR-03, NFR-AV-04; правило Spectral `post-idempotency-key` |
| `X-Request-Id` (запрос и ответы 2xx) | NFR-OBS-01, OAS-HDR-02 |
| `404 RESOURCE_NOT_FOUND` | AC-18, BR-GEN-02, OAS-ERR-07 |
| `400`, `401`, `500` | OAS-ERR-04 |

### 5.2 Схемы и поля

| Элемент | Источник |
|---|---|
| `OwnerType` (`INDIVIDUAL`, `SOLE_PROPRIETOR`) | Task, «Границы»; FR-GEN-04; BR-DR-OPEN-05 |
| `WalletApplicationStatus` (11 значений) | §8.1; OAS-TYPE-08 (перечислять полностью, с атрибутами); `FAILED` — FR-OPEN-10, `integrations.md` (INT-DRP-03) |
| `CreateWalletApplicationRequest.ownerType` | Шаг 2; AC-01, AC-01a |
| `WalletApplication.applicationId`, `ownerType`, `status`, `rejectReason`, `errorCode`, `expiresAt`, `walletId`, `createdAt`, `updatedAt` | Task, «Ключевые данные: заявка»; AC-01, AC-07, AC-08, AC-14, AC-15, AC-19 |
| `errorCode` enum (`PLATFORM_UNAVAILABLE`, `UPSTREAM_TIMEOUT`) | §8.1 (FAILED), FR-OPEN-10; `integrations.md` (Платформа ЦР, решение 25.11.2026) |
| `rejectReason` enum | AC-07, AC-08, AC-14; A9; `integrations.md` (платформа, антифрод); BR-DR-OPEN-14, BR-AML-01 |
| `EsiaAuthorization` (`applicationId`, `status`, `authorizationUrl`) | AC-06, INT-ESIA-01 |
| `WalletContract` (`contractId`, `contractVersion`, `documentUrl`, `platformRulesUrl`, `tariffsUrl`) | AC-09; Task, «Ключевые данные: договор»; BR-DR-OPEN-08; `contractId` — OAS-TYPE-10 |
| `AcceptWalletContractRequest` (`contractVersion`, `accepted: true`) | AC-10, шаг 8; BR-DR-OPEN-07, 08 |
| `SubmitCertificateRequestRequest` (`csr`, `keyAlgorithm`) | Task, «Ключевые данные: запрос на сертификат»; AC-13; BR-DR-OPEN-10; `csr` — OAS-TYPE-11, `keyAlgorithm` — OAS-TYPE-12 |
| `Wallet` (`walletId`, `ownerType`, `status`, `walletNumberMasked`, `balance`, `openedAt`) | AC-16; Task, «Ключевые данные: кошелёк»; FR-OPEN-13; `walletNumberMasked` — OAS-TYPE-13, `MASK-WALLET` |
| `Wallet.status` enum (§8.2, 5 значений) | §8.2; OAS-TYPE-08 |
| `Money` | OAS-TYPE-04 (баланс) |
| `Problem` | OAS-ERR-01, 02 |
| `WalletApplicationProblem` (`ownerType`, `applicationId`, `reason`) | AC-03 («с указанием типа»), AC-04 («идентификатор существующей заявки»), AC-01b («причина „не является ИП“»); значения `reason` — BR-DR-OPEN-01, 15 |
| `parameters`: `ApplicationId`, `WalletId`, `IdempotencyKey`, `XRequestId` | OAS-PATH-02, 03; OAS-HDR-02, 03 |
| `headers`: `Location`, `RetryAfter`, `CacheControlNoStore`, `XRequestId` | OAS-HDR-02, 04, 05, 06 |
| `responses`: `BadRequest`, `Unauthorized`, `NotFound`, `InvalidApplicationState`, `IdempotencyKeyReused`, `TooManyRequests`, `InternalServerError`, `PlatformUnavailable`, `GatewayTimeout` | OAS-ERR-05; соответствующие операции выше |
| `examples`: `IdempotencyKeyReused`, `InvalidApplicationState` | OAS-EX-01 (переиспользуемые примеры ошибок) |

### 5.3 Элементы диаграммы

| Элемент диаграммы | Источник |
|---|---|
| Участники Клиент, Мобильное приложение, КошелёкЦР | Task, «Акторы» |
| Участники АБС, Госуслуги, Платформа ЦР, Антифрод (в `box "Внешние системы"`, порядок первого вызова: шаги 2, 3, 5, 10) | Task, «Акторы», «Внешние системы»; `integrations.md` |
| Разделы `== Шаг 1 … 13 ==` | Основной сценарий Task |
| Раздел «Асинхронная обработка» (`[67]`–`[71]`) | Шаг 10 (асинхронно); INT-AF-03, INT-DRP-03 (результат, до 3 повторных отправок), INT-DRP-08; NFR-AV-03; §8.1; FR-OPEN-10 |
| Раздел «Истечение срока заявки» | A10, AC-19, NFR-RET-01 |
| Раздел «Отмена заявки» | A6, AC-12, AC-20 |
| Ветки шага 2 | A1, A1a, A2, A3; AC-01…05; BR-GEN-03, 04; NFR-RATE-01 |
| Ветки шага 4 | A4; AC-07; `integrations.md` (ЕСИА) |
| Ветки шага 5 | A5, E2; AC-08 |
| Ветки шага 8 | A7, E1, E4; AC-10, 11, 17, 20; INT-DRP-02 |
| Ветки шага 10 | A8, A9, E1, E3; AC-13, 14, 21; BR-AF-03, 06; NFR-AV-03 |
| Ветки отмены | A6, E4; AC-12, 20 |
| `loop` опроса (`[72]`–`[75]`) и ветка `429` | NFR-PERF-03, NFR-RATE-03 |
| Note «таймаут клиента 70 с» | NFR-PERF-05 |
| Notes «закрытый ключ на устройстве», «пароль на устройстве» | BR-DR-OPEN-10, 11; FR-GEN-06; шаги 9, 12 |
| Легенда (400 / 401 / 500, `RP-EXP`, BR-IDM-03) | SKILL.md (соглашения по диаграмме); NFR-AV-05; BR-IDM-03 |

### 5.4 Чувствительные данные

| Поле | Уровень | Решение |
|---|---|---|
| `walletNumberMasked` | C2 | `MASK-WALLET` (`0x****` + 4 символа, `pattern`), `x-sensitivity`; полный номер не возвращается (SD-API-04, 05); ответ `getWallet` с `Cache-Control: no-store` |
| `balance` | C2 | Владельцу — полностью; в логах `MASK-HIDE`; `Cache-Control: no-store` |
| `csr` | C1 | `writeOnly`; в логах `MASK-HASH`; закрытый ключ в API отсутствует (BR-DR-OPEN-10) |
| `authorizationUrl` | C1 | Содержит только `state` = id заявки; код авторизации и токены ЕСИА клиенту не возвращаются (§3.3) |
| Данные ЕСИА (ФИО, СНИЛС, паспорт) | C2 / C3 | В клиентский API не входят (SD-API-16); на диаграмме — только категориями `[29]` |
| Решение антифрода | C3 | Клиенту — только общий `OPERATION_DECLINED_BY_SECURITY` (BR-AF-03, SD-API-15); на диаграмме — только `ALLOW` / `REVIEW` / `DENY` |
| Пароль к кошельку | C3 | В API не передаётся (BR-DR-OPEN-11) |
| Идентификаторы в путях | C1 | Только UUID (OAS-PATH-03, SD-API-02); C2/C3 в query нет |
| Примеры | — | Синтетические: вымышленные UUID, домены `*.example`, номер `0x****eAed` |

## 6. Ретраи и таймауты

| Вызов | Таймаут | Повторы | Исход после исчерпания / по таймауту | Источник | Сообщения `[NN]` |
|---|---|---|---|---|---|
| INT-ABS-01 (шаг 2) | 1 с | `RP-EXP`, 3 попытки | `503 CORE_BANKING_UNAVAILABLE` / `504 UPSTREAM_TIMEOUT`, заявка не создаётся; повтор клиентом с тем же ключом выполняется заново. Сведения об ИП сохраняются в заявке для шага 10 (FR-OPEN-09) | `integrations.md` (АБС, политика повтора); BR-IDM-03 | `[06]`–`[09]` |
| INT-ESIA-01 (шаг 3) | 20 с | `RP-EXP`, 3 попытки | `503 ESIA_UNAVAILABLE` / `504 UPSTREAM_TIMEOUT`, статус без изменений | `integrations.md` (ЕСИА); E1; AC-17; NFR-PERF-05 (70 с у клиента) | `[21]`–`[24]` |
| INT-ESIA-02 (шаг 4, служебный) | 20 с | `RP-EXP`, 3 попытки | Служебный обработчик → 500; заявка в `ESIA_PENDING`; клиент заново начинает шаг 3 | `integrations.md` (ЕСИА) | `[28]`–`[29]` |
| INT-DRP-01 (шаг 5, служебный) | 1,5 с | `RP-EXP`, 3 попытки | Служебный обработчик → 500; заявка в `ESIA_PENDING` | E2; `integrations.md` (платформа) | `[30]`–`[31]` |
| INT-DRP-02 (шаг 6) | 1,5 с | `RP-EXP`, 3 попытки | `503 PLATFORM_UNAVAILABLE` / `504 UPSTREAM_TIMEOUT`, статус без изменений | E1; AC-17; `integrations.md` | `[35]`–`[38]` |
| INT-DRP-02 (шаг 8) | 1,5 с | `RP-EXP`, 3 попытки | `503 PLATFORM_UNAVAILABLE` / `504 UPSTREAM_TIMEOUT`, `CONTRACT_PENDING`; повтор клиентом с тем же ключом — версия проверяется заново | AC-11, AC-17, E1; FR-OPEN-06; BR-IDM-03 | `[45]`–`[48]` |
| INT-AF-01 (шаг 10) | 500 мс | `RP-EXP`, 3 попытки | `503 SERVICE_UNAVAILABLE`, `KEYS_PENDING`; ни ручной проверки, ни отказа; повтор клиентом — новая проверка | E3; AC-21; BR-AF-06; BR-IDM-03 | `[58]`–`[61]` |
| INT-DRP-03 (шаг 10, отправка) | 1,5 с на приём | Без `RP-EXP` | Не доставлен — `503 PLATFORM_UNAVAILABLE`, `KEYS_PENDING`; отправлен — `202`, `PROCESSING` | AC-17, E1; `integrations.md` (политика повтора; недоставленный и неподтверждённый запрос) | `[64]`–`[66]` |
| INT-DRP-08 (асинхронно: приём не подтверждён или нет результата за 30 с) | 1,5 с | `RP-EXP`, 3 попытки | Повторы исчерпаны — `FAILED` (`PLATFORM_UNAVAILABLE`) | `integrations.md` (Платформа ЦР, решение 25.11.2026); NFR-AV-03; FR-OPEN-10 | `[69]`–`[70]` |
| Повторная отправка INT-DRP-03 (асинхронно) | 1,5 с на приём | Только после INT-DRP-08 «не получена», не более 3 раз, тот же id операции | Исчерпаны — `FAILED` (`PLATFORM_UNAVAILABLE`); результат не уточнён за время ожидания — `FAILED` (`UPSTREAM_TIMEOUT`) | `integrations.md` (Платформа ЦР, решение 25.11.2026); NFR-AV-03; FR-OPEN-10 | `[71]` |
| INT-DRP-04 (шаг 13) | 1,5 с | `RP-EXP`, 3 попытки | `503 PLATFORM_UNAVAILABLE` / `504 UPSTREAM_TIMEOUT` | `integrations.md`; AC-17 | `[79]`–`[81]` |
| Опрос `getWalletApplication` (шаг 11) | — | Каждые 2 с, не дольше 60 с; `429` чаще 1 раза в секунду | Через 60 с — «заявка в обработке» | NFR-PERF-03, NFR-RATE-03 | `[72]`–`[75]` |
| Повтор клиентом `acceptWalletApplicationContract` / `cancelWalletApplication` после таймаута | — | Тот же `Idempotency-Key` | Сохранённый `200`, не `409` | E4; AC-20; BR-IDM-01 | `[51]`–`[52]`, `[88]`–`[89]` |

## 7. Допущения

Действующих допущений нет.

Сняты 25.11.2026 по ответам владельца (правка `fix(human)`): A-01 (версия договора — запросом INT-DRP-02 в момент принятия; ERR-02), A-02 (сведения об ИП — из заявки, шаг 2), A-03 (`202` с `REJECTED` допустим), A-04 (до 3 повторных отправок; INT-DRP-08 недоступна — `FAILED`), A-05 (форматы: OAS-TYPE-10…13, `MASK-WALLET`), A-06 (условия `FAILED` и `errorCode`; ERR-03), A-07 (разграничение `503` / `FAILED` на шаге 10 — подтверждено, стало правилом в `integrations.md`).

## 8. Противоречия между источниками

| # | Противоречие | Решение |
|---|---|---|
| 1 | AC-14: «Дано заявка в статусе `PROCESSING` → антифрод → `ON_REVIEW` / `REJECTED`» и §8.1 (`KEYS_PENDING` → `PROCESSING` → `ON_REVIEW`) vs E3 / AC-21: при недоступности антифрода заявка остаётся в `KEYS_PENDING`, т. е. проверка выполняется до перехода в `PROCESSING` в рамках запроса. | Антифрод вызывается синхронно в `submitCertificateRequest`; при недоступности — `503`, `KEYS_PENDING`; при решении — заявка в `PROCESSING` / `ON_REVIEW` / `REJECTED` в ответе `202` (подтверждено 25.11.2026, BR-AF-02). Порядок переходов §8.1 сохраняется логически. |
| 2 | Task, «Внешние системы»: платформа ЦР — шаги 5, 6, 10 (шаг 13 не указан) vs FR-OPEN-13 / `integrations.md` (INT-DRP-04 используется в FR-OPEN-13) и FR-GEN-05 (баланс — данные платформы). | `getWallet` вызывает INT-DRP-04; у операции `503 PLATFORM_UNAVAILABLE` / `504 UPSTREAM_TIMEOUT` (AC-17 покрывает «шаг, требующий обращения к платформе»). |
| 3 | `business-rules.md` BR-DR-OPEN-02 и §9: `ESIA_CONSENT_DENIED` / `ESIA_ACCOUNT_NOT_CONFIRMED` — HTTP 422 vs AC-07, A4, `integrations.md` (ЕСИА): результат влияет на статус заявки, а не на HTTP-ответ. | По приоритету AC: статус `REJECTED` с `rejectReason`; HTTP 422 с этими кодами не используется. |
| 4 | BR-DR-OPEN-04: проверка по данным платформы → HTTP `409 WALLET_ALREADY_EXISTS` vs AC-08 / A5: при проверке на платформе — статус `REJECTED` с причиной. | На шаге 2 (кошелёк открыт через наш банк) — `409 WALLET_ALREADY_EXISTS` (AC-03); на шаге 5 (платформа) — `REJECTED` / `WALLET_ALREADY_EXISTS` (AC-08). |
| 5 | BR-DR-OPEN-10 (`422 CERTIFICATE_ISSUE_FAILED`), BR-DR-OPEN-14 / BR-AML-01 (`422 WALLET_OPENING_REFUSED`) vs шаг 10 асинхронный, `integrations.md`: отказы платформы — статус `REJECTED`. | `rejectReason: CERTIFICATE_ISSUE_FAILED` / `WALLET_OPENING_REFUSED` у заявки; HTTP 422 с этими кодами не используется. |
| 6 | Task, шаг 10: «сведения об ИП из АБС» vs решение 25.11.2026: сведения получены на шаге 2 и хранятся в заявке. | АБС на шаге 10 не вызывается, `503 CORE_BANKING_UNAVAILABLE` у `submitCertificateRequest` нет (FR-OPEN-09). |
| 7 | AC-02 не требует причины в отказе `CLIENT_NOT_ELIGIBLE`, а BR-DR-OPEN-01 («В OAS») задаёт `reason: AGE / CITIZENSHIP / NOT_SOLE_PROPRIETOR`; AC-01b требует причину «не является ИП». | Поле `reason` введено по AC-01b; значения `AGE`, `CITIZENSHIP` — по BR-DR-OPEN-01 (противоречия с AC нет, но AC их прямо не требует — решение 25.11.2026: `reason` возвращается всегда, AC-02 не меняется). |
| 8 | AC-11 требует обнаружить новую версию договора, а шаг 8 в Task не размечен `[EXT:DR_PLATFORM]`. | По приоритету AC и решению 25.11.2026 (вариант А) — вызов INT-DRP-02 в запросе принятия; 25.11.2026 владелец добавил в шаг 8 SCRUM-2194 `[EXT:DR_PLATFORM]`, в E1 — шаг 8 (ERR-02). |
| 9 | E1 / AC-17 (`503`, статус не меняется) vs решение 25.11.2026 (`FAILED` при исчерпании повторных отправок INT-DRP-03) на шаге 10. | Разграничено по доставке запроса: недоставленный — `503`, `KEYS_PENDING`; отправленный без подтверждения — `202`, `PROCESSING`, далее до `FAILED` (решение 25.11.2026, `integrations.md`). |

## 9. Новые бизнес-коды

Нет. Все коды в OAS и на диаграмме — из каталога `knowledge/business-rules.md` §9. Значения `reason` (`AGE`, `CITIZENSHIP`, `NOT_SOLE_PROPRIETOR`) — из BR-DR-OPEN-01, 15.

## 10. Открытые вопросы

Нет.

Закрыты 25.11.2026 (ответы владельца продукта): условия `FAILED` и `errorCode`; повторные отправки INT-DRP-03 (до 3) и недоступность INT-DRP-08; сведения об ИП из заявки; `202` с `REJECTED`; проверка версии договора запросом к платформе; форматы `walletNumber`, `contractId`, CSR, `keyAlgorithm`; `reason: AGE / CITIZENSHIP` возвращается всегда; разграничение `503` / `FAILED` на шаге 10.

## 11. Проверка человеком

<!-- Заполняет проверяющий -->
