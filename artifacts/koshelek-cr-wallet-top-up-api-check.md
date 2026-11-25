# Отчёт проверки: koshelek-cr-wallet-top-up

| Параметр | Значение |
|---|---|
| OAS | `artifacts/koshelek-cr-wallet-top-up-api.yaml` (OpenAPI 3.0.3, API 1.1.0) |
| Sequence-диаграмма | `artifacts/koshelek-cr-wallet-top-up-sequence.puml` |
| Task | SCRUM-2195 «КошелёкЦР - Пополнение кошелька с обычного счёта» |
| Story | SCRUM-2197 «КошелёкЦР - пополнение кошелька цифрового рубля с обычного счёта» (связь *relates to*) |
| Продукт / контекст | `knowledge/context/koshelek-cr/` |
| Подготовлено | Агентом (skill `oas-from-jira`), 2026-11-25. Версия 1.1.0: решения владельца продукта от 25.11.2026 по открытым вопросам версии 1.0.0 (раздел 10). Черновик для проверки человеком. |

## 1. Источники

| Источник | Как использован |
|---|---|
| SCRUM-2195 (Jira) | Акторы, границы, предусловия, шаги 1–9, альтернативы A1–A10, исключения E1–E3, внешние системы, ключевые данные, НФТ, подтверждённые решения |
| SCRUM-2197 (Jira) | AC-01…AC-21 |
| `knowledge/context/koshelek-cr/functional-requirements.md` | FR-GEN-01…07, FR-TOP-01…10 |
| `knowledge/context/koshelek-cr/non-functional-requirements.md` | NFR-PERF-01…03, NFR-PERF-06, NFR-AV-02…05, NFR-RATE-02, 03, NFR-SEC-01, 03…05, NFR-OBS-01, 02, NFR-CMP-03 |
| `knowledge/context/koshelek-cr/integrations.md` | Схема «Пополнение кошелька», политика `RP-EXP` и окно `W`, INT-ABS-02…05, INT-DRP-02, 04…06, 08, INT-AF-02, 03, таблицы соответствия ошибок |
| `knowledge/business-rules.md` | §1.1–1.4 (в т. ч. BR-CNF-04), §2, §4 (в т. ч. BR-DR-TOP-07), §6, §8.2, §8.4, §8.7, §9, §10 в редакции PR «knowledge: решения 25.11.2026 по пополнению кошелька» |
| `knowledge/sensitive-data.md` | §3.2, §3.3, §4 (`MASK-ACCOUNT`, `MASK-WALLET`, `MASK-HASH`), §5 SD-API-01…04, 07, 10, 13, 15 |
| `knowledge/oas-conventions.md` | Весь файл |
| `artifacts/koshelek-cr-wallet-opening-api.yaml` | Общие компоненты (`Problem`, `Money`, заголовки, `IdempotencyKey`) повторены дословно ради единого стиля |

## 2. Результаты проверок

### 2.1 Spectral

```
npx @stoplight/spectral-cli@6 lint artifacts/koshelek-cr-wallet-top-up-api.yaml --ruleset .spectral.yaml
Spectral 6.17.1: No results with a severity of 'hint' or higher found!
```

0 ошибок, 0 предупреждений. `.spectral.yaml` не менялся.

### 2.2 PlantUML

```
java -jar plantuml.jar -checkonly artifacts/koshelek-cr-wallet-top-up-sequence.puml
PlantUML 1.2024.7, код возврата 0
```

