# Отчёт проверки: koshelek-cr-wallet-top-up

| Параметр | Значение |
|---|---|
| OAS | `artifacts/koshelek-cr-wallet-top-up-api.yaml` (OpenAPI 3.0.3, API 1.0.0) |
| Sequence-диаграмма | `artifacts/koshelek-cr-wallet-top-up-sequence.puml` |
| Task | SCRUM-2195 «КошелёкЦР - Пополнение кошелька с обычного счёта» |
| Story | SCRUM-2197 «КошелёкЦР - пополнение кошелька цифрового рубля с обычного счёта» (связь *relates to*) |
| Продукт / контекст | `knowledge/context/koshelek-cr/` |
| Подготовлено | Агентом (skill `oas-from-jira`), 2026-11-25. Черновик для проверки человеком. |

## 1. Источники

| Источник | Как использован |
|---|---|
| SCRUM-2195 (Jira) | Акторы, границы, предусловия, шаги 1–9, альтернативы A1–A10, исключения E1–E3, внешние системы, ключевые данные, НФТ, подтверждённые решения |
| SCRUM-2197 (Jira) | AC-01…AC-21 |
| `knowledge/context/koshelek-cr/functional-requirements.md` | FR-GEN-01…07, FR-TOP-01…09 |
| `knowledge/context/koshelek-cr/non-functional-requirements.md` | NFR-PERF-01…03, NFR-AV-02…05, NFR-RATE-02, 03, NFR-SEC-01, 03…05, NFR-OBS-01, 02, NFR-CMP-03 |
| `knowledge/context/koshelek-cr/integrations.md` | Схема «Пополнение кошелька», политика `RP-EXP`, INT-ABS-02…05, INT-DRP-02, 04…06, 08, INT-AF-02, 03, таблицы соответствия ошибок |
| `knowledge/business-rules.md` | §1.1–1.4, §2, §4, §6, §8.2, §8.4, §8.7, §9 |
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

- Рендер просмотрен целиком (`-DPLANTUML_LIMIT_SIZE=16384`, 1625×6464 px): подписи не обрезаны, пустых веток нет.
- Сверка бизнес-кодов OAS ↔ диаграмма (команда из SKILL.md, шаг 9): пустой вывод. На первом прогоне скрипт не нашёл `ACCOUNT_CURRENCY_NOT_SUPPORTED`: код стоял сразу после `\n`, и `\b` не срабатывал. Подпись переписали, код на диаграмме остался.
- Статусы из OAS в метках `[S]`: все 9 (`CREATED`, `CHECKING`, `ON_REVIEW`, `FUNDS_HELD`, `PLATFORM_PROCESSING`, `RELEASING_HOLD`, `COMPLETED`, `REJECTED`, `FAILED`). Каждое значение `rejectReason` и `errorCode` стоит в ветке своего статуса.
- Операции OAS на диаграмме: все 5, с тем же методом, путём и `operationId`.

## 3. Операции

| # | Метод и путь | operationId | Шаг Task | Сообщения на диаграмме |
|---|---|---|---|---|
| 1 | `GET /wallets` | `listWallets` | 2 | `[02]`–`[06]` |
| 2 | `GET /wallets/{walletId}/top-up-source-accounts` | `listTopUpSourceAccounts` | 4 | `[08]`–`[13]` |
| 3 | `GET /wallets/{walletId}/top-up-limit` | `getWalletTopUpLimit` | 5 | `[14]`–`[20]` |
| 4 | `POST /wallets/{walletId}/top-ups` | `createWalletTopUp` | 7 | `[22]`–`[59]` |
| 5 | `GET /wallets/{walletId}/top-ups/{topUpId}` | `getWalletTopUp` | 8 | `[72]`–`[75]`; переходы статуса `[60]`–`[71]` |

Шаги 1, 3, 6, 9 размечены `[UI]` и операций не дают (`[01]`, `[07]`, `[21]`, `[76]`).

## 4. Матрица покрытия AC

