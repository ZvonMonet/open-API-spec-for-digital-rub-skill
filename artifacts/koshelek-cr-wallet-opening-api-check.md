# Отчёт проверки: koshelek-cr-wallet-opening-api.yaml

| Параметр | Значение |
|---|---|
| Артефакт | `artifacts/koshelek-cr-wallet-opening-api.yaml` (OpenAPI 3.0.3, API 1.0.0) |
| Task | SCRUM-2194 «КошелёкЦР - открытие кошелька цифрового рубля» |
| Story | SCRUM-2196 (связь *relates to*, единственная) |
| Продукт / контекст | `knowledge/context/koshelek-cr/` |
| Подготовлено | Агентом (skill `oas-from-jira`), 2026-11-19. Черновик для проверки человеком. |

> Шаблона `skills/oas-from-jira/templates/artifact-check.md` в репозитории нет, поэтому отчёт составлен по разделам, перечисленным в шаге 9 SKILL.md.

## 1. Источники

| Источник | Как использован |
|---|---|
| SCRUM-2194 (Jira, Atlassian MCP) | Границы, сценарий `[UI]`/`[API]`/`[EXT]`, альтернативы A1–A10, E1, ключевые данные, НФТ |
| SCRUM-2196 (Jira, Atlassian MCP) | AC-01…AC-19, подтверждённые решения, допущение про приложение для ИП |
| `knowledge/context/koshelek-cr/functional-requirements.md` | FR-GEN-*, FR-OPEN-01…13 |
| `knowledge/context/koshelek-cr/non-functional-requirements.md` | NFR-PERF-01…03, NFR-AV-02, NFR-AV-04, NFR-RATE-01, NFR-RATE-03, NFR-RET-01, NFR-OBS-01, 02 |
| `knowledge/context/koshelek-cr/integrations.md` | INT-ABS-01, INT-ESIA-01/02, INT-DRP-01…04, INT-AF-01, таблицы соответствия ошибок |
| `knowledge/business-rules.md` | BR-* из Task, статусные модели §8.1, §8.2, каталог кодов §9 |
| `knowledge/sensitive-data.md` | §3 (каталог полей), §4 (MASK-WALLET), §5 (SD-API-*), §9 (`x-sensitivity`) |
| `knowledge/oas-conventions.md` | Оформление целиком |

## 2. Результат Spectral

```
npx @stoplight/spectral-cli@6 lint artifacts/koshelek-cr-wallet-opening-api.yaml --ruleset .spectral.yaml
Spectral 6.17.1 — No results: 0 errors, 0 warnings
```

`.spectral.yaml` не менялся. Во время проверки найдена и исправлена одна ошибка `oas3-valid-media-example`: в ответе 422 у `createWalletApplication` стоял `oneOf`, а пример `CLIENT_NOT_ELIGIBLE` подходил под обе схемы. `oneOf` заменён на `anyOf`.

## 3. Операции

| # | Метод и путь | operationId | Шаг Task |
|---|---|---|---|
| 1 | `POST /wallet-applications` | `createWalletApplication` | 2 |
| 2 | `GET /wallet-applications/{applicationId}` | `getWalletApplication` | 11 (и переходы на стороне сервера: 4, 5, 10, A10) |
| 3 | `POST /wallet-applications/{applicationId}/esia-authorization` | `createEsiaAuthorization` | 3 |
| 4 | `GET /wallet-applications/{applicationId}/contract` | `getWalletApplicationContract` | 6 |
| 5 | `POST /wallet-applications/{applicationId}/contract-acceptance` | `acceptWalletApplicationContract` | 8 |
| 6 | `POST /wallet-applications/{applicationId}/certificate-requests` | `submitCertificateRequest` | 10 |
| 7 | `POST /wallet-applications/{applicationId}/cancel` | `cancelWalletApplication` | A6 / AC-12 |
| 8 | `GET /wallets/{walletId}` | `getWallet` | 13 |

Шаги `[UI]` 1, 4, 7, 9, 12 операций не дают. Шаг 5 (`[EXT:DR_PLATFORM]` без `[API]`) выполняет сервер; клиент видит его как переход статуса в `getWalletApplication`.

## 4. Матрица покрытия AC