- Рендер просмотрен целиком (`-DPLANTUML_LIMIT_SIZE=16384`, 1622×6340 px): подписи не обрезаны, пустых веток нет.
- Сверка бизнес-кодов OAS ↔ диаграмма (команда из SKILL.md, шаг 9, по каталогу из PR с решениями 25.11.2026): пустой вывод. `WALLET_CREDIT_REFUSED` в каталоге без HTTP-кода (это `errorCode` операции), скрипт его не видит; проверен вручную: есть в OAS и на диаграмме.
- Статусы из OAS в метках `[S]`: все 9 (`CREATED`, `CHECKING`, `ON_REVIEW`, `FUNDS_HELD`, `PLATFORM_PROCESSING`, `RELEASING_HOLD`, `COMPLETED`, `REJECTED`, `FAILED`). Каждое значение `rejectReason` и `errorCode` стоит в ветке своего статуса.
- Операции OAS на диаграмме: все 5, с тем же методом, путём и `operationId`.

## 3. Операции

| # | Метод и путь | operationId | Шаг Task | Сообщения на диаграмме |
|---|---|---|---|---|
| 1 | `GET /wallets` | `listWallets` | 2 | `[02]`–`[06]` |
| 2 | `GET /wallets/{walletId}/top-up-source-accounts` | `listTopUpSourceAccounts` | 4 | `[08]`–`[13]` |
| 3 | `GET /wallets/{walletId}/top-up-limit` | `getWalletTopUpLimit` | 5 | `[14]`–`[20]` |
| 4 | `POST /wallets/{walletId}/top-ups` | `createWalletTopUp` | 7 | `[22]`–`[54]` |
| 5 | `GET /wallets/{walletId}/top-ups/{topUpId}` | `getWalletTopUp` | 8 | `[67]`–`[70]`; переходы статуса `[55]`–`[66]` |

Шаги 1, 3, 6, 9 размечены `[UI]` и операций не дают (`[01]`, `[07]`, `[21]`, `[71]`).

## 4. Матрица покрытия AC

| AC | Элемент OAS | Элемент диаграммы | Покрыт |
|---|---|---|---|
| AC-01 | `listWallets` → 200 `WalletList` (`ownerType`, `status`, `walletNumberMasked`) | Шаг 2, `[06]` | Да |
| AC-02 | `listTopUpSourceAccounts` → 200 `SourceAccountList` (фильтр по владельцу, статусу, валюте, типу; `debitAllowed`) | Шаг 4, `[13]` + note фильтра | Да |
| AC-03 | `getWalletTopUpLimit` → 200 `limited = true`, `limitAmount`, `usedAmount`, `remainingAmount`, `windowDays = 30` | Шаг 5, `[17]`–`[20]` | Да |
| AC-04 | `getWalletTopUpLimit` → 200 `limited = false` | Шаг 5, `[16]` | Да |
| AC-05 | `createWalletTopUp` → 202 `TopUp` (`topUpId`, `amount`, `sourceAccountId`, `walletId`, `status = CHECKING`), `Location` | Шаг 7, `[54]` | Да |
| AC-06 | `getWalletTopUp` → `COMPLETED`, `walletBalanceAfter`, `balanceAsOf` | Асинхронная обработка, `[59]`–`[61]`; `[70]` | Да |
| AC-07 | `IdempotencyKey`; 202 повтор, описание операции | `[24]` | Да |
| AC-08 | `createWalletTopUp` → 422 `INSUFFICIENT_FUNDS`; гонка после ответа → `REJECTED` (`INSUFFICIENT_FUNDS`) | `[46]`; `[56]`–`[57]` + note | Да |
| AC-09 | `createWalletTopUp` → 422 `TOP_UP_MONTHLY_LIMIT_EXCEEDED` + `TopUpProblem.remainingAmount` | `[36]`–`[37]`, `[47]` | Да |
| AC-10 | 403 `ACCOUNT_NOT_OWNED`; 422 `ACCOUNT_CLOSED`, `ACCOUNT_CURRENCY_NOT_SUPPORTED`, `ACCOUNT_RESTRICTED`, `ACCOUNT_TYPE_NOT_SUPPORTED` | `[43]`–`[45]` | Да |
| AC-11 | 400 `VALIDATION_ERROR`, `errors[].field = amount.value` | `[26]` | Да |
| AC-12 | 422 `WALLET_BLOCKED`, `WALLET_RESTRICTED` (по INT-DRP-04) | `[30]`–`[31]`, `[39]`, `[41]` | Да |
| AC-13 | 403 `CERTIFICATE_INVALID` | `[28]` | Да |
| AC-14 | 202 `REJECTED` (`OPERATION_DECLINED_BY_SECURITY`) / `ON_REVIEW`; переход после ручной проверки | `[52]`, `[53]`, `[55]` | Да |
| AC-15 | `getWalletTopUp` → `RELEASING_HOLD` → `FAILED`, `errorCode` | Note отказа платформы, `[66]` + notes `[S]` | Да |
| AC-16 | 503 `CORE_BANKING_UNAVAILABLE` / `PLATFORM_UNAVAILABLE`, 504 `UPSTREAM_TIMEOUT` в операциях 2–4; `FAILED` (`CORE_BANKING_UNAVAILABLE`), если АБС недоступна дольше окна после ответа | `[12]`, `[19]`, `[38]`; note после `[57]` | Да |
| AC-17 | `PLATFORM_PROCESSING` до уточнения INT-DRP-08, описание `getWalletTopUp` | `[63]`–`[64]`, note «сохраняется» | Да |
| AC-18 | `getWalletTopUp` → 404 `RESOURCE_NOT_FOUND` | `[69]` | Да |
| AC-19 | 409 `CONTRACT_REACCEPTANCE_REQUIRED` (сверка с INT-DRP-02 в запросе) | `[32]`–`[33]`, `[42]` | Да |
| AC-20 | 503 `SERVICE_UNAVAILABLE`, `Retry-After`; повтор с ключом проходит антифрод заново | `[48]`–`[51]` | Да |
| AC-21 | 429 `TOO_MANY_REQUESTS`, `Retry-After` | `[25]` | Да |