| AC | Элемент OAS | Элемент диаграммы | Покрыт |
|---|---|---|---|
| AC-01 | `listWallets` → 200 `WalletList` (`ownerType`, `status`, `walletNumberMasked`) | Шаг 2, `[06]` | Да |
| AC-02 | `listTopUpSourceAccounts` → 200 `SourceAccountList` (фильтр по владельцу, статусу, валюте, типу) | Шаг 4, `[13]` + note фильтра | Да |
| AC-03 | `getWalletTopUpLimit` → 200 `limited = true`, `limitAmount`, `usedAmount`, `remainingAmount`, `windowDays = 30` | Шаг 5, `[17]`–`[20]` | Да |
| AC-04 | `getWalletTopUpLimit` → 200 `limited = false` | Шаг 5, `[16]` | Да |
| AC-05 | `createWalletTopUp` → 202 `TopUp` (`topUpId`, `amount`, `sourceAccountId`, `walletId`, `status`), `Location` | Шаг 7, `[59]` | Да |
| AC-06 | `getWalletTopUp` → `COMPLETED`, `walletBalanceAfter`, `balanceAsOf` | Асинхронная обработка, `[63]`–`[66]`; `[75]` | Да |
| AC-07 | `IdempotencyKey`; 202 повтор, описание операции | `[24]` | Да |
| AC-08 | `createWalletTopUp` → 422 `INSUFFICIENT_FUNDS` | `[44]`, `[54]` | Да |
| AC-09 | `createWalletTopUp` → 422 `TOP_UP_MONTHLY_LIMIT_EXCEEDED` + `TopUpProblem.remainingAmount` | `[35]`–`[36]`, `[45]` | Да |
| AC-10 | 403 `ACCOUNT_NOT_OWNED`; 422 `ACCOUNT_CLOSED`, `ACCOUNT_CURRENCY_NOT_SUPPORTED`, `ACCOUNT_RESTRICTED`, `ACCOUNT_TYPE_NOT_SUPPORTED` | `[41]`–`[43]` | Да |
| AC-11 | 400 `VALIDATION_ERROR`, `errors[].field = amount.value` | `[26]` | Да |
| AC-12 | 422 `WALLET_BLOCKED`, `WALLET_RESTRICTED` (по INT-DRP-04) | `[29]`–`[30]`, `[38]`–`[39]` | Да |
| AC-13 | 403 `CERTIFICATE_INVALID` | `[28]` | Да |
| AC-14 | 202 `REJECTED` (`OPERATION_DECLINED_BY_SECURITY`) / `ON_REVIEW`; переход после ручной проверки | `[50]`, `[51]`, `[60]` | Да |
| AC-15 | `getWalletTopUp` → `RELEASING_HOLD` → `FAILED`, `errorCode` | `[67]` + notes `[S]` | Да |
| AC-16 | 503 `CORE_BANKING_UNAVAILABLE` / `PLATFORM_UNAVAILABLE`, 504 `UPSTREAM_TIMEOUT` в операциях 2–4 | `[12]`, `[19]`, `[37]`, `[55]` | Да |
| AC-17 | `PLATFORM_PROCESSING` до уточнения INT-DRP-08, описание `getWalletTopUp` | `[68]`–`[69]`, note «сохраняется» | Да |
| AC-18 | `getWalletTopUp` → 404 `RESOURCE_NOT_FOUND` | `[74]` | Да |
| AC-19 | 409 `CONTRACT_REACCEPTANCE_REQUIRED` (сверка с INT-DRP-02 в запросе) | `[31]`–`[32]`, `[40]` | Да |
| AC-20 | 503 `SERVICE_UNAVAILABLE`, `Retry-After`; повтор с ключом проходит антифрод заново | `[46]`–`[49]` | Да |
| AC-21 | 429 `TOO_MANY_REQUESTS`, `Retry-After` | `[25]` | Да |

## 5. Трассировка элементов

### 5.1 Операции, ответы, заголовки

