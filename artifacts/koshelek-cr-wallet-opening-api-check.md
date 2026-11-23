# Отчёт проверки: koshelek-cr-wallet-opening

| Параметр | Значение |
|---|---|
| OAS | `artifacts/koshelek-cr-wallet-opening-api.yaml` (OpenAPI 3.0.3, API 1.0.0) |
| Sequence-диаграмма | `artifacts/koshelek-cr-wallet-opening-sequence.puml` |
| Task | SCRUM-2194 «КошелёкЦР - открытие кошелька цифрового рубля» |
| Story | SCRUM-2196 (связь *relates to*, единственная) |
| Продукт / контекст | `knowledge/context/koshelek-cr/` |
| Подготовлено | Агентом (skill `oas-from-jira`): OAS — 2026-11-19, диаграмма и исправление OAS — 2026-11-23. Черновик для проверки человеком. |

## 0. Изменения OAS в этой версии

Диаграмма строилась по OAS из `main`. При трассировке шага 10 обнаружилось, что описание `submitCertificateRequest` противоречит источникам: в OAS антифрод был асинхронным и шёл **после** передачи запроса на платформу. Task же (таблица «Внешние системы») требует синхронный антифрод, а последовательность в `integrations.md` задаёт порядок «антифрод → платформа → 202». По SKILL.md сначала исправляется OAS, потом рисуется диаграмма.

Что изменено в YAML (путь, схемы и коды ответов не менялись):
- `submitCertificateRequest.description`: синхронно, до ответа 202, — АБС (для ИП), затем антифрод (таймаут 500 мс; при таймауте результат считается `REVIEW`), затем при `ALLOW` передача запроса на платформу. Асинхронно — только результат платформы.
- Ответ `202` содержит заявку в статусе по решению антифрода: `PROCESSING`, `ON_REVIEW` или `REJECTED (OPERATION_DECLINED_BY_SECURITY)`. Добавлены примеры `onReview` и `rejectedSecurity`.
- `getWalletApplication.description`: переход `PROCESSING → ON_REVIEW` (теперь он происходит внутри `submitCertificateRequest`) заменён на `ON_REVIEW → PROCESSING` (ручная проверка пройдена).
- `x-assumption` операции и A-04 в `info.description` приведены к новому порядку.

## 1. Источники

| Источник | Как использован |
|---|---|
| SCRUM-2194 (Jira, Atlassian MCP; перечитана 2026-11-23, без изменений) | Акторы, границы, сценарий `[UI]` / `[API]` / `[EXT]`, альтернативы A1–A10, E1, внешние системы, НФТ |
| SCRUM-2196 (Jira, Atlassian MCP; без изменений) | AC-01…AC-19, подтверждённые решения |
| `knowledge/context/koshelek-cr/functional-requirements.md` | FR-GEN-*, FR-OPEN-01…13 |
| `knowledge/context/koshelek-cr/non-functional-requirements.md` | NFR-PERF-01…03, NFR-AV-02…04, NFR-RATE-01, 03, NFR-RET-01, NFR-OBS-01, 02 |
| `knowledge/context/koshelek-cr/integrations.md` | INT-ABS-01, INT-ESIA-01/02, INT-DRP-01…04, 08, INT-AF-01, 03; таймауты, политика повтора, таблицы соответствия ошибок, последовательность «Открытие кошелька» |
| `knowledge/business-rules.md` | BR-* из Task, статусные модели §8.1, §8.2, каталог кодов §9 |
| `knowledge/sensitive-data.md` | §3, §4 (MASK-WALLET), §5 (SD-API-*), §9 |
| `knowledge/oas-conventions.md` | Оформление OAS |

## 2. Результаты проверок

### 2.1 Spectral

```
npx @stoplight/spectral-cli@6 lint artifacts/koshelek-cr-wallet-opening-api.yaml --ruleset .spectral.yaml
Spectral 6.17.1 — 0 errors, 0 warnings
```

`.spectral.yaml` не менялся. В первой версии (2026-11-19) была исправлена одна ошибка `oas3-valid-media-example`: в ответе 422 `createWalletApplication` `oneOf` заменён на `anyOf`.

### 2.2 PlantUML

```
java -jar plantuml-1.2024.7.jar -checkonly artifacts/koshelek-cr-wallet-opening-sequence.puml
код возврата 0
```

