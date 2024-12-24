# Цель лабораторной работы
Разработать патч для сетевого драйвера Beaglebone Black (BBB). 
Патч должен добавлять следующий функционал:
1) Печать в log номер прерывания драйвера
2) Печать в log адреса области ввода-ввывода
3) Печать в log размера области
4) Печать в log содержимого и размера данных входящих/исходящих данных
Патч необходимо добавить в buildroot и протестировать его.
# Выполнение
## Написание патча
На BBB запущена команда:
`ls -l /sys/class/net/eth0/device/driver`

Полученный вывод команды:
`lrwxrwxrwx    1 root    root            0 Jan 1 1970 /sys/class/net/eth0/device/driver -> ../../../../../../../bus/platform/drivers/cpsw-switch`

Из этого видно, что за сеть отвечает драйвер cpsw-switch.
В исходниках ядра можно найти файл cpsw_new.c
Добавляем отладочный вывод в log в код, записываем новую версию драйвера в файл. Чтобы получить патч, вводим команду:
`diff -p cpsw_new.c cpsw_new_patch.c > cpsw_new.patch`

## Установка в buildroot
Кладем патч в директорию buildroot, сообщаем путь до патча. Компилируем ядро, скомпилированный образ сохраняем на SD-карту. 

## Вывод ядра
При запуске появляется следующий вывод:
```
[    0.504666] cpsw_probe called
[    0.505094] IRQ rx number: 29
[    0.505268] IRQ tx number: 2a
[    0.505320] IRQ misc number: 2b
[    0.505451] IO region: 0, size: 4000
```
Видно, что отладочные данные выводятся. 
Запустим сам сетевой интерфейс, чтобы проверить вывод данных о размере пакетов.
`echo auto eth0 >> /etc/network/interfaces`
`echo iface eth0 inet dhcp >> /etc/network/interfaces`
`ifup eth0`
Получаем следующий лог:
```
[ 2466.855816] Outgoing packet data: caf11f20, size: 70                        
[ 2467.078200] Incoming packet data: b3274ac1, size: 46                        
[ 2467.104441] Incoming packet data: f89012d3, size: 46                        
[ 2467.120104] Incoming packet data: 6479ca23, size: 46                        
[ 2467.552187] Incoming packet data: 9f3b12ab, size: 270
```