| AC | Элемент OAS | Покрыт |
|---|---|---|
| AC-01 | `createWalletApplication` → 201, `WalletApplication` (`id`, `ownerType`, `status=CREATED`, `expiresAt`) | Да |
| AC-01a | `createWalletApplication`, `ownerType=SOLE_PROPRIETOR` → 201; в description: кошелёк другого типа открытию не мешает | Да |
| AC-01b | `createWalletApplication` → 422 `CLIENT_NOT_ELIGIBLE`, `reason=NOT_SOLE_PROPRIETOR` (`ClientNotEligibleProblem`) | Да |
| AC-02 | `createWalletApplication` → 422 `CLIENT_NOT_ELIGIBLE`, `reason=AGE` / `CITIZENSHIP` | Да |
| AC-03 | `createWalletApplication` → 409 `WALLET_ALREADY_EXISTS` + `ownerType` (`WalletAlreadyExistsProblem`) | Да |
| AC-04 | `createWalletApplication` → 409 `APPLICATION_IN_PROGRESS` + `applicationId` (`ApplicationInProgressProblem`) | Да |
| AC-05 | `createWalletApplication`: `Idempotency-Key` обязателен; повтор возвращает ту же заявку (201); другое тело → 422 `IDEMPOTENCY_KEY_REUSED` | Да |
| AC-06 | `createEsiaAuthorization` → 200 `EsiaAuthorization` (`authorizationUrl`, `status=ESIA_PENDING`); 409 `INVALID_APPLICATION_STATE` не из `CREATED` | Да |
| AC-07 | `getWalletApplication`: `status=REJECTED`, `rejectReason` `ESIA_CONSENT_DENIED` / `ESIA_ACCOUNT_NOT_CONFIRMED` | Да |
| AC-08 | `getWalletApplication`: `status=REJECTED`, `rejectReason=WALLET_ALREADY_EXISTS` | Да |
| AC-09 | `getWalletApplicationContract` → 200 `WalletContract` (`contractId`, `contractVersion`, `documentUrl`, `platformRulesUrl`, `tariffsUrl`) | Да |
| AC-10 | `acceptWalletApplicationContract` → 200, `status=KEYS_PENDING`, `acceptedContractVersion`, `contractAcceptedAt` | Да |
| AC-11 | `acceptWalletApplicationContract` → 409 `CONTRACT_VERSION_OUTDATED` | Да |
| AC-12 | `cancelWalletApplication` → 200, `status=CANCELLED`; 409 `INVALID_APPLICATION_STATE` после передачи CSR | Да |
| AC-13 | `submitCertificateRequest` → 202 + `Location`, `status=PROCESSING`; в запросе только `csr` (`writeOnly`) и `keyAlgorithm` | Да |
| AC-14 | `getWalletApplication`: `ON_REVIEW` / `REJECTED` с `rejectReason=OPERATION_DECLINED_BY_SECURITY` (без деталей) | Да |
| AC-15 | `getWalletApplication`: `status=COMPLETED`, `walletId`. Передача сведений об ИП на платформу описана в `submitCertificateRequest` (внутренний вызов, в контракте не виден) | Да |
| AC-16 | `getWallet` → 200 `Wallet` (`ownerType`, `status=ACTIVE`, `walletNumberMasked`, `balance` 0.00 RUB, `openedAt`) | Да |
| AC-17 | 503 `ServiceUnavailable` (`PLATFORM_UNAVAILABLE` / `CORE_BANKING_UNAVAILABLE`) и 504 `GatewayTimeout` у операций с внешними вызовами; «статус не меняется, повторить позже» — в описании ответов и операций | Да |
| AC-18 | 404 `NotFound` (`RESOURCE_NOT_FOUND`) у всех операций с `applicationId` | Да |
| AC-19 | `WalletApplication.expiresAt`, статус `EXPIRED`; дальнейшие шаги → 409 `INVALID_APPLICATION_STATE` | Да |

Все AC закрыты. AC, отнесённых к категории «вне OAS», нет.

## 5. Трассировка элементов

### 5.1 Операции, ответы, заголовки