- Рендер просмотрен (PNG 1896 × 7280 px): подписи читаются, пустых веток нет. Высота больше лимита PNG по умолчанию (4096 px), поэтому рендер запускался с `-DPLANTUML_LIMIT_SIZE=16384`; в начале файла есть комментарий об этом.
- Сверка бизнес-кодов OAS ↔ диаграмма: пустой вывод.
- Статусы из OAS в метках `[S]`: все 11 (`CREATED`, `ESIA_PENDING`, `CONTRACT_PENDING`, `KEYS_PENDING`, `PROCESSING`, `ON_REVIEW`, `COMPLETED`, `REJECTED`, `CANCELLED`, `EXPIRED`, `FAILED`).
- Все 6 значений `rejectReason` есть в ветках `REJECTED`.
- Операции OAS на диаграмме: все 8.

## 3. Операции

| # | Метод и путь | operationId | Шаг Task | Сообщения на диаграмме |
|---|---|---|---|---|
| 1 | `POST /wallet-applications` | `createWalletApplication` | 2 | `[02]`–`[14]` |
| 2 | `GET /wallet-applications/{applicationId}` | `getWalletApplication` | 11 | `[35]`–`[38]`, `[90]`–`[93]` |
| 3 | `POST /wallet-applications/{applicationId}/esia-authorization` | `createEsiaAuthorization` | 3 | `[15]`–`[24]` |
| 4 | `GET /wallet-applications/{applicationId}/contract` | `getWalletApplicationContract` | 6 | `[39]`–`[47]` |
| 5 | `POST /wallet-applications/{applicationId}/contract-acceptance` | `acceptWalletApplicationContract` | 8 | `[49]`–`[58]` |
| 6 | `POST /wallet-applications/{applicationId}/certificate-requests` | `submitCertificateRequest` | 10 | `[60]`–`[79]` |
| 7 | `POST /wallet-applications/{applicationId}/cancel` | `cancelWalletApplication` | A6 / AC-12 | `[102]`–`[105]` |
| 8 | `GET /wallets/{walletId}` | `getWallet` | 13 | `[95]`–`[101]` |

Шаги `[UI]` 1, 4, 7, 9, 12 операций OAS не дают; на диаграмме они показаны сообщениями без вызова модуля (`[01]`, `[25]`–`[26]`, `[48]`, `[59]`, `[94]`). Шаг 5 выполняет сервер, он показан служебным вызовом `[30]`–`[34]`.

## 4. Матрица покрытия AC

