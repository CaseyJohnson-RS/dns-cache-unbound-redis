# Утилиты

Краткий справочник по инструментам, используемым в кейсе.

---

## dig

`dig` (Domain Information Groper) — основной инструмент для DNS-запросов и диагностики. Входит в пакет `dnsutils`.

### Базовый синтаксис

```
dig [@сервер] <домен> [тип_записи] [опции]
```

### Примеры

```bash
# A-запись (IPv4-адрес) через локальный резолвер
dig @127.0.0.1 yandex.ru A

# A-запись через публичный DNS (Google) — для сравнения
dig @8.8.8.8 yandex.ru A

# Только IP, без лишнего вывода
dig @127.0.0.1 yandex.ru A +short

# Запрос с DNSSEC (флаг "ad" в ответе = валидация прошла)
dig @127.0.0.1 iana.org A +dnssec

# Трассировка рекурсивного поиска (корень → TLD → авторитетный сервер)
dig @127.0.0.1 iana.org A +trace

# Другие типы записей
dig @127.0.0.1 iana.org AAAA   # IPv6
dig @127.0.0.1 iana.org NS     # серверы имён
dig @127.0.0.1 iana.org MX     # почтовые серверы
dig @127.0.0.1 iana.org TXT    # текстовые записи
dig @127.0.0.1 iana.org SOA    # начало зоны
dig @127.0.0.1 iana.org DNSKEY # ключи DNSSEC
```

### Что смотреть в выводе

```
;; ANSWER SECTION:
yandex.ru.    295    IN    A    77.88.55.70
              ^^^
              TTL в секундах — убывает при повторных запросах из кэша
```

| Поле в ответе | Что означает |
|---|---|
| TTL убывает (300 → 295 → ...) | Ответ из кэша резолвера |
| TTL снова полный (300) | Кэш истёк, запись получена заново |
| Флаг `ad` в `flags:` | DNSSEC-валидация прошла успешно |
| `SERVFAIL` в статусе | Ошибка валидации DNSSEC (или нет ответа) |
| `NXDOMAIN` в статусе | Домен не существует |

---

## unbound-control

Утилита управления работающим Unbound-резолвером. Позволяет перезагружать конфиг, просматривать и чистить кэш без перезапуска службы.

> Для работы Unbound должен быть запущен.

### Команды

```bash
# Перезагрузить конфиг (без потери кэша)
sudo unbound-control reload

# Посмотреть весь кэш
sudo unbound-control dump_cache

# Найти конкретную запись в кэше
sudo unbound-control dump_cache | grep yandex.ru

# Удалить одну запись из кэша
sudo unbound-control flush yandex.ru

# Полностью очистить кэш (все записи зоны)
sudo unbound-control flush_zone .

# Статистика резолвера
sudo unbound-control stats_noreset
```

### Конфиг Unbound

Основной файл конфига: `/etc/unbound/unbound.conf`
(При сборке из исходников: `/usr/local/etc/unbound/unbound.conf`)

Структура:
```yaml
server:
    # настройки сервера

cachedb:
    # настройки внешнего кэша (Redis)
```

Проверить, что конфиг валиден:
```bash
sudo unbound-checkconf
```

---

## redis-cli

Консольный клиент для взаимодействия с Redis. Используется для проверки, что DNS-записи попали во внешний кэш.

### Команды

```bash
# Проверить, что Redis работает
redis-cli ping           # → PONG

# Посмотреть все ключи в БД
redis-cli keys '*'

# Найти ключи, связанные с конкретным доменом
redis-cli keys '*yandex*'

# Получить значение по ключу (бинарные данные DNS-записи)
redis-cli get "<ключ>"

# Посмотреть TTL записи в Redis (-1 = бессрочно, число = секунды)
redis-cli TTL "<ключ>"

# Удалить ключ вручную
redis-cli del "<ключ>"

# Показать количество ключей в БД
redis-cli dbsize
```

### Пример: проверить первую запись после dig

```bash
redis-cli get "$(redis-cli keys '*' | head -1)"
```

### Конфиг Redis

Файл: `/etc/redis/redis.conf`

Ключевые параметры для кейса:

```
# Сохранение на диск (persistence)
save 900 1       # сохранить если 1 изменение за 900 сек
save 300 10      # сохранить если 10 изменений за 300 сек
save 60 10000    # сохранить если 10000 изменений за 60 сек
```

После изменения конфига:
```bash
sudo systemctl restart redis-server
```

---

## systemctl (управление службами)

```bash
# Статус
sudo systemctl status unbound
sudo systemctl status redis-server

# Запуск / остановка / перезапуск
sudo systemctl start unbound
sudo systemctl stop unbound
sudo systemctl restart unbound

# Включить / отключить автозапуск
sudo systemctl enable unbound
sudo systemctl disable unbound
```

> При работе с **Unbound из исходников** (`/usr/local/sbin/unbound`) службы systemctl не применяются — запуск вручную:
> ```bash
> sudo /usr/local/sbin/unbound -c /usr/local/etc/unbound/unbound.conf
> ```


# TODO

Добавить описание команд

1. dig yandex.ru A +norecurse @ns1.yandex.ru