| Элемент OAS | Источник |
|---|---|
| `createWalletApplication` | Task шаг 2; AC-01…05; FR-OPEN-01, 02 |
| ↳ 403 `Forbidden` (`IDENTIFICATION_REQUIRED`, `CLIENT_BLOCKED`) | Task, предусловия (BR-GEN-03, 04); integrations: соответствие ошибок АБС |
| ↳ 409, 422 | AC-01b, 02, 03, 04, 05; BR-IDM-02 |
| ↳ 429 + `Retry-After` | NFR-RATE-01 (5 заявок в час) |
| ↳ 503 / 504 | Внешний вызов АБС (INT-ABS-01), integrations: `CORE_BANKING_UNAVAILABLE` / `UPSTREAM_TIMEOUT`; AC-17 |
| ↳ `Idempotency-Key` | POST создаёт ресурс; AC-05; BR-IDM-01; NFR-AV-04 (24 ч) |
| ↳ 201 + `Location` | OAS-HDR-04 |
| `getWalletApplication` | Task шаг 11; AC-07, 08, 14, 15, 18, 19; FR-OPEN-10; NFR-PERF-03 (интервал 2 с, 60 с) |
| ↳ 429 | NFR-RATE-03 (не чаще 1 раза в секунду на ресурс) |
| ↳ без 503/504 | Внешние системы не вызываются |
| `createEsiaAuthorization` | Task шаг 3; AC-06; FR-OPEN-03; INT-ESIA-01 |
| ↳ `Idempotency-Key`, 422 `IDEMPOTENCY_KEY_REUSED` | Task E1 (повтор с тем же ключом для шагов 3, 6, 10); BR-IDM-02 |
| ↳ 503 / 504 | Task E1; integrations (ЕСИА → `PLATFORM_UNAVAILABLE`); AC-17 |
| `getWalletApplicationContract` | Task шаг 6; AC-09; FR-OPEN-05; INT-DRP-02 |
| ↳ 409 `INVALID_APPLICATION_STATE` | business-rules §8.1 (шаг не в своём статусе) |
| ↳ 503 / 504 | Task E1; AC-17 |
| `acceptWalletApplicationContract` | Task шаг 8; AC-10, 11; FR-OPEN-06; BR-DR-OPEN-07, 08, 09 |
| ↳ 503 / 504 | Допущение A-03 (сверка версии на платформе) |
| `submitCertificateRequest` | Task шаг 10; AC-13…15; FR-OPEN-07…09; INT-ABS-01, INT-AF-01, INT-DRP-03 |
| ↳ 202 + `Location` | NFR-PERF-02; OAS-ASYNC-01 |
| ↳ `Idempotency-Key` | Task E1 |
| ↳ 503 / 504 | Task E1; AC-17; integrations (АБС, платформа) |
| `cancelWalletApplication` | AC-12; Task A6; FR-OPEN-11; business-rules §8.1 |
| `getWallet` | Task шаг 13; AC-16; FR-OPEN-13; INT-DRP-04 |
| ↳ `Cache-Control: no-store` | SD-API-10 (баланс и номер кошелька — C2) |
| ↳ 503 / 504 | Внешний вызов платформы (INT-DRP-04); AC-17 |
| 400 / 401 / 500 у всех операций | OAS-ERR-04 |
| `X-Request-Id` (параметр и заголовок ответа) | OAS-HDR-02; NFR-OBS-01 |

### 5.2 Схемы и поля

| Элемент | Источник |
|---|---|
| `OwnerType` (`INDIVIDUAL`, `SOLE_PROPRIETOR`) | Task, границы; BR-DR-OPEN-05; FR-GEN-04 |
| `WalletApplicationStatus` (11 значений) | business-rules §8.1 (полная модель, OAS-TYPE-08) |
| `CreateWalletApplicationRequest.ownerType` | Task шаг 2; AC-01, 01a |
| `WalletApplication.id`, `ownerType`, `status`, `createdAt`, `updatedAt` | Task, ключевые данные |
| `WalletApplication.rejectReason` | Task, ключевые данные; AC-07, 08, 14; Task A9; integrations (платформа: `CERTIFICATE_ISSUE_FAILED`, `WALLET_OPENING_REFUSED`) |
| `WalletApplication.walletId` | AC-15; Task шаг 11 |
| `WalletApplication.acceptedContractVersion`, `contractAcceptedAt` | AC-10 («фиксируется с версией и временем») |
| `WalletApplication.expiresAt` | AC-01; AC-19; NFR-RET-01 |
| `EsiaAuthorization` (`applicationId`, `status`, `authorizationUrl`) | AC-06 |
| `WalletContract` (5 полей) | AC-09; Task, ключевые данные |
| `AcceptWalletContractRequest.contractId`, `contractVersion` | AC-10; BR-DR-OPEN-08 |
| `AcceptWalletContractRequest.accepted` (`enum: [true]`) | BR-DR-OPEN-07 |
| `SubmitCertificateRequestRequest.csr`, `keyAlgorithm` | Task, ключевые данные («CSR, алгоритм ключа»); AC-13; BR-DR-OPEN-10 |
| `Wallet` (`id`, `ownerType`, `status`, `walletNumberMasked`, `balance`, `openedAt`) | AC-16; Task, ключевые данные; business-rules §8.2 |
| `Money` | `Wallet.balance`; OAS-TYPE-04 |
| `Problem` | OAS-ERR-01, 02 |
| `ClientNotEligibleProblem.reason` | AC-01b («с причиной»); BR-DR-OPEN-01 (`AGE` / `CITIZENSHIP` / `NOT_SOLE_PROPRIETOR`) |
| `WalletAlreadyExistsProblem.ownerType` | AC-03 («с указанием типа») |
| `ApplicationInProgressProblem.applicationId` | AC-04 |
| `components.responses.InvalidApplicationState`, `IdempotencyKeyReused` | Общие для нескольких операций (OAS-ERR-05) |

