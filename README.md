# Домашнее задание к занятию 3    <br>  ***«Резервное копирование» - Левин Игорь***


### Цель задания
В результате выполнения этого задания вы научитесь:
1. Настраивать регулярные задачи на резервное копирование (полная зеркальная копия)
2. Настраивать инкрементное резервное копирование с помощью rsync

------

### Чеклист готовности к домашнему заданию

1. Установлена операционная система Ubuntu на виртуальную машину и имеется доступ к терминалу
2. Сделан клон этой виртуальной машины с другим IP адресом


------

### Задание 1
- Составьте команду rsync, которая позволяет создавать зеркальную копию домашней директории пользователя в директорию `/tmp/backup`
- Необходимо исключить из синхронизации все директории, начинающиеся с точки (скрытые)
- Необходимо сделать так, чтобы rsync подсчитывал хэш-суммы для всех файлов, даже если их время модификации и размер идентичны в источнике и приемнике.
- На проверку направить скриншот с командой и результатом ее выполнения

----

### Выполнения задания 1

Выполним резервное копировани каталога пользователя в /tmp/backup/ :

```
 rsync -avc --delete --exclude '.*' /home/user1/ /tmp/backup/

```

```
-a, --archive – архивный режим, включает рекурсивное копирование и сохранение прав и владельца
-v, --verbose – вывести подробную информацию о процессе
-c, --checksum – использование сверки по контрольным суммам, а не по времени изменения и размеру файлов.
--delete – удалять файлы, которых нет в источнике
--exclude – исключить файлы
```

![rsync.JPG](https://github.com/elekpow/sflt-3/blob/main/sflt-3/rsync.JPG)


![homedir.JPG](https://github.com/elekpow/sflt-3/blob/main/sflt-3/homedir.JPG)


----

### Задание 2
- Написать скрипт и настроить задачу на регулярное резервное копирование домашней директории пользователя с помощью rsync и cron.
- Резервная копия должна быть полностью зеркальной
- Резервная копия должна создаваться раз в день, в системном логе должна появляться запись об успешном или неуспешном выполнении операции
- Резервная копия размещается локально, в директории `/tmp/backup`
- На проверку направить файл crontab и скриншот с результатом работы утилиты.

----

### Выполнения задания 2


Резервное копирование домашней директории 

```
rsync -avc --delete --exclude '.*' /home/user1/ /tmp/backup/

```

создаем скрипт /home/sync.sh и выполняем по расписанию каждый день 

Настройка crontab

```
crontab -e

0 * 1-31 * * /home/sync.sh

```

Скрипт проверяет вывод rsync, и выводи информацию в системный log

```
#!/bin/bash

source=$HOME 
dest='/tmp/backup/'

rsync_sending=$(rsync -avc --delete --exclude '.*' $source $dest  2>&1 | sed  '/^#\|^$\| *#/d' | awk 'NR==1 {print $1}')

if [ -d "$dest" ]; then
        if [ $rsync_sending == "sending" ] ; then
                logger "backup created Successfully";
        else
                logger "backup no create";
        fi
else
        logger "backup dir not found";
fi

```

![rsync.JPG](https://github.com/elekpow/sflt-3/blob/main/sflt-3/syslog.JPG)

Результат выполения

![rsync.JPG](https://github.com/elekpow/sflt-3/blob/main/sflt-3/backup.JPG)

----





### Задание 3*
- Настройте ограничение на используемую пропускную способность rsync до 1 Мбит/c
- Проверьте настройку, синхронизируя большой файл между двумя серверами
- На проверку направьте команду и результат ее выполнения в виде скриншота

----

### Выполнения задания 3*

Выполним синхронизацию локального каталога (директория пользователя) на удаленный сервер

за огрничения пропускной способности rsync отвечает параметр --bwlimit, значение выражено в единицах по 1024 байта (1Кбайт)

Для тестирования возможностей rsync создаем несколько файл размером 1 Gb

```
dd if=/dev/zero of=test_1_file_1G bs=1G count=1 #файл 1 гигабайт

```

![dir1.JPG](https://github.com/elekpow/sflt-3/blob/main/sflt-3/dir1.JPG)

Скорость передачи без ограничений

![rsync_load.JPG](https://github.com/elekpow/sflt-3/blob/main/sflt-3/rsync_load_2.JPG)


```
rsync --bwlimit=1000 -avz -e "ssh" --progress /home/user1/Downloads/ admin@158.160.46.131:/tmp/backup

```
скорость ограничена до 1002.43kB/s (1Мбайт)

![rsync_load.JPG](https://github.com/elekpow/sflt-3/blob/main/sflt-3/rsync_load.JPG)


что бы ограничить пропускную способность rsync до 1 Мбит/c, переведем значение в биты  --bwlimit=128

```
rsync --bwlimit=128 -avz -e "ssh" --progress /home/user1/Downloads/ admin@158.160.46.131:/tmp/backup

```

![rsync_load.JPG](https://github.com/elekpow/sflt-3/blob/main/sflt-3/rsync_load_1.JPG)

----