| Элемент OAS | Источник |
|---|---|
| `listWallets`, `x-external-systems: [DR_PLATFORM]` | Шаг 2 `[API]`; FR-TOP-09; Task «Внешние системы»: платформа, шаг 2; INT-DRP-04 |
| `listWallets` без пагинации, `maxItems: 2` | BR-DR-OPEN-04, 05 (не больше двух кошельков) |
| `listTopUpSourceAccounts`, `[CORE_BANKING]` | Шаг 4 `[API]` `[EXT:CORE_BANKING]`; FR-TOP-01; INT-ABS-02 |
| `getWalletTopUpLimit`, `[DR_PLATFORM]` | Шаг 5 `[API]` `[EXT:DR_PLATFORM]`; FR-TOP-04; INT-DRP-05 |
| `createWalletTopUp` 202 + `Location` | Шаг 7 («принята к асинхронной обработке»); NFR-PERF-02; OAS-ASYNC-01, 04 |
| `createWalletTopUp`, `[DR_PLATFORM, CORE_BANKING, ANTIFRAUD]` | Шаг 7; INT-DRP-02, 04…06, INT-ABS-02, 03, INT-AF-02 |
| `getWalletTopUp` | Шаг 8 `[API]`; FR-TOP-07; OAS-ASYNC-02 |
| 403 `ACCOUNT_NOT_OWNED`, `CERTIFICATE_INVALID` | A1, A7; BR-ACC-01; §8.7 |
| 404 во всех операциях с `walletId` / `topUpId` | BR-GEN-02, OAS-ERR-07, AC-18 |
| 409 `CONTRACT_REACCEPTANCE_REQUIRED` | A6a, AC-19, BR-DR-OPEN-09 |
| 422 коды счёта, лимита, кошелька | A1, A2, A4, A5, A6; BR-ACC-02…06, BR-DR-TOP-02, §8.2 |
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
| `WalletStatus` | §8.2 |
| `SourceAccount` (`accountId`, `accountNumberMasked`, `accountType`, `availableBalance`) | AC-02, Task «Ключевые данные» |
| `TopUpLimit` (`limited`, `windowDays`, `limitAmount`, `usedAmount`, `remainingAmount`) | AC-03, AC-04, BR-DR-TOP-02…04 |
| `CreateWalletTopUpRequest` (`sourceAccountId`, `amount`, `signature`) | FR-TOP-02, FR-GEN-07, NFR-SEC-05, BR-DR-TOP-01 |
| `TopUpStatus` | §8.4 без `AWAITING_CONFIRMATION`, `EXPIRED`, `CANCELLED` (см. раздел 8, п. 4) |
| `TopUp.rejectReason` | §8.4 (`REJECTED`), AC-14, A-07 |
| `TopUp.errorCode` | AC-15, A9, таблица соответствия ошибок платформы, A-01 |
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
| Раздел «Асинхронная обработка» | Шаг 8, BR-DR-TOP-06, §8.4, E2, AC-15, AC-17 |
| `loop` опроса 2 с / 60 с, ветка 429 | NFR-PERF-03, NFR-RATE-03 |

### 5.4 Чувствительные данные

| Поле | Уровень | Решение |
|---|---|---|
| `walletNumberMasked` | C2 | Только маска `MASK-WALLET`; полного номера в API нет (SD-API-04, 05) |
| `accountNumberMasked` | C2 | Только маска `MASK-ACCOUNT`; в путях и теле запроса передаётся `accountId` (SD-API-02, 03) |
| `availableBalance`, `amount`, `usedAmount`, `remainingAmount`, `walletBalanceAfter` | C2 | Полностью владельцу, в логах `MASK-HIDE`; ответы с `Cache-Control: no-store` |
| `signature` | C1 | `writeOnly`, в логах `MASK-HASH` |
| Решение антифрода | C3 | Не раскрывается: общий код `OPERATION_DECLINED_BY_SECURITY` (BR-AF-03, SD-API-15) |
| Пароль к кошельку, закрытый ключ ЭП | C3 | В API нет (FR-GEN-06, BR-DR-OPEN-10) |

## 6. Ретраи и таймауты