| AC | Элемент OAS | Элемент диаграммы | Покрыт |
|---|---|---|---|
| AC-01 | `createWalletApplication` → 201, `WalletApplication` (`id`, `ownerType`, `CREATED`, `expiresAt`) | Шаг 2, `[14]` | Да |
| AC-01a | то же, `ownerType=SOLE_PROPRIETOR`; кошелёк другого типа не мешает | Шаг 2, `[14]` | Да |
| AC-01b | 422 `CLIENT_NOT_ELIGIBLE`, `reason=NOT_SOLE_PROPRIETOR` | `[11]` | Да |
| AC-02 | 422 `CLIENT_NOT_ELIGIBLE`, `reason=AGE` / `CITIZENSHIP` | `[10]` | Да |
| AC-03 | 409 `WALLET_ALREADY_EXISTS` + `ownerType` | `[12]` | Да |
| AC-04 | 409 `APPLICATION_IN_PROGRESS` + `applicationId` | `[13]` | Да |
| AC-05 | `Idempotency-Key`; повтор → 201 та же заявка; другое тело → 422 `IDEMPOTENCY_KEY_REUSED` | `[04]`, `[05]` | Да |
| AC-06 | `createEsiaAuthorization` → 200 `authorizationUrl`, `ESIA_PENDING` | `[19]`, `[23]`–`[24]` | Да |
| AC-07 | `getWalletApplication`: `REJECTED` (`ESIA_CONSENT_DENIED` / `ESIA_ACCOUNT_NOT_CONFIRMED`) | `[27]`–`[29]`, `[38]` | Да |
| AC-08 | `getWalletApplication`: `REJECTED (WALLET_ALREADY_EXISTS)` | `[30]`–`[32]`, `[38]` | Да |
| AC-09 | `getWalletApplicationContract` → 200 `WalletContract` | `[42]`, `[46]`–`[47]` | Да |
| AC-10 | `acceptWalletApplicationContract` → 200, `KEYS_PENDING`, `acceptedContractVersion`, `contractAcceptedAt` | `[57]`–`[58]` | Да |
| AC-11 | 409 `CONTRACT_VERSION_OUTDATED` | `[55]`–`[56]` | Да |
| AC-12 | `cancelWalletApplication` → 200 `CANCELLED`; 409 после передачи CSR | `[102]`–`[105]` | Да |
| AC-13 | `submitCertificateRequest` → 202 + `Location`, `PROCESSING`; только `csr` (`writeOnly`) и `keyAlgorithm` | `[59]`–`[60]`, `[73]`–`[79]` | Да |
| AC-14 | 202 с `ON_REVIEW` / `REJECTED (OPERATION_DECLINED_BY_SECURITY)`; дальнейшие переходы — `getWalletApplication` | `[68]`–`[72]`, `[80]`–`[83]` | Да |
| AC-15 | `getWalletApplication`: `COMPLETED`, `walletId`; сведения об ИП передаются на платформу (внутренний вызов) | `[64]`–`[65]`, `[74]`, `[85]`–`[86]`, `[93]` | Да |
| AC-16 | `getWallet` → 200 `Wallet` (`ACTIVE`, `walletNumberMasked`, `balance` 0.00 RUB, `openedAt`), `no-store` | `[97]`, `[100]`–`[101]` | Да |
| AC-17 | 503 / 504 у операций с внешними вызовами; статус заявки не меняется | `[08]`, `[21]`, `[44]`, `[54]`, `[66]`, `[76]`, `[99]` | Да |
| AC-18 | 404 `RESOURCE_NOT_FOUND` у операций с `applicationId` | `[16]`, `[37]`, `[40]`, `[50]`, `[61]`, `[92]`, `[103]` | Да |
| AC-19 | `expiresAt`, `EXPIRED`; дальнейшие шаги → 409 `INVALID_APPLICATION_STATE` | `[106]`, `[17]` | Да |

## 5. Трассировка элементов

### 5.1 Операции, ответы, заголовки

| Элемент OAS | Источник |
|---|---|
| `createWalletApplication` | Task шаг 2; AC-01…05; FR-OPEN-01, 02 |
| ↳ 403 `IDENTIFICATION_REQUIRED` / `CLIENT_BLOCKED` | Task, предусловия (BR-GEN-03, 04); integrations: ошибки АБС |
| ↳ 409, 422 | AC-01b, 02, 03, 04, 05; BR-IDM-02 |
| ↳ 429 + `Retry-After` | NFR-RATE-01 |
| ↳ 503 / 504 | INT-ABS-01; integrations: `CORE_BANKING_UNAVAILABLE` / `UPSTREAM_TIMEOUT`; AC-17 |
| ↳ `Idempotency-Key`, 201 + `Location` | AC-05; BR-IDM-01; NFR-AV-04; OAS-HDR-04 |
| `getWalletApplication` | Task шаг 11; AC-07, 08, 14, 15, 18, 19; NFR-PERF-03 |
| ↳ 429 | NFR-RATE-03 |
| `createEsiaAuthorization` | Task шаг 3; AC-06; INT-ESIA-01 |
| ↳ `Idempotency-Key`, 422 | Task E1; BR-IDM-02 |
| ↳ 503 / 504 | Task E1; integrations (ЕСИА → `PLATFORM_UNAVAILABLE`); AC-17 |
| `getWalletApplicationContract` | Task шаг 6; AC-09; INT-DRP-02 |
| `acceptWalletApplicationContract` | Task шаг 8; AC-10, 11; BR-DR-OPEN-07, 08, 09 |
| ↳ 503 / 504 | Допущение A-03 |
| `submitCertificateRequest` | Task шаг 10; AC-13…15; INT-ABS-01, INT-AF-01, INT-DRP-03 |
| ↳ 202 + `Location`; статус по решению антифрода | NFR-PERF-02; OAS-ASYNC-01; Task (антифрод синхронно); AC-14 |
| ↳ `Idempotency-Key`, 503 / 504 | Task E1; AC-17 |
| `cancelWalletApplication` | AC-12; Task A6; business-rules §8.1 |
| `getWallet` | Task шаг 13; AC-16; INT-DRP-04; SD-API-10 (`no-store`) |
| 400 / 401 / 500 у всех операций | OAS-ERR-04 |
| `X-Request-Id` | OAS-HDR-02; NFR-OBS-01 |