## 5. Трассировка элементов

### 5.1 Операции, ответы, заголовки

| Элемент OAS | Источник |
|---|---|
| `listWallets`, `x-external-systems: [DR_PLATFORM]` | Шаг 2 `[API]`; FR-TOP-09; Task «Внешние системы»: платформа, шаг 2; INT-DRP-04 |
| `listWallets` без пагинации, `maxItems: 2` | BR-DR-OPEN-04, 05 (не больше двух кошельков) |
| `listTopUpSourceAccounts`, `[CORE_BANKING]` | Шаг 4 `[API]` `[EXT:CORE_BANKING]`; FR-TOP-01; INT-ABS-02 |
| `getWalletTopUpLimit`, `[DR_PLATFORM]` | Шаг 5 `[API]` `[EXT:DR_PLATFORM]`; FR-TOP-04; INT-DRP-05 |
| `createWalletTopUp` 202 + `Location`, статус `CHECKING` | Шаг 7 («принята к асинхронной обработке»); NFR-PERF-02, NFR-PERF-06; OAS-ASYNC-01, 04; BR-DR-TOP-06 |
| `createWalletTopUp`, `[DR_PLATFORM, CORE_BANKING, ANTIFRAUD]` | Шаг 7; синхронно INT-DRP-02, 04, 05, INT-ABS-02, INT-AF-02; после ответа INT-ABS-03, INT-DRP-06 |
| `getWalletTopUp` | Шаг 8 `[API]`; FR-TOP-07; OAS-ASYNC-02 |
| 403 `ACCOUNT_NOT_OWNED`, `CERTIFICATE_INVALID` | A1, A7; BR-ACC-01; §8.7 |
| 404 во всех операциях с `walletId` / `topUpId`, в т. ч. закрытый кошелёк | BR-GEN-02, OAS-ERR-07, AC-18, §8.2 |
| 409 `CONTRACT_REACCEPTANCE_REQUIRED` | A6a, AC-19, BR-DR-OPEN-09 |
| 422 коды счёта, лимита, кошелька | A1, A2, A4, A5, A6; BR-ACC-02…06, BR-DR-TOP-02, §8.2 |
| 422 `SIGNATURE_INVALID` | BR-CNF-04 |
| 422 `IDEMPOTENCY_KEY_REUSED` | BR-IDM-02 |
| 429 у `createWalletTopUp` | NFR-RATE-02, AC-21 |
| 429 у `getWalletTopUp` | NFR-RATE-03 |
| 503 / 504 у операций 1–4 | Внешний вызов по `integrations.md`; E1, E3; BR-TIME-03, BR-AF-06 |
| `Idempotency-Key` у `createWalletTopUp` | BR-IDM-01, OAS-HDR-03, A10 |
| `Cache-Control: no-store` у 200/202 | SD-API-10 (ответы с C2) |
| `Retry-After` у 429 и 503 | OAS-HDR-05 |

