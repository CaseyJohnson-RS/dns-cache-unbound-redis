# 1.1A. Кэширование ответа Yandex.ru (адресная запись) — проверка истечения времени кэширования

Задача: показать, что Unbound кэширует A-запись `yandex.ru`, и что после истечения TTL запись пропадает из кэша.

## Шаг 1. Первый запрос — добавление в кэш

Выполним запрос к нашему резолверу:

```bash
dig @127.0.0.1 yandex.ru A
```

В выводе смотрим на поле `TTL` в секции `ANSWER` — это время, на которое Unbound закэшировал запись.

<div align="center">
  <img src="assets/images/STEP1_1A_1.png" width="50%" />
</div>

## Шаг 2. Второй запрос — подтверждение кэширования

Через несколько секунд повторяем тот же запрос:

```bash
dig @127.0.0.1 yandex.ru A
```

Значение TTL должно уменьшиться — это означает, что ответ пришёл из кэша, а не от авторитетного сервера.

<div align="center">
  <img src="assets/images/STEP1_1A_2.png" width="50%" />
</div>

## Шаг 3. Просмотр состояния кэша через unbound-control

```bash
sudo unbound-control dump_cache | grep yandex
```

Видим запись с оставшимся TTL.

<div align="center">
  <img src="assets/images/STEP1_1A_3.png" width="50%" />
</div>

## Шаг 4. Ожидание истечения TTL

Ждём, пока TTL истечёт (или принудительно сбросим кэш для демонстрации):

```bash
sudo unbound-control flush yandex.ru
```

Проверяем, что запись удалена из кэша:

```bash
sudo unbound-control dump_cache | grep yandex
```

Вывод должен быть пустым.

<div align="center">
  <img src="assets/images/STEP1_1A_4.png" width="50%" />
</div>

## Шаг 5. Запрос после истечения TTL

```bash
dig @127.0.0.1 yandex.ru A
```

Теперь Unbound снова выполняет рекурсивный запрос к авторитетному серверу — TTL в ответе будет исходным (не уменьшенным).

<div align="center">
  <img src="assets/images/STEP1_1A_5.png" width="50%" />
</div>

Мы видим, что начальный TTL для всех записей равен 600 секунд.