### 5.2 Схемы и поля

| Элемент | Источник |
|---|---|
| `OwnerType` | Task, границы; BR-DR-OPEN-05 |
| `WalletApplicationStatus` (11 значений) | business-rules §8.1 (OAS-TYPE-08) |
| `WalletApplication.rejectReason` | AC-07, 08, 14; Task A9; integrations (ошибки платформы) |
| `WalletApplication.walletId` | AC-15 |
| `WalletApplication.acceptedContractVersion`, `contractAcceptedAt` | AC-10 |
| `WalletApplication.expiresAt` | AC-01, AC-19; NFR-RET-01 |
| `EsiaAuthorization` | AC-06 |
| `WalletContract` | AC-09 |
| `AcceptWalletContractRequest` (`accepted: enum [true]`) | AC-10; BR-DR-OPEN-07, 08 |
| `SubmitCertificateRequestRequest` | Task, ключевые данные; AC-13; BR-DR-OPEN-10 |
| `Wallet`, `Money` | AC-16; business-rules §8.2; OAS-TYPE-04 |
| `Problem`, `ClientNotEligibleProblem`, `WalletAlreadyExistsProblem`, `ApplicationInProgressProblem` | OAS-ERR-01, 02; AC-01b, 02, 03, 04; BR-DR-OPEN-01 |

### 5.3 Элементы диаграммы

| Элемент диаграммы | Источник |
|---|---|
| Участники: Клиент, Мобильное приложение, КошелёкЦР | Task, «Акторы» |
| Участники: АБС, Госуслуги, Платформа ЦР, Антифрод (в порядке первого вызова: шаги 2, 3, 5, 10) | Task, «Внешние системы»; коды — `integrations.md` |
| Разделы `== Шаг N ==` | Task, «Основной сценарий» |
| `[25]`–`[27]` переход в ЕСИА и служебный результат | Task шаг 4; INT-ESIA-02 (служебный endpoint вне клиентского API) |
| `[30]`–`[34]` проверка счёта на платформе | Task шаг 5; INT-DRP-01; A5 |
| `[68]`–`[73]` решение антифрода | Task шаг 10, A8; INT-AF-01; integrations «Антифрод» (`DENY` → `REJECTED`, `REVIEW` → `ON_REVIEW`) |
| `[80]`–`[83]` решение ручной проверки | INT-AF-03; business-rules §8.1 (`ON_REVIEW → PROCESSING / REJECTED`) |
| `[84]` уточнение статуса на платформе | NFR-AV-03; integrations (таймаут асинхронной операции, INT-DRP-08) |
| `[85]`–`[89]` результат платформы | INT-DRP-03; A9; integrations (ошибки платформы); §8.1 (`FAILED`) |
| `[106]` истечение заявки | Task A10; AC-19; NFR-RET-01 |
| Легенда (400 / 401 / 500) | SKILL.md, соглашения по диаграмме |

### 5.4 Чувствительные данные

| Поле | Уровень | Решение |
|---|---|---|
| `Wallet.walletNumberMasked` | C2, BANK_SECRECY | `MASK-WALLET`; полный номер не передаётся |
| `Wallet.balance` | C2, BANK_SECRECY | Владельцу полностью, в логах `MASK-HIDE`; ответ с `Cache-Control: no-store` |
| `SubmitCertificateRequestRequest.csr` | C1 | `writeOnly` |
| Данные и токены ЕСИА | C3 | В API и на диаграмме не показаны (SD-API-16) |
| Закрытый ключ ЭП, пароль кошелька | C3 | В API отсутствуют; на диаграмме — только шаги `[UI]` на устройстве |
| Детали антифрода | C3 | На диаграмме только решение `ALLOW` / `REVIEW` / `DENY` (BR-AF-03) |

## 6. Ретраи и таймауты

