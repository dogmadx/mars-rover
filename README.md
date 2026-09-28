# Марсоход — решение команды

Цель — чтобы ровер проехал как можно дальше. Сервер соберёт образ по
`Dockerfile`, запустит `train.py` (не дольше ~2 часов) и возьмёт модель из
`/output/policy.onnx`. На проверке 48 заездов на закрытых биомах по 300 секунд
максимум; балл — медиана максимальных расстояний.

Репозиторий — одна рабочая папка: серверное стартовое решение плюс публичная
среда организаторов (все тренировочные биомы, GUI). Исходные версии обеих
папок в том виде, как их прислали, лежат под тегом `upstream-v0.16`.

## Что можно менять

| Файл | Для чего |
|---|---|
| `model.py` | Сеть: 160 входов → 31 действие, с памятью |
| `train.py` | Параметры PPO, `TOTAL_FRAMES`, лимит времени (7140 с) |
| `python/mars_rover_env/configs/env.yaml` | Параметры тренировочного мира |
| `cpp/include/mars/custom_biomes.inc.hpp` | Свои тренировочные биомы |
| `cpp/include/mars/biomes/*.inc.hpp` | Публичные тренировочные биомы |

## Что НЕ менять

Серверный контракт: `cpp/include/mars/action.hpp`,
`python/mars_rover_env/actions.py`, наблюдения, `rover_rig.yaml` и конструкцию
ровера, а также физику в `cpp/src/`. На сервере они свои, и изменения только
рассинхронизируют обучение с проверкой.

## Сборка и проверка

Нужны Python 3.10+, компилятор C++20, `setuptools`, `wheel`, `pybind11`, `torch`.

```bash
python -m pip install setuptools wheel pybind11 numpy pyyaml torch
make build                                   # собрать C++ среду
python -m unittest discover -s tests -v      # smoke-тесты
python -c "import _mars_rover_cpp as m; print([b['id'] for b in m.biome_catalog()])"
```

После обучения модель проверяется командой `python check_policy.py путь/к/policy.onnx`
(нужен модуль `arena` из серверного образа).

## GUI

```bash
make gui
mars-rover-play --list-biomes
mars-rover-play --biome gravity_shelf_lug --debug
```

## Добавление тренировочного биома

1. Откройте `cpp/include/mars/custom_biomes.inc.hpp`.
2. Скопируйте `ExampleTrainingBiome`, задайте уникальные `id()` и
   `display_name()`, настройте `sample_params()` и `visuals()`.
3. Оставьте `split()` равным `BiomeSplit::Train`.
4. Добавьте статический экземпляр в `custom_biomes::append`.
5. Выполните `make build` и проверьте каталог командой выше.

## Отправка

```bash
python -m unittest discover -s tests -v
python package_submission.py      # → submission.zip
```

В архив попадают только корневые файлы решения и папки `cpp/`, `python/`,
`tests/`; `gui/` и прочее локальное не уходит. `make check` собирает Docker-образ
как сервер, но для этого нужен образ `arena-base`.

После каждой отправки ставьте тег `sub-NN` и записывайте балл ниже.

| Тег | Что изменили | Балл |
|---|---|---|
| | | |
