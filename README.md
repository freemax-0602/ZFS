**Домашнее задание №6**

**Тема** ***"Управление пакетами. Дистрибьция софта"***

**Задача**
 - определить алгоритм с наилучшим сжатием
 - определить настройки pool’a
 - найти сообщение от преподавателей
 - составить список команд, которыми получен результат с их выводами

---
**Результат выполнения задания**
- Поднят `Vagranfile`: 

```
vagrant status 
Current machine states:

zfs                       running (virtualbox)

The VM is running. To stop this VM, you can run `vagrant halt` to
shut it down forcefully, or you can run `vagrant suspend` to simply
suspend the virtual machine. In either case, to restart it again,
simply run `vagrant up`.
```

- Для проведения эксперимента создан скрипт ansible, позволяющий:
    - создать 4 zpool 
    - установить алгоритм сжатия для zpool
    - скачать тестовый файл в каждый pool
    - получает тестовый архив для экспорта
    - 

1. Определение алгоритма с наилучшем сжатием

  **После запуска скрипта, проверено:**

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

2. Определение настроек пула:

- Скачан и распакован архив для экспорта
```
[root@zfs ~]# ls
anaconda-ks.cfg  archive.tar.gz  original-ks.cfg
[root@zfs ~]# tar -xzvf archive.tar.gz
zpoolexport/
zpoolexport/filea
zpoolexport/fileb
```

- Проверка того, можно ли импортировать данный пул в каталог:
```
[root@zfs ~]# zpool import -d zpoolexport/
   pool: otus
     id: 6554193320433390805
  state: ONLINE
 action: The pool can be imported using its name or numeric identifier.
 config:

	otus                         ONLINE
	  mirror-0                   ONLINE
	    /root/zpoolexport/filea  ONLINE
	    /root/zpoolexport/fileb  ONLINE
[root@zfs ~]#
```

- Импорт данного пула к в ОС:

```
[root@zfs ~]# zpool import -d zpoolexport/ otus
[root@zfs ~]# zpool status
  pool: otus
 state: ONLINE
  scan: none requested
config:

	NAME                         STATE     READ WRITE CKSUM
	otus                         ONLINE       0     0     0
	  mirror-0                   ONLINE       0     0     0
	    /root/zpoolexport/filea  ONLINE       0     0     0
	    /root/zpoolexport/fileb  ONLINE       0     0     0

errors: No known data errors
```

3. Работа со снапшотом, поиск сообщения от преподавателя

-  Получен тестовый файл от преподователя:
```
[root@zfs ~]# ls
anaconda-ks.cfg  archive.tar.gz  original-ks.cfg  otus_task2.file  zpoolexport
[root@zfs ~]#
```

- Восстановление файловой системы из снапшота и просмотр сообщения от преподователя
```
[root@zfs ~]# ls
anaconda-ks.cfg  archive.tar.gz  original-ks.cfg  otus_task2.file  zpoolexport
[root@zfs ~]# zfs receive otus/test@today < otus_task2.file
[root@zfs ~]# find /otus/test -name "secret_message"
/otus/test/task1/file_mess/secret_message
[root@zfs ~]# cat /otus/test/task1/file_mess/secret_message
https://otus.ru/lessons/linux-hl/
```