| Вызов | Таймаут | Повторы | Исход после исчерпания / по таймауту | Источник | Сообщения |
|---|---|---|---|---|---|
| INT-ABS-01 Профиль клиента (шаг 2) | 1 с | 1 повтор (только чтение) | 503 `CORE_BANKING_UNAVAILABLE` / 504 `UPSTREAM_TIMEOUT`; заявка не создаётся | integrations «АБС»; AC-17 | `[06]`–`[08]` |
| INT-ABS-01 Сведения об ИП (шаг 10) | 1 с | 1 повтор (только чтение) | 503 / 504; заявка остаётся `KEYS_PENDING` | integrations «АБС»; E1 | `[64]`–`[66]` |
| INT-ESIA-01 Запрос авторизации | не задан | без повтора (A-06) | 503 `PLATFORM_UNAVAILABLE` / 504; заявка остаётся `CREATED` | integrations «ЕСИА» (503); E1 | `[19]`–`[21]` |
| INT-DRP-01 Проверка наличия счёта | 1,5 с | без повтора (A-06) | не задано — открытый вопрос 10 | integrations «Платформа ЦР» | `[30]` |
| INT-DRP-02 Договор (шаги 6, 8) | 1,5 с | без повтора (A-06) | 503 / 504; статус не меняется | integrations; E1; AC-17 | `[42]`–`[44]`, `[52]`–`[54]` |
| INT-AF-01 Оценка заявки | 500 мс | без повтора | результат считается `REVIEW` → 202 `ON_REVIEW` | integrations «Антифрод» | `[68]`, `[71]`–`[72]` |
| INT-DRP-03 Открытие счёта: подтверждение приёма | 1,5 с | без повтора (A-06) | 503 / 504; заявка остаётся `KEYS_PENDING` | integrations; E1; AC-17 | `[74]`–`[76]` |
| INT-DRP-03 Открытие счёта: результат | 30 с | повторная отправка запрещена | уточнение статуса INT-DRP-08; заявка остаётся `PROCESSING` | integrations; NFR-AV-03 | `[84]` |
| INT-DRP-04 Данные кошелька | 1,5 с | без повтора (A-06) | 503 / 504 | integrations; AC-17 | `[97]`–`[99]` |
| Повтор клиентом после 503 / 504 (`createWalletApplication`, `createEsiaAuthorization`, `submitCertificateRequest`) | — | с тем же `Idempotency-Key`, в течение 24 ч | исходный результат | Task E1; NFR-AV-04 | `[08]` (note), `[22]`, `[67]`, `[77]` |
| Повтор клиентом `getWalletApplicationContract` | — | обычный повтор (GET) | — | Task E1 | `[45]` |
| Опрос статуса заявки | — | каждые 2 с, не дольше 60 с; не чаще 1 раза в секунду | 429 `TOO_MANY_REQUESTS`; через 60 с — «заявка в обработке» | NFR-PERF-03; NFR-RATE-03 | `[35]`–`[38]`, `[90]`–`[93]` |

## 7. Допущения

| ID | Допущение | Где отражено |
|---|---|---|
| A-01 | ИП открывает кошелёк для предпринимательской деятельности в том же мобильном приложении (Story, Task). | OAS `info.description` |
| A-02 | CSR — PKCS#10 DER в base64 (`format: byte`, до 8192 символов); алгоритмы `GOST_R_34_10_2012_256`, `GOST_R_34_10_2012_512`. | OAS `x-assumption` у `csr`, `keyAlgorithm` |
| A-03 | При принятии договора версия сверяется с актуальной на платформе ЦР (отсюда 503 / 504). | OAS `x-assumption`; диаграмма, note у `[52]` |
| A-04 | В `submitCertificateRequest` до ответа 202 синхронно выполняются: АБС (для ИП), антифрод, передача запроса на платформу. Асинхронно — только результат платформы. | OAS `x-assumption`, `info.description`; диаграмма, шаг 10 |
| A-05 | Маска номера кошелька — `**** XXXX`. | OAS `x-assumption` у `walletNumberMasked` |
| A-06 | Для ЕСИА и синхронных операций платформы в `integrations.md` политика повтора не задана (для ЕСИА не задан и таймаут) — принято «без повтора» (SKILL.md, шаг 3). | Диаграмма, notes у `[19]`, `[30]`, `[42]`, `[52]`, `[74]`, `[97]` |
| A-07 | Повтор `createWalletApplication` с тем же ключом возвращает 201 с той же заявкой. | OAS description; диаграмма `[04]` |
| A-08 | Договор — подресурс-синглтон `/contract`, принятие — `/contract-acceptance`, отмена — `POST …/cancel`. | Пути OAS |