| Вызов | Таймаут | Повторы | Исход после исчерпания / по таймауту | Источник | Сообщения `[NN]` |
|---|---|---|---|---|---|
| INT-DRP-04 (список кошельков, проверка кошелька, баланс) | 1,5 с | `RP-EXP`, 3 попытки | 503 `PLATFORM_UNAVAILABLE` / 504 `UPSTREAM_TIMEOUT`; в асинхронной части операция ждёт | `integrations.md` «Политика повтора», NFR-AV-05 | `[03]`, `[29]`, `[64]` |
| INT-ABS-02 (счета, проверка счёта) | 1 с | `RP-EXP`, 3 попытки | 503 `CORE_BANKING_UNAVAILABLE` / 504 `UPSTREAM_TIMEOUT` | То же | `[10]`, `[33]` |
| INT-DRP-05 (лимит) | 1,5 с | `RP-EXP`, 3 попытки | 503 `PLATFORM_UNAVAILABLE` / 504 `UPSTREAM_TIMEOUT` | То же | `[17]`, `[35]` |
| INT-DRP-02 (версия договора) | 1,5 с | `RP-EXP`, 3 попытки | 503 `PLATFORM_UNAVAILABLE` / 504 `UPSTREAM_TIMEOUT` | То же | `[31]` |
| INT-AF-02 (антифрод) | 500 мс | `RP-EXP`, 3 попытки | 503 `SERVICE_UNAVAILABLE`, операция не создаётся; повтор клиентом проходит проверку заново | BR-AF-06, BR-IDM-03, E3 | `[46]`, `[49]` |
| INT-ABS-03 (блокировка) | 1 с | Без повтора: изменяющая операция АБС | Недоступна → 503 `CORE_BANKING_UNAVAILABLE`; таймаут → 504 `UPSTREAM_TIMEOUT` (A-05) | `integrations.md` «Политика повтора» | `[52]`, `[61]` |
| INT-DRP-06 (зачисление) | 1,5 с на приём, результат до 30 с | Без повтора до уточнения INT-DRP-08; после ответа «не получено» модуль отправляет повторно с тем же id, до 3 раз (A-01) | Приём не подтверждён → 202 `PLATFORM_PROCESSING`; отправки исчерпаны → `RELEASING_HOLD` → `FAILED` (`PLATFORM_UNAVAILABLE`) | NFR-AV-03, E2, `integrations.md` «Платформа ЦР» | `[58]`, `[62]`, `[70]` |
| INT-DRP-08 (уточнение статуса) | 1,5 с | `RP-EXP`, 3 попытки | Операция остаётся в `PLATFORM_PROCESSING` (AC-17); `FAILED` по недоступности уточнения не ставится: риск двойного зачисления | NFR-AV-03, §8.4 | `[68]` |
| INT-ABS-04 / INT-ABS-05 (списание / снятие блокировки) | 1 с | Без повтора: изменяющая операция АБС | Операция остаётся в текущем статусе, модуль дорабатывает шаг (A-09) | `integrations.md` «Политика повтора», BR-TIME-03 | `[66]`, `[67]`, `[71]` |
| `GET` статуса клиентом | нет | Опрос каждые 2 с, не дольше 60 с; не чаще 1 раза в секунду | 429 `TOO_MANY_REQUESTS`; после 60 с приложение показывает «операция в обработке» | NFR-PERF-03, NFR-RATE-03 | `[72]`–`[75]` |

## 7. Допущения

| ID | Допущение | Где отражено (OAS / диаграмма) |
|---|---|---|
| A-01 | Платформа не получила зачисление (по INT-DRP-08): модуль отправляет INT-DRP-06 повторно с тем же id не больше 3 раз, как для INT-DRP-03. После этого снимает блокировку, операция `FAILED`, `errorCode = PLATFORM_UNAVAILABLE`. | `TopUp.errorCode` `x-assumption`; note в разделе «Асинхронная обработка» |
| A-02 | Счета с ограничениями на расходные операции в список для пополнения не попадают (FR-TOP-01: «с которых можно пополнить»). | `listTopUpSourceAccounts` `x-assumption`; note `[13]` |
| A-03 | Для кошелька ИП модуль отвечает `limited = false` без запроса к платформе. | `getWalletTopUpLimit` `x-assumption`; note у `[16]` |
| A-04 | Кошелёк в `CLOSING` или `CLOSED` отклоняется кодом `WALLET_BLOCKED`. | `createWalletTopUp` `x-assumption`; note у `[38]` |
| A-05 | Таймаут блокировки INT-ABS-03 возвращается клиенту как 504, операция не создаётся. Возможную блокировку модуль сверяет вне клиентского API. | Ответ 504 `createWalletTopUp` `x-assumption`; `[56]` + note |
| A-06 | Типы счёта в API: `CURRENT` (текущий счёт физлица) и `SETTLEMENT` (расчётный счёт ИП). Соответствие типам АБС задаёт адаптер. | `SourceAccount.accountType` `x-assumption`; note `[13]` |
| A-07 | Если после `ON_REVIEW` АБС отклонила блокировку, операция переходит в `REJECTED` с кодом отказа АБС (`INSUFFICIENT_FUNDS`, `ACCOUNT_RESTRICTED`, `ACCOUNT_CLOSED`). | `TopUp.rejectReason` `x-assumption`; ветка после `[61]` |
| A-08 | Состав подписываемых данных распоряжения и формат подписи задаёт платформа ЦР. Контракт фиксирует только поле `signature` (Base64). | `CreateWalletTopUpRequest.signature` `x-assumption`; note шага 6 |
| A-09 | Сбой INT-ABS-03 (после проверки), INT-ABS-04 или INT-ABS-05 в асинхронной части не переводит операцию в `FAILED`: она остаётся в текущем статусе, модуль дорабатывает шаг. | `getWalletTopUp` `x-assumption`; notes в разделе «Асинхронная обработка» |

