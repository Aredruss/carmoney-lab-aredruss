```
git -C /Users/aredruss/carmoney-lab/.kilo/worktrees/adaptable-soprano branch --show-current && git -C /Users/aredruss/carmoney-lab/.kilo/worktrees/adaptable-soprano rev-parse --show-toplevel
adaptable-soprano
/Users/aredruss/carmoney-lab/.kilo/worktrees/adaptable-soprano

Tests in `tests/Unit/`:

- `LtvCalculatorTest.php` — проверяет расчёт LTV (корректные значения и исключения при невалидных amount/marketValue).
- `DecisionEngineTest.php` — проверяет выбор решения (`approve`/`review`/`reject`) по LTV на границах порогов.
- `VinValidatorTest.php` — проверяет формат VIN (длина 17, регистр, запрещённые буквы I/O/Q, спецсимволы, пустая строка).
- `ApplicationValidatorTest.php` — проверяет валидацию заявки: нормализация VIN, отказ по будущему году, минимуму суммы и сбор всех ошибок сразу.
- `AssessmentServiceTest.php` — проверяет сквозной сценарий `assess()`: низкий LTV → `approve` с лимитом, средний → `review` с нулём, высокий → `reject`.

Работаю в `/Users/aredruss/carmoney-lab/.kilo/worktrees/adaptable-soprano` на ветке `adaptable-soprano`.


areduss@Aredusss-MacBook-Pro carmoney-lab % git worktree list
/Users/aredruss/carmoney-lab                                    de56e11 [d1/1.2.1-1.2.3-aredruss]
/Users/aredruss/carmoney-lab/.kilo/worktrees/adaptable-soprano  de56e11 [adaptable-soprano]
/Users/aredruss/carmoney-lab/.kilo/worktrees/gregarious-flower  de56e11 (detached HEAD)
areduss@Aredusss-MacBook-Pro carmoney-lab % 
```