### Planer
Создал план, но ничего не редактировал

### Scout
Прошелся по коду, четко рассказал, что и где

#### Где читается пробег (mileage) — карта находок

Источник: read-only grep по проекту от scout'а, 2026-09-24.

##### Конфигурация
- `backend/config/rules.php:23` — порог валидации `max_mileage_km => 500000` в секции `vehicle`.

##### Backend — Domain
- `backend/src/Domain/ApplicationValidator.php:43` — нормализация: `$mileage = (int) ($payload['mileage'] ?? -1);`.
- `backend/src/Domain/ApplicationValidator.php:44` — проверка диапазона против `rules.vehicle.max_mileage_km`.
- `backend/src/Domain/ApplicationValidator.php:45` — запись сообщения об ошибке в `$errors['mileage']`.
- `backend/src/Domain/ApplicationValidator.php:78` — пробег попадает в нормализованный выход: `'mileage' => $mileage`.
- `backend/src/Domain/ApplicationValidator.php:22–23` — PHPDoc типа возврата `validate()` включает `mileage:int`.
- В `LtvCalculator`, `DecisionEngine`, `VehicleAge`, `VinValidator` пробег не упоминается.

##### Backend — HTTP
- В `backend/src/Http/` упоминаний `mileage` нет (контроллеры читают поле транзитом).

##### Backend — Repository и БД
- `backend/src/Repository/ApplicationRepository.php:19` — PHPDoc параметра `$input` перечисляет `mileage:int`.
- `backend/src/Repository/ApplicationRepository.php:38–39` — INSERT в `vehicles` использует bind-плейсхолдер `:mileage`.
- `backend/src/Repository/ApplicationRepository.php:45` — биндинг: `':mileage' => $input['mileage']`.
- `backend/src/Repository/ApplicationRepository.php:68` — SELECT-лист для выдачи заявок включает `v.mileage_km`.
- `db/schema.sql:22` — колонка `mileage_km INT UNSIGNED NOT NULL` в `vehicles`.
- `db/seed.sql:31–32` — пример INSERT с `mileage_km = 20000` (LADA Vesta 2025).

##### Frontend
- `frontend/index.html:30–31` — `<label for="mileage">Пробег, км</label>` + `<input id="mileage" name="mileage" type="number" value="84000" required>`.
- `frontend/app.js:8` — `const NUMERIC_FIELDS = ['year', 'mileage', 'market_value', 'requested_amount', 'term_months'];`.

##### Тесты
- `tests/Unit/ApplicationValidatorTest.php:34` — фикстура заявки: `'mileage' => 84000`.
- `tests/Unit/AssessmentServiceTest.php:38` — фикстура заявки: `'mileage' => 96000`.
- `tests/Unit/DecisionEngineTest.php` — упоминаний `mileage` нет.
- `tests/Feature/…` — прямых совпадений по ключевому слову не найдено (фикстуры могут задавать поле под другим именем — уточнять отдельно, если важно).

##### Документация и спека
- `docs/setup/code_map.md:14, 35–36, 38, 42, 47, 51–53` — поведение и узкие места: пробег не влияет на решение; валидатор отбрасывает > 500 000 до решения; точка для правила «> 400 000 → review» — `AssessmentService::assess()` либо расширение `DecisionEngine::decide()`.
- `docs/setup/kilo_hello.md:2` — пробег как поле входной заявки.
- `docs/README.md:4` — эпик дня 1: «пробег ≤ 400 000, иначе `review`» (ID `MILEAGE`).
- `docs/hw1/README.md:14` — заголовок PR: `[ДЗ.1] Пробег ≤ 400000`.
- `docs/plan/README.md:13–14` — обязательные граничные кейсы `399999 / 400000 / 400001, пустой пробег` и риск существующего `max_mileage_km`.
- `docs/plan/plan_COMMENT-index-php.md:32` — `max_mileage_km` и тесты на пробег вне рамок задачи про комментарий.
- `docs/sources/LOAN-12.md:9, 36–37` — ТЗ клиента: пробег в составе заявки; таблица решений по LTV без пробега как фактора.
- `docs/sources/client_note.md:24, 40` — клиентский кейс: пробег 380 000 / 412 300; расхождение одометр-vs-заявка.
- `docs/sources/CASE-08.md:15, 23` — справочник по VIN возвращает пробег; скидка за пробег при расчёте залоговой стоимости.
- `README.md:6, 62, 127` — публичное описание, пример curl `"mileage":84000`, заголовок PR ДЗ.1.
- `AGENTS.md:4` — пробег в описании сервиса.

##### Где не нашлось
- `mocks/`, `scripts/`, `.githooks/`, `.kilo/`, `kilo.jsonc`, `Makefile`, `docker-compose.yml` — совпадений по `mileage`/`пробег` нет.

##### Итог по роли пробега
- **Пишется:** HTTP-слой → JSON → `ApplicationValidator` нормализует/проверяет → `AssessmentService` транзитом → `ApplicationRepository` → INSERT в `vehicles.mileage_km` → SELECT возвращает в карточке заявки.
- **Читается для решения:** нигде. `DecisionEngine::decide(float $ltv)` пробег не получает; `LtvCalculator`, `VehicleAge`, `VinValidator` про него не знают. Кандидатные точки для правила «> 400 000 → review»: `AssessmentService::assess()` после строки с `$decision = …` или расширение `DecisionEngine::decide()` (`docs/setup/code_map.md:33–36`).