## 8. Противоречия между источниками

| # | Противоречие | Решение |
|---|---|---|
| 1 | AC-19 требует отказа, если Банк России изменил договор, а клиент его не принял. У шага 7 Task нет вызова INT-DRP-02, в таблице внешних систем Task платформа указана с INT-DRP-04…06, 08. | По приоритету AC → Task и правилу из ERR-02: `createWalletTopUp` вызывает INT-DRP-02 в том же запросе (`RP-EXP`, 503/504). Сверка с сохранённой версией не допускается: она не увидит новую версию. В SCRUM-2195 шаг 7 стоит дополнить INT-DRP-02. |
| 2 | AC-12 требует проверить блокировку и ограничения кошелька. Ограничения ставит Банк России (BR-DR-RST-02…04), модуль о них знает только от платформы. Шаг 7 Task не перечисляет INT-DRP-04. | Та же логика: INT-DRP-04 вызывается в `createWalletTopUp`. Task «Внешние системы» перечисляет INT-DRP-04 для платформы, противоречие только в тексте шага. |
| 3 | `integrations.md` (таблица соответствия ошибок платформы): отказ платформы по лимиту или ограничениям даёт статус операции `REJECTED`. Task A9, AC-15 и §8.4: отказ платформы ведёт через `RELEASING_HOLD` в `FAILED`. | Приоритет у AC-15: `FAILED` с `errorCode`. `REJECTED` оставлен для отказов до зачисления (антифрод, A-07). Таблицу в `integrations.md` стоит уточнить. |
| 4 | OAS-TYPE-08 требует перечислять статусы полностью; §8.4 включает `AWAITING_CONFIRMATION`, `EXPIRED`, `CANCELLED`. | §8.4 сам говорит, что `AWAITING_CONFIRMATION` нужен только при подтверждении отдельным шагом; `EXPIRED` и `CANCELLED` достижимы только из него и через отмену, которой нет в объёме (Task «Не входит» OTP, FR-TOP-05). Статусы исключены, причина записана в `TopUpStatus.description`. |
| 5 | OAS-PAGE-01 требует курсорную пагинацию для списков. | У клиента не больше двух кошельков (BR-DR-OPEN-04, 05), счетов-источников единицы. Пагинация не следует из требований (OAS-FILE-07), схемы названы `WalletList` и `SourceAccountList`, а не `*Page`. |

## 9. Новые бизнес-коды

Нет. Все коды из каталога `knowledge/business-rules.md` §9.

## 10. Открытые вопросы

1. Какие данные входят в подписываемое распоряжение и в каком формате подпись (CMS, raw ГОСТ Р 34.10-2012)? Влияет на `signature` и проверку в модуле (A-08).
2. Как модуль сверяет блокировку, если INT-ABS-03 не ответила за 1 с? В АБС нет операции «статус блокировки по id операции». Влияет на ответ 504 и на риск «зависшей» блокировки (A-05).
3. Сколько раз модуль повторно отправляет INT-DRP-06 после ответа INT-DRP-08 «не получено»? Сейчас взято 3, как для INT-DRP-03 (A-01).
4. Какой код вернуть при пополнении кошелька в `CLOSING` или `CLOSED`? В каталоге нет отдельного кода (A-04).
5. Что делать, если INT-ABS-04 или INT-ABS-05 недоступна: оставить операцию в текущем статусе до доработки или ввести отдельный статус? От ответа зависит, когда клиент увидит `COMPLETED` (A-09).
6. Показывать ли в списке счетов счета с ограничениями на расход, чтобы клиент видел причину отказа? Сейчас скрыты (A-02).
7. Какой `errorCode` ставить, если платформа отказала в зачислении по причине вне таблицы соответствия? В каталоге нет общего кода отказа зачисления.
8. NFR-PERF-01 (2 с p95) для `createWalletTopUp`: до ответа идут до 6 синхронных вызовов (INT-DRP-04, 02, 05, INT-ABS-02, 03, INT-AF-02). В худшем случае с повторами `RP-EXP` время превысит 2 с. Нужен ли отдельный таймаут клиента, как NFR-PERF-05 для ЕСИА?

## 11. Проверка человеком

<!-- Заполняет проверяющий -->