### 5.2 Схемы и поля

| Элемент | Источник |
|---|---|
| `WalletSummary` (`walletId`, `ownerType`, `status`, `walletNumberMasked`) | AC-01, FR-TOP-09, Task «Ключевые данные», OAS-TYPE-13 |
| `WalletStatus` (без `CLOSED`) | §8.2: закрытый кошелёк в API не существует |
| `SourceAccount` (`accountId`, `accountNumberMasked`, `accountType`, `availableBalance`, `debitAllowed`) | AC-02, Task «Ключевые данные», FR-TOP-01 |
| `TopUpLimit` (`limited`, `windowDays`, `limitAmount`, `usedAmount`, `remainingAmount`) | AC-03, AC-04, BR-DR-TOP-02…04 |
| `CreateWalletTopUpRequest` (`sourceAccountId`, `amount`, `signature`) | FR-TOP-02, FR-GEN-07, NFR-SEC-05, BR-DR-TOP-01, BR-CNF-04 (формат подписи) |
| `TopUpStatus` | §8.4 без `AWAITING_CONFIRMATION`, `EXPIRED`, `CANCELLED` (см. раздел 8, п. 4) |
| `TopUp.rejectReason` | §8.4 (`REJECTED`, атрибуты), AC-14, таблица соответствия ошибок АБС |
| `TopUp.errorCode` | AC-15, A9, §8.4 (атрибуты), BR-DR-TOP-07, таблица соответствия ошибок платформы |
| `TopUp.fee` | BR-DR-TOP-05 («поле `fee` оставить для расширения»), FR-TOP-08 |
| `TopUp.walletBalanceAfter`, `balanceAsOf` | AC-06, BR-DR-OPS-03 |
| `TopUp.createdAt`, `completedAt` | Task «Ключевые данные» («даты создания и завершения») |
| `TopUpProblem.remainingAmount` | AC-09 |
| `Money`, `Problem`, общие ответы | OAS-TYPE-04, OAS-ERR-02, OAS-ERR-05 |

### 5.3 Элементы диаграммы

| Элемент диаграммы | Источник |
|---|---|
| Участники Клиент, Приложение, КошелёкЦР | Task «Акторы» |
| Платформа ЦР, АБС, Антифрод (порядок первого вызова) | Task «Внешние системы»; `integrations.md` |
| Разделы «Шаг 1…9» | Task «Основной сценарий» |
| Ветки 422 / 403 / 409 в шаге 7 | A1–A7, A6a; AC-08…AC-13, AC-19 |
| Ветки DENY / REVIEW / ALLOW | A8, AC-14, `integrations.md` «Антифрод» |
| Ветка «повтор с тем же ключом» | A10, AC-07, BR-IDM-01 |
| `opt` повтор клиентом после 503 антифрода | E3, AC-20, BR-IDM-03 |
| Раздел «Асинхронная обработка» | Шаг 8, BR-DR-TOP-06, §8.4, E2, AC-15, AC-17; integrations «АБС», «Платформа ЦР» (окно `W`) |
| `par` проверок в шаге 7 | NFR-PERF-06 |
| `loop` опроса 2 с / 60 с, ветка 429 | NFR-PERF-03, NFR-RATE-03 |