### 5.3 Чувствительные данные

| Поле | Уровень | Решение |
|---|---|---|
| `Wallet.walletNumberMasked` | C2, BANK_SECRECY | `MASK-WALLET`; полный номер не передаётся (SD-API-04) |
| `Wallet.balance` | C2, BANK_SECRECY | Полностью владельцу, в логах `MASK-HIDE`; ответ с `Cache-Control: no-store` |
| `SubmitCertificateRequestRequest.csr` | C1 | `writeOnly` (sensitive-data §3.3) |
| Данные и токены ЕСИА | C3 | В API отсутствуют (SD-API-16) |
| Закрытый ключ ЭП, пароль кошелька | C3 | В API отсутствуют (BR-DR-OPEN-10, Story «Подтверждённые решения») |
| Пути | — | Только UUID (`applicationId`, `walletId`); query-параметров нет |

## 6. Допущения

| ID | Допущение | Где отражено |
|---|---|---|
| A-01 | ИП открывает кошелёк для предпринимательской деятельности в том же мобильном приложении (допущение из Story и Task). | `info.description` |
| A-02 | CSR передаётся как PKCS#10 DER в base64 (`format: byte`, до 8192 символов). Алгоритмы ключа: `GOST_R_34_10_2012_256`, `GOST_R_34_10_2012_512`. В требованиях формат и перечень не заданы. | `x-assumption` у `csr`, `keyAlgorithm` |
| A-03 | При принятии договора модуль сверяет версию с актуальной на платформе ЦР. Отсюда `x-external-systems: [DR_PLATFORM]` и ответы 503/504. | `x-assumption` у `acceptWalletApplicationContract` |
| A-04 | В `submitCertificateRequest` до ответа 202 синхронно выполняются получение сведений об ИП из АБС и передача запроса на платформу (иначе нечем объяснить E1 и AC-17). Антифрод и результат платформы обрабатываются асинхронно (AC-14 говорит о заявке уже в `PROCESSING`). | `x-assumption` у `submitCertificateRequest` |
| A-05 | Маскированный номер кошелька: `**** XXXX` (последние 4 символа). Формат номера не подтверждён. | `x-assumption` у `walletNumberMasked` |
| A-06 | Повтор `createWalletApplication` с тем же ключом возвращает 201 с той же заявкой, а не 200. | description операции |
| A-07 | Шаг 6 (договор) — `GET`, он идемпотентен и без `Idempotency-Key`; в E1 повтор «с тем же ключом» для этого шага неприменим. | description операции |
| A-08 | Договор — подресурс-синглтон `/contract` (единственное число), принятие — `/contract-acceptance`, отмена — `POST .../cancel` (OAS-PATH-04). | Пути |

## 7. Противоречия между источниками

| # | Противоречие | Решение |
|---|---|---|
| 1 | Task E1 называет недоступность внешних систем только для шагов 3, 6, 10. По integrations.md шаг 2 синхронно вызывает АБС, у которой есть ошибки `CORE_BANKING_UNAVAILABLE` / `UPSTREAM_TIMEOUT`, а шаг 13 вызывает платформу (INT-DRP-04). | По правилу SKILL.md (503/504 у операций с внешним вызовом по integrations.md) 503/504 добавлены в `createWalletApplication` и `getWallet`. |
| 2 | Task, таблица внешних систем: шаг 8 к платформе не обращается. integrations.md: «Версия договора неактуальна» — результат платформы, 409 при принятии договора. | Взята сверка на платформе (допущение A-03), вынесено в открытые вопросы. |
| 3 | Статусная модель §8.1 содержит `FAILED` с полем `errorCode`, в Task и Story этого статуса нет. | `FAILED` включён в enum, потому что OAS-TYPE-08 требует полного перечня. Поле `errorCode` не добавлено: значения не определены, в AC оно не требуется. Вынесено в открытые вопросы. |
| 4 | Task шаг 10: «антифрод → платформа» внутри одного вызова. AC-14: решение антифрода приходит для заявки в `PROCESSING`. | Антифрод выполняется асинхронно, после ответа 202 (допущение A-04). |
| 5 | §8.1: из `ON_REVIEW` в `PROCESSING` после проверки; в Task A8 нет пути обратно из `ON_REVIEW`. | Описано по модели §8.1. |
| 6 | Story AC-02 не требует поле причины для возраста и гражданства, BR-DR-OPEN-01 его задаёт. | `reason` возвращается для всех трёх случаев (по BR-DR-OPEN-01). |

## 8. Новые бизнес-коды

Нет. Все коды взяты из каталога `business-rules.md` §9: `UNAUTHORIZED`, `RESOURCE_NOT_FOUND`, `IDENTIFICATION_REQUIRED`, `CLIENT_BLOCKED`, `CLIENT_NOT_ELIGIBLE`, `ESIA_ACCOUNT_NOT_CONFIRMED`, `ESIA_CONSENT_DENIED`, `WALLET_ALREADY_EXISTS`, `APPLICATION_IN_PROGRESS`, `INVALID_APPLICATION_STATE`, `CONTRACT_VERSION_OUTDATED`, `CERTIFICATE_ISSUE_FAILED`, `WALLET_OPENING_REFUSED`, `OPERATION_DECLINED_BY_SECURITY`, `IDEMPOTENCY_KEY_REUSED`, `VALIDATION_ERROR`, `TOO_MANY_REQUESTS`, `INTERNAL_ERROR`, `PLATFORM_UNAVAILABLE`, `CORE_BANKING_UNAVAILABLE`, `UPSTREAM_TIMEOUT`.

Значения `ClientNotEligibleProblem.reason` (`AGE`, `CITIZENSHIP`, `NOT_SOLE_PROPRIETOR`) взяты из BR-DR-OPEN-01.

## 9. Открытые вопросы

1. **Кошелёк ИП:** нужны ли от клиента дополнительные сведения или документы, кроме данных из АБС? (Task, FR §8 п. 2.) Если нужны, в `CreateWalletApplicationRequest` или в отдельный шаг понадобятся новые поля.
2. **Принятие договора:** сверяется ли версия договора с платформой в момент принятия (A-03), или модуль хранит актуальную версию у себя? От ответа зависят 503/504 у `acceptWalletApplicationContract`.
3. **Формат CSR и алгоритмы ключа ЭП** (A-02): какие требования предъявляет платформа ЦР?
4. **Повторная ссылка на Госуслуги:** что делать, если клиент закрыл страницу Госуслуг или ссылка истекла, а заявка в `ESIA_PENDING`? Сейчас повтор возможен только с тем же `Idempotency-Key`; новый запрос → 409.
5. **`FAILED`:** нужно ли клиенту поле `errorCode` и с какими значениями? Может ли клиент создать новую заявку сразу после `FAILED`?
6. **Возврат из `ON_REVIEW`:** сколько заявка может оставаться в `ON_REVIEW` и истекает ли она через 24 часа? По §8.1 переход `ON_REVIEW → EXPIRED` отсутствует.
7. **Формат номера счёта цифрового рубля** и правило `MASK-WALLET` (sensitive-data §11, вопрос 1).
8. **Уведомление об открытии** (BR-DR-OPEN-12): в Task и Story не упомянуто, callback в OAS не добавлен. Подтвердить, что push — вне клиентского API.
9. **Лица частной практики** (BR-DR-OPEN-06): нужен ли третий `ownerType`?

## 10. Проверка человеком

<!-- Заполняет проверяющий -->