## 8. Противоречия между источниками

| # | Противоречие | Решение |
|---|---|---|
| 1 | Task E1 называет недоступность внешних систем только для шагов 3, 6, 10, но шаг 2 вызывает АБС, а шаг 13 — платформу (integrations.md). | 503 / 504 есть и у `createWalletApplication`, и у `getWallet` (правило SKILL.md). |
| 2 | Task: шаг 8 к платформе не обращается; integrations.md: «Версия договора неактуальна» — результат платформы, 409 при принятии. | Сверка на платформе (A-03); открытый вопрос 2. |
| 3 | В статусной модели §8.1 есть `FAILED` с полем `errorCode`; в Task и Story этого статуса нет. | `FAILED` включён в enum (OAS-TYPE-08), на диаграмме — `[89]`. `errorCode` не добавлен. |
| 4 | Task, AC-14 и integrations.md по-разному описывают момент проверки антифрода: AC-14 — «для заявки в `PROCESSING`», Task — синхронно, integrations — «АФ → платформа → 202». | **Исправлено в этой версии** (раздел 0): антифрод синхронный, до передачи на платформу; 202 возвращает `PROCESSING` / `ON_REVIEW` / `REJECTED`. Прежняя версия OAS (антифрод асинхронно, после передачи на платформу) противоречила Task. |
| 5 | §8.1: из `ON_REVIEW` после проверки — в `PROCESSING`; в Task A8 обратного пути нет. | По §8.1 (`[80]`–`[83]`). |
| 6 | AC-02 не требует поля причины для возраста и гражданства; BR-DR-OPEN-01 его задаёт. | `reason` для всех трёх случаев. |

## 9. Новые бизнес-коды

Нет. Все коды взяты из `business-rules.md` §9; на диаграмме нет кодов, которых нет в OAS (сверка — раздел 2.2).

## 10. Открытые вопросы

1. Нужны ли для кошелька ИП дополнительные сведения или документы от клиента, кроме данных АБС? (Task, FR §8.)
2. Сверяется ли версия договора с платформой при принятии (A-03), или модуль хранит актуальную версию у себя? От ответа зависят 503 / 504 у `acceptWalletApplicationContract`.
3. Формат CSR и алгоритмы ключа ЭП (A-02).
4. Как получить новую ссылку на Госуслуги, если заявка зависла в `ESIA_PENDING`? Сейчас повтор возможен только с тем же `Idempotency-Key`.
5. Нужно ли клиенту поле `errorCode` для `FAILED`?
6. Истекает ли заявка в `ON_REVIEW`? По §8.1 перехода `ON_REVIEW → EXPIRED` нет.
7. Формат номера счёта ЦР и `MASK-WALLET` (sensitive-data §11).
8. Уведомление об открытии (BR-DR-OPEN-12): подтвердить, что оно вне клиентского API.
9. Нужен ли отдельный `ownerType` для лиц частной практики (BR-DR-OPEN-06)?
10. **Новое.** Что происходит, если на шаге 5 платформа недоступна (INT-DRP-01)? Заявка остаётся в `ESIA_PENDING` и проверка повторяется сервером, или заявка переходит в `FAILED`? В источниках не задано.
11. **Новое.** Политика повтора и таймаут для ЕСИА, политика повтора для синхронных операций платформы (A-06). Пока принято «без повтора».
12. **Новое.** Повторяет ли модуль проверку антифрода при повторе клиентом с тем же `Idempotency-Key` после 503 от платформы, или использует сохранённое решение?

## 11. Проверка человеком

| # | Замечание | Тип | Исправление | ai-log |
|---|---|---|---|---|
| 1 | `acceptWalletApplicationContract` и `cancelWalletApplication` — `POST` без `Idempotency-Key`. Если сервер выполнил переход, а клиент получил `504`, повтор без ключа вернёт `409 INVALID_APPLICATION_STATE`, и приложение не узнает, выполнен ли шаг. | Ошибка агента | Добавлены `Idempotency-Key` и `422 IDEMPOTENCY_KEY_REUSED` в обе операции; на диаграмме — ключ в сообщениях, ветка `422` и `opt` повтора с сохранённым ответом. Коммит `fix(human)`. | ERR-01 |
