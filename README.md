**Домашнее задание №6**

**Тема** ***"Управление пакетами. Дистрибьция софта"***

**Задача**
 - определить алгоритм с наилучшим сжатием
 - определить настройки pool’a
 - найти сообщение от преподавателей
 - составить список команд, которыми получен результат с их выводами

---
**Результат выполнения задания**
1. Поднят `Vagranfile`: 

```
vagrant status 
Current machine states:

zfs                       running (virtualbox)

The VM is running. To stop this VM, you can run `vagrant halt` to
shut it down forcefully, or you can run `vagrant suspend` to simply
suspend the virtual machine. In either case, to restart it again,
simply run `vagrant up`.
```

2. Для проведения эксперимента создан скрипт ansible, позволяющий:
    - создать 4 zpool 
    - установить алгоритм сжатия для zpool
    - скачать тестовый файл в каждый pool
    - 

3. После запуска скрипта, проверено:

- Информация о созданных zpool

```
[root@zfs ~]# zpool list
NAME    SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP    HEALTH  ALTROOT
otus1   480M  21.7M   458M        -         -     0%     4%  1.00x    ONLINE  -
otus2   480M  17.7M   462M        -         -     0%     3%  1.00x    ONLINE  -
otus3   480M  10.8M   469M        -         -     0%     2%  1.00x    ONLINE  -
otus4   480M  39.3M   441M        -         -     0%     8%  1.00x    ONLINE  -
```

- Проверка установленных алгоритмов сжатия, zpool

```
[root@zfs ~]# zfs get all | grep compression
otus1  compression           lzjb                   local
otus2  compression           lz4                    local
otus3  compression           gzip-9                 local
otus4  compression           zle                    local
```

- Проверка скачанных тестовых файлов
```
[root@zfs ~]# ls -l /otus*
/otus1:
total 22078
-rw-r--r--. 1 root root 41043469 May 11 21:12 pg2600.converter.log

/otus2:
total 17998
-rw-r--r--. 1 root root 41043469 May 11 21:13 pg2600.converter.log

/otus3:
total 10962
-rw-r--r--. 1 root root 41043469 May 11 21:13 pg2600.converter.log

/otus4:
total 40109
-rw-r--r--. 1 root root 41043469 May 11 21:14 pg2600.converter.log
```

- Проверка, сколько места занимает один и тот же файл в разных пулах и проверим степень сжатия файлов:
```
[root@zfs ~]# zfs list
NAME    USED  AVAIL     REFER  MOUNTPOINT
otus1  21.7M   330M     21.6M  /otus1
otus2  17.7M   334M     17.6M  /otus2
otus3  10.8M   341M     10.7M  /otus3
otus4  39.3M   313M     39.2M  /otus4
```
Таким образом видно, что алгоритм **gzip-9** самый эффективный по сжатию. 

- 