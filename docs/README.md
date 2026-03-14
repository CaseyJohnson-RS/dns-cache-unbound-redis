# Практическое руководство по выполнению кейса

## 0. Подготовка среды

#### Установка **Ubuntu server 22.04** в Virtual Box.

 - 2 ядра
 - 4 ГБ RAM
 - Проверяем сетевой адаптер NAT
 - После установки выполняем `sudo apt update && sudo apt upgrade -y`

> [Сылка на страницу с образом](https://www.linuxvmimages.com/images/ubuntuserver-2204/)

![alt text](assets/image.png)

#### Установка Unbound и утилит

```bash
sudo apt install -y unbound unbound-host dnsutils redis-server
```

 - `unbound` - наш DNS-резолвер
 - `unbound-host` - упрощённая утилита для DNS-запросов
 - `dnsutils` - пакет с утилитой `dig` (основной инструмент, которым будем пользоваться для диагностики)
 - `redis-server` - внешний кэш

> Подробнее про каждую можно почитать [тут](docs/UTILS.md)