### 5.4 Чувствительные данные

| Поле | Уровень | Решение |
|---|---|---|
| `walletNumberMasked` | C2 | Только маска `MASK-WALLET`; полного номера в API нет (SD-API-04, 05) |
| `accountNumberMasked` | C2 | Только маска `MASK-ACCOUNT`; в путях и теле запроса передаётся `accountId` (SD-API-02, 03) |
| `debitAllowed` | C2 | Только признак; основание ограничения не передаётся (SD-API-15) |
| `availableBalance`, `amount`, `usedAmount`, `remainingAmount`, `walletBalanceAfter` | C2 | Полностью владельцу, в логах `MASK-HIDE`; ответы с `Cache-Control: no-store` |
| `signature` | C1 | `writeOnly`, в логах `MASK-HASH` |
| Решение антифрода | C3 | Не раскрывается: общий код `OPERATION_DECLINED_BY_SECURITY` (BR-AF-03, SD-API-15) |
| Пароль к кошельку, закрытый ключ ЭП | C3 | В API нет (FR-GEN-06, BR-DR-OPEN-10) |

## 6. Ретраи и таймауты

| Вызов | Таймаут | Повторы | Исход после исчерпания / по таймауту | Источник | Сообщения `[NN]` |
|---|---|---|---|---|---|
| INT-DRP-04 (список кошельков, проверка кошелька, баланс) | 1,5 с | `RP-EXP`, 3 попытки | 503 `PLATFORM_UNAVAILABLE` / 504 `UPSTREAM_TIMEOUT`; в асинхронной части операция ждёт | `integrations.md` «Политика повтора», NFR-AV-05 | `[03]`, `[30]`, `[60]` |
| INT-ABS-02 (счета, проверка счёта) | 1 с | `RP-EXP`, 3 попытки | 503 `CORE_BANKING_UNAVAILABLE` / 504 `UPSTREAM_TIMEOUT` | То же | `[10]`, `[34]` |
| INT-DRP-05 (лимит) | 1,5 с | `RP-EXP`, 3 попытки | 503 `PLATFORM_UNAVAILABLE` / 504 `UPSTREAM_TIMEOUT` | То же | `[17]`, `[36]` |
| INT-DRP-02 (версия договора) | 1,5 с | `RP-EXP`, 3 попытки | 503 `PLATFORM_UNAVAILABLE` / 504 `UPSTREAM_TIMEOUT` | То же | `[32]` |
| INT-AF-02 (антифрод) | 500 мс | `RP-EXP`, 3 попытки | 503 `SERVICE_UNAVAILABLE`, операция не создаётся; повтор клиентом проходит проверку заново | BR-AF-06, BR-IDM-03, E3 | `[48]`, `[51]` |
| INT-ABS-03 (блокировка, после ответа 202) | 1 с | Серии `RP-EXP` в окне `W` = 10 мин; идемпотентна по id операции | Окно истекло → `FAILED` (`CORE_BANKING_UNAVAILABLE`); поставленная блокировка снимается после восстановления АБС | `integrations.md` «АБС», BR-DR-TOP-06 | `[56]` |
| INT-DRP-06 (зачисление) | 1,5 с на приём, результат до 30 с | Без повтора до уточнения INT-DRP-08; после ответа «не получено» модуль отправляет повторно сериями `RP-EXP` в окне `W`, тот же id | Окно истекло → `RELEASING_HOLD` → `FAILED` (`PLATFORM_UNAVAILABLE`) | NFR-AV-03, E2, `integrations.md` «Платформа ЦР» | `[58]`, `[65]` |
| INT-DRP-08 (уточнение статуса) | 1,5 с | `RP-EXP`, 3 попытки | Операция остаётся в `PLATFORM_PROCESSING` (AC-17); `FAILED` по недоступности уточнения не ставится: риск двойного зачисления | NFR-AV-03, §8.4 | `[63]` |
| INT-ABS-04 (списание, после `COMPLETED`) | 1 с | Серии `RP-EXP` в окне `W`; идемпотентна по id блокировки | Окно истекло → инцидент; статус операции не меняется | `integrations.md` «АБС» | `[62]` |
| INT-ABS-05 (снятие блокировки) | 1 с | Серии `RP-EXP` до снятия; идемпотентна по id блокировки | Операция в `RELEASING_HOLD` до снятия; после окна `W` модуль поднимает инцидент | `integrations.md` «АБС», BR-ACC-07 | `[66]` |
| `POST` создания клиентом | 10 с (API Gateway и клиент) | Повтор с тем же `Idempotency-Key` | Ответ 5xx не сохраняется, запрос выполняется заново | NFR-PERF-06, BR-IDM-03 | `[22]`, `[51]` |
| `GET` статуса клиентом | нет | Опрос каждые 2 с, не дольше 60 с; не чаще 1 раза в секунду | 429 `TOO_MANY_REQUESTS`; после 60 с приложение показывает «операция в обработке» | NFR-PERF-03, NFR-RATE-03 | `[67]`–`[70]` |

## 7. Допущения

| ID | Допущение | Где отражено (OAS / диаграмма) |
|---|---|---|
| A-03 | Для кошелька ИП модуль отвечает `limited = false` без запроса к платформе. | `getWalletTopUpLimit` `x-assumption`; note у `[16]` |
| A-06 | Типы счёта в API: `CURRENT` (текущий счёт физлица) и `SETTLEMENT` (расчётный счёт ИП). Соответствие типам АБС задаёт адаптер. | `SourceAccount.accountType` `x-assumption`; note `[13]` |

Допущения A-01, A-02, A-04, A-05, A-07, A-08, A-09 версии 1.0.0 сняты решениями от 25.11.2026 (раздел 10).

## 8. Противоречия между источниками

| # | Противоречие | Решение |
|---|---|---|
| 1 | AC-19 требует отказа, если Банк России изменил договор, а клиент его не принял. У шага 7 Task нет вызова INT-DRP-02, в таблице внешних систем Task платформа указана с INT-DRP-04…06, 08. | По приоритету AC → Task и правилу из ERR-02: `createWalletTopUp` вызывает INT-DRP-02 в том же запросе (`RP-EXP`, 503/504). Сверка с сохранённой версией не допускается: она не увидит новую версию. В SCRUM-2195 шаг 7 стоит дополнить INT-DRP-02. |
| 2 | AC-12 требует проверить блокировку и ограничения кошелька. Ограничения ставит Банк России (BR-DR-RST-02…04), модуль о них знает только от платформы. Шаг 7 Task не перечисляет INT-DRP-04. | Та же логика: INT-DRP-04 вызывается в `createWalletTopUp`. Task «Внешние системы» перечисляет INT-DRP-04 для платформы, противоречие только в тексте шага. |
| 3 | `integrations.md` (таблица соответствия ошибок платформы): отказ платформы по лимиту или ограничениям даёт статус операции `REJECTED`. Task A9, AC-15 и §8.4: отказ платформы ведёт через `RELEASING_HOLD` в `FAILED`. | Приоритет у AC-15: `FAILED` с `errorCode`. `REJECTED` оставлен для отказов до зачисления (антифрод, отказ АБС в блокировке). Таблица исправлена в PR с решениями 25.11.2026. |
| 4 | OAS-TYPE-08 требует перечислять статусы полностью; §8.4 включает `AWAITING_CONFIRMATION`, `EXPIRED`, `CANCELLED`. | §8.4 сам говорит, что `AWAITING_CONFIRMATION` нужен только при подтверждении отдельным шагом; `EXPIRED` и `CANCELLED` достижимы только из него и через отмену, которой нет в объёме (Task «Не входит» OTP, FR-TOP-05). Статусы исключены, причина записана в `TopUpStatus.description`. |
| 5 | OAS-PAGE-01 требует курсорную пагинацию для списков. | У клиента не больше двух кошельков (BR-DR-OPEN-04, 05), счетов-источников единицы. Пагинация не следует из требований (OAS-FILE-07), схемы названы `WalletList` и `SourceAccountList`, а не `*Page`. |
| 6 | Шаг 7 Task предполагает блокировку суммы до ответа приложению. NFR-PERF-01 (2 с p95) при шести последовательных вызовах не выполнить. | Решение 25.11.2026 (NFR-PERF-06): проверки параллельно, `202` после антифрода, блокировка и зачисление асинхронно. AC-08 и AC-10 выполняются синхронной проверкой по INT-ABS-02; редкий отказ АБС в блокировке приходит статусом `REJECTED`. |

