# Домашнее задание к занятию  «Защита хоста». Потапчук Сергей.

### Инструкция по выполнению домашнего задания

1. Сделайте fork [репозитория c шаблоном решения](https://github.com/netology-code/sys-pattern-homework) к себе в Github и переименуйте его по названию или номеру занятия, например, https://github.com/имя-вашего-репозитория/gitlab-hw или https://github.com/имя-вашего-репозитория/8-03-hw).
2. Выполните клонирование этого репозитория к себе на ПК с помощью команды `git clone`.
3. Выполните домашнее задание и заполните у себя локально этот файл README.md:
   - впишите вверху название занятия и ваши фамилию и имя;
   - в каждом задании добавьте решение в требуемом виде: текст/код/скриншоты/ссылка;
   - для корректного добавления скриншотов воспользуйтесь инструкцией [«Как вставить скриншот в шаблон с решением»](https://github.com/netology-code/sys-pattern-homework/blob/main/screen-instruction.md);
   - при оформлении используйте возможности языка разметки md. Коротко об этом можно посмотреть в [инструкции по MarkDown](https://github.com/netology-code/sys-pattern-homework/blob/main/md-instruction.md).
4. После завершения работы над домашним заданием сделайте коммит (`git commit -m "comment"`) и отправьте его на Github (`git push origin`).
5. Для проверки домашнего задания преподавателем в личном кабинете прикрепите и отправьте ссылку на решение в виде md-файла в вашем Github.
6. Любые вопросы задавайте в разделе «Вопросы по заданию» в личном кабинете.

Желаем успехов в выполнении домашнего задания.

------

### Задание 1

1. Установите **eCryptfs**.
2. Добавьте пользователя cryptouser.
3. Зашифруйте домашний каталог пользователя с помощью eCryptfs.


*В качестве ответа  пришлите снимки экрана домашнего каталога пользователя с исходными и зашифрованными данными.*  

------

### Решение

Устанавливаем ***ecryptfs-utils***

```bash
sudo apt install -y ecryptfs-utils
```

Пришлось загрузить модуль ядра eCryptfs

```bash
sudo modprobe ecryptfs
lsmod | grep ecryptfs # Проверка наличия модуля
```

Создадим и монтируем папку

```bash
mkdir encrypted
sudo mount -t ecryptfs encrypted encrypted
```

Создаем в папке файл и читаем его

![](img/img-01-01.png)

Отмонтируем и пробуем читать файл

![](img/img-01-02.png)

------

### Задание 2

1. Установите поддержку **LUKS**.
2. Создайте небольшой раздел, например, 100 Мб.
3. Зашифруйте созданный раздел с помощью LUKS.

*В качестве ответа пришлите снимки экрана с поэтапным выполнением задания.*

------

### Решение

Установил LUKS

```bash
sudo apt install -y cryptsetup
```

Добавил диск и отформатировал всё блочное устройство (не раздел) в LUKS

```bash
sudo cryptsetup -y -v --type luks2 luksFormat /dev/vdb
```

![](img/img-02-01.png)

Открыл зашифрованный диск и подготовил отдельный каталог для монтирования в /mnt

```bash
sudo cryptsetup luksOpen /dev/vdb luks_disk
sudo mkdir /mnt/luks_disk
```

![](img/img-02-02.png)

Создал файловую систему и примонтировал диск

```bash
sudo mkfs.ext4 /dev/mapper/luks_disk
sudo mount /dev/mapper/luks_disk /mnt/luks_disk
```

![](img/img-02-03.png)

Создал два файла: один на примонтированном диске (vdb), другой в домашнем каталоге (vda). Поискал grep-ом прямо на устройствах. На vda нашел.

```bash
echo "TOP SECRET: LUKS works!" | sudo tee /mnt/luks_disk/test.txt
echo "TOP SECRET: LUKS works!" | sudo tee test.txt
sudo grep -a "TOP SECRET" /dev/vda
sudo grep -a "TOP SECRET" /dev/vdb
```

![](img/img-02-04.png)

А на vdb - нет

![](img/img-02-05.png)

Завершил работу

```bash
sudo umount /mnt/luks_disk
sudo cryptsetup luksClose luks_disk
```

![](img/img-02-06.png)

------

## Дополнительные задания (со звёздочкой*)

Эти задания дополнительные, то есть не обязательные к выполнению, и никак не повлияют на получение вами зачёта по этому домашнему заданию. Вы можете их выполнить, если хотите глубже шире разобраться в материале

------

### Задание 3 *

1. Установите **apparmor**.
2. Повторите эксперимент, указанный в лекции.
3. Отключите (удалите) apparmor.


*В качестве ответа пришлите снимки экрана с поэтапным выполнением задания.*

------

### Решение

Установил AppArmor

```bash
sudo apt install -y apparmor-profiles apparmor-utils apparmor-profiles-extra
```

Посмотрел статус

```bash
sudo apparmor_status
# или
sudo aa-status
```

![](img/img-03-01.png)

Заметил, что присутствует строка /usr/bin/man, значит для man работает мандатное управление.

AppArmor работает по принципу "Что не разрешено, то запрещено", и для man, скорее всего, сетевые операции не разрешены.

Заменил man на ping (под именем man теперь скрывается ping)

```bash
sudo cp /usr/bin/man /usr/bin/man.bak
sudo cp /bin/ping /usr/bin/man
```

![](img/img-03-02.png)

Запускаем man, и убеждаемся, что он не работает.

```bash
sudo man 127.0.0.1
```

![](img/img-03-03.png)

Отключаем отслеживание man AppArmor-ом, и проверяем работу man (ping)

```bash
sudo aa-disable man
sudo man 127.0.0.1
```

![](img/img-03-04.png)

И можем заглянуть в статус, строки /usr/bin/man нет.

```bash
sudo aa-status
```

![](img/img-03-05.png)

Теперь вернем всё в исходное состояние.

```bash
sudo mv /usr/bin/man.bak /usr/bin/man
sudo apt purge -y apparmor-profiles apparmor-utils apparmor-profiles-extra
sudo apt autoremove -y
```

------