## 9. Новые бизнес-коды

Добавлены в каталог `knowledge/business-rules.md` §9 в PR с решениями 25.11.2026:

- `SIGNATURE_INVALID` (422, BR-CNF-04): подпись распоряжения не прошла проверку модулем;
- `WALLET_CREDIT_REFUSED` (`errorCode` операции, BR-DR-TOP-07): платформа отказала в зачислении по причине вне таблицы соответствия.

## 10. Открытые вопросы

Вопросы версии 1.0.0 закрыты решениями владельца продукта от 25.11.2026:

| # | Вопрос | Решение | Что изменилось в артефакте |
|---|---|---|---|
| 1 | Подписываемое распоряжение (A-08) | Канонический JSON (RFC 8785) из `walletId`, `sourceAccountId`, `amount`, `Idempotency-Key`; отсоединённая CMS по ГОСТ; модуль проверяет подпись (BR-CNF-04) | `signature.description`; 422 `SIGNATURE_INVALID`; `[29]` |
| 2 | Таймаут блокировки INT-ABS-03 (A-05) | Блокировка идемпотентна по id операции и выполняется после ответа 202 | Убраны ответы 422 / 503 / 504 по блокировке из `createWalletTopUp`; переходы `CHECKING` → `FUNDS_HELD` / `REJECTED` / `FAILED` |
| 3 | Повторная отправка INT-DRP-06 (A-01) | Серии `RP-EXP` в окне `W` = 10 мин; платформа отбрасывает дубли | Описание `getWalletTopUp`, `[65]` |
| 4 | Кошелёк в `CLOSING` / `CLOSED` (A-04) | `CLOSING` → `WALLET_BLOCKED`; `CLOSED` → 404 и не возвращается в списке | `WalletStatus` без `CLOSED`; `[39]`, `[40]` |
| 5 | Сбой INT-ABS-04 / 05 (A-09) | Повтор по id блокировки; `COMPLETED` после зачисления; `RELEASING_HOLD` до снятия | `TopUpStatus.description`; `[62]`, `[66]` |
| 6 | Счета с ограничениями на расход (A-02) | Показываются с `debitAllowed: false`, без основания | `SourceAccount.debitAllowed`, пример |
| 7 | Отказ платформы вне таблицы | `errorCode: WALLET_CREDIT_REFUSED` | `TopUp.errorCode` |
| 8 | 2 с p95 для `createWalletTopUp` | Проверки параллельно, 202 после антифрода, таймаут клиента 10 с (NFR-PERF-06) | Описание операции; `par` в `[30]`–`[37]`; 202 `CHECKING` |

Остались:

1. Формат подписи (BR-CNF-04) нужно сверить с альбомом сообщений платформы. Если платформа требует свой документ распоряжения, понадобится шаг подготовки документа на сервере.
2. В SCRUM-2195 шаг 7 стоит дополнить вызовами INT-DRP-02 и INT-DRP-04 (противоречия 1–2) и асинхронной блокировкой (противоречие 6).

## 11. Проверка человеком

<!-- Заполняет проверяющий -->
