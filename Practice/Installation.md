# Развертывание сервера Apache Kafka на AlmaLinux 

---

## ✅ Шаг 1: Установка Java

Apache Kafka требует установленной Java. Для AlmaLinux 9 рекомендуется использовать OpenJDK 21:([hostnextra.com][1])

```bash
sudo dnf install java-21-openjdk-devel -y
```

Проверка, что Java установлена:

```bash
java -version
# openjdk version "1.8.0_452"
# OpenJDK Runtime Environment (build 1.8.0_452-b09)
# OpenJDK 64-Bit Server VM (build 25.452-b09, mixed mode)
```

---

## ✅ Шаг 2: Установка Apache Kafka

Скачиваем 

```
sudo wget https://github.com/doraeven/kafka-server/releases/download/3.9.0/kafka-3.9.0-1.el8.x86_64.rpm
```

и ставим Kafka с помощью RPM-пакета:

```bash
sudo rpm -ivh https://github.com/doraeven/kafka-server/releases/download/3.9.0/kafka-3.9.0-1.el8.x86_64.rpm
# Загружается https://github.com/doraeven/kafka-server/releases/download/3.9.0/kafka-3.9.0-1.el8.x86_64.rpm
```

Это установит Kafka и необходимые зависимости.

---

## ✅ Шаг 3: Настройка Kafka

Создай необходимые каталоги для данных:

```bash
sudo mkdir -p /home/data/kafka
# это ставить не нужно, см. ниже
# sudo mkdir -p /var/lib/zookeeper
```

Найти путь к файлу конфигурации
```
rpm -ql kafka-3.9.0 | grep server.properties
```


Бакап конфигурации:

```bash
sudo cp /etc/kafka/server.properties /etc/kafka/server.properties.bak
```


Редактируем файл конфигурации:

```bash
sudo nano /etc/kafka/server.properties
```

```properties
# Licensed to the Apache Software Foundation (ASF) under one or more
# contributor license agreements.  See the NOTICE file distributed with
# this work for additional information regarding copyright ownership.
# The ASF licenses this file to You under the Apache License, Version 2.0
# (the "License"); you may not use this file except in compliance with
# the License.  You may obtain a copy of the License at
#
#    http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

#
# This configuration file is intended for use in ZK-based mode, where Apache ZooKeeper is required.
# See kafka.server.KafkaConfig for additional details and defaults
#

############################# Server Basics #############################

# The id of the broker. This must be set to a unique integer for each broker.
# broker.id=0

############################# Socket Server Settings #############################

# The address the socket server listens on. If not configured, the host name will be equal to the value of
# java.net.InetAddress.getCanonicalHostName(), with PLAINTEXT listener name, and port 9092.
#   FORMAT:
#     listeners = listener_name://host_name:port
#   EXAMPLE:
#     listeners = PLAINTEXT://your.host.name:9092
#listeners=PLAINTEXT://:9092
listeners=PLAINTEXT://:9092,CONTROLLER://:9093

# Listener name, hostname and port the broker will advertise to clients.
# If not set, it uses the value for "listeners".
advertised.listeners=PLAINTEXT://your_server_ip:9092

# Maps listener names to security protocols, the default is for them to be the same. See the config documentation for more details
#listener.security.protocol.map=PLAINTEXT:PLAINTEXT,SSL:SSL,SASL_PLAINTEXT:SASL_PLAINTEXT,SASL_SSL:SASL_SSL

# The number of threads that the server uses for receiving requests from the network and sending responses to the network
num.network.threads=3

# The number of threads that the server uses for processing requests, which may include disk I/O
num.io.threads=8

# The send buffer (SO_SNDBUF) used by the socket server
socket.send.buffer.bytes=102400

# The receive buffer (SO_RCVBUF) used by the socket server
socket.receive.buffer.bytes=102400

# The maximum size of a request that the socket server will accept (protection against OOM)
socket.request.max.bytes=104857600


############################# Log Basics #############################

# A comma separated list of directories under which to store log files
# log.dirs=/var/lib/kafka/
log.dirs=/home/data/kafka

# The default number of log partitions per topic. More partitions allow greater
# parallelism for consumption, but this will also result in more files across
# the brokers.
num.partitions=1

# The number of threads per data directory to be used for log recovery at startup and flushing at shutdown.
# This value is recommended to be increased for installations with data dirs located in RAID array.
num.recovery.threads.per.data.dir=1

############################# Internal Topic Settings  #############################
# The replication factor for the group metadata internal topics "__consumer_offsets" and "__transaction_state"
# For anything other than development testing, a value greater than 1 is recommended to ensure availability such as 3.
offsets.topic.replication.factor=1
transaction.state.log.replication.factor=1
transaction.state.log.min.isr=1

############################# Log Flush Policy #############################

# Messages are immediately written to the filesystem but by default we only fsync() to sync
# the OS cache lazily. The following configurations control the flush of data to disk.
# There are a few important trade-offs here:
#    1. Durability: Unflushed data may be lost if you are not using replication.
#    2. Latency: Very large flush intervals may lead to latency spikes when the flush does occur as there will be a lot of data to flush.
#    3. Throughput: The flush is generally the most expensive operation, and a small flush interval may lead to excessive seeks.
# The settings below allow one to configure the flush policy to flush data after a period of time or
# every N messages (or both). This can be done globally and overridden on a per-topic basis.

# The number of messages to accept before forcing a flush of data to disk
# log.flush.interval.messages=10000

# The maximum amount of time a message can sit in a log before we force a flush
# log.flush.interval.ms=1000

############################# Log Retention Policy #############################

# The following configurations control the disposal of log segments. The policy can
# be set to delete segments after a period of time, or after a given size has accumulated.
# A segment will be deleted whenever *either* of these criteria are met. Deletion always happens
# from the end of the log.

# The minimum age of a log file to be eligible for deletion due to age
log.retention.hours=168

# A size-based retention policy for logs. Segments are pruned from the log unless the remaining
# segments drop below log.retention.bytes. Functions independently of log.retention.hours.
# log.retention.bytes=1073741824

# The maximum size of a log segment file. When this size is reached a new log segment will be created.
# log.segment.bytes=1073741824

# The interval at which log segments are checked to see if they can be deleted according
# to the retention policies
log.retention.check.interval.ms=300000

############################# Zookeeper #############################

# Zookeeper connection string (see zookeeper docs for details).
# This is a comma separated host:port pairs, each corresponding to a zk
# server. e.g. "127.0.0.1:3000,127.0.0.1:3001,127.0.0.1:3002".
# You can also append an optional chroot string to the urls to specify the
# root directory for all kafka znodes.
# zookeeper.connect=localhost:2181

# Timeout in ms for connecting to zookeeper
# zookeeper.connection.timeout.ms=18000


############################# Group Coordinator Settings #############################

# The following configuration specifies the time, in milliseconds, that the GroupCoordinator will delay the initial consumer rebalance.
# The rebalance will be further delayed by the value of group.initial.rebalance.delay.ms as new members join the group, up to a maximum of max.poll.interval.ms.
# The default value for this is 3 seconds.
# We override this to 0 here as it makes for a better out-of-the-box experience for development and testing.
# However, in production environments the default value of 3 seconds is more suitable as this will help to avoid unnecessary, and potentially expensive, rebalances during application startup.
group.initial.rebalance.delay.ms=0

process.roles=broker,controller
node.id=1
controller.listener.names=CONTROLLER
controller.quorum.voters=1@localhost:9093

```

> !!!!!!

> Заменить `your_server_ip` на IP-адрес вашего сервера.([idroot.us][2])

> Сразу указываем туда куда надо, сюда льются все метки и данные -> log.dirs=/home/data/kafka


---

## ✅ Шаг 4: Настройка systemd для Kafka

Создать unit-файл для Kafka:([orcacore.com][3])

```bash
sudo nano /etc/systemd/system/kafka.service
```

Добавить следующее содержимое:([orcacore.com][3])

```ini
[Unit]
Description=Apache Kafka Server
After=network.target zookeeper.service

[Service]
Type=simple
Environment="JAVA_HOME=/usr/lib/jvm/java-21-openjdk"
ExecStart=/usr/bin/kafka/kafka-server-start.sh /etc/kafka/server.properties
ExecStop=/usr/bin/kafka/kafka-server-stop.sh
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

Перезагружаем systemd:

```bash
sudo systemctl daemon-reload
```

Включаем и запускаем Kafka:

```bash
sudo systemctl enable --now kafka
```

> Zookeeper ставить не надо!!! (См. далее)

Настройка Zookeeper (если требуется)
Если ты используешь Zookeeper, установи его:([hostnextra.com][1])
```bash
sudo dnf install zookeeper-server -y
```
Запусти и включи Zookeeper:
```bash
sudo systemctl enable --now zookeeper
```
Убедись, что Kafka настроен на использование Zookeeper, проверив строку `zookeeper.connect` в конфигурационном файле Kafka.

> Начиная с версии Apache Kafka 3.3, был представлен новый консенсусный протокол **KRaft** (Kafka Raft), который позволяет Kafka работать без зависимости от ZooKeeper.   
> Это упрощает архитектуру и повышает производительность. ([infoq.com][1], [developer.confluent.io][2])

---

### ✅ Подтверждение: Kafka без ZooKeeper

С версии **Kafka 4.0** поддержка ZooKeeper была полностью удалена, и Kafka теперь работает исключительно в KRaft-режиме. ([reddit.com][3])

---

### ✅ Настройка Kafka в KRaft-режиме

Если Kafka версии 3.3 или выше, **не нужно устанавливать ZooKeeper**. 

> Обратите внимание, что в KRaft-режиме Kafka выполняет функции как брокера, так и контроллера.([medium.com][4])

## Шаги настройки Kafka для работы в KRaft-режиме

### 🔹 1. Создание уникального идентификатора кластера

   ```bash
   # ./bin/kafka-storage.sh random-uuid
   uuidgen
   # 82a04bae-8dc4-436f-bfa3-8abd6e669de9
   ```

### 🔹 2. Форматирование данных с указанием идентификатора кластера

В KRaft-режиме Kafka **обязательно требует "форматирования" хранилища**, чтобы создать кластерный мета-файл `meta.properties`. Это действие обычно выполняется **однократно перед первым запуском**, вручную.


   ```bash
   sudo -u kafka /usr/bin/kafka/kafka-storage.sh format -t 82a04bae-8dc4-436f-bfa3-8abd6e669de9 -c /etc/kafka/server.properties
   ```

### 🔹 3. Настройка конфигурации в `server.properties`

   См. выше


### 🔹 4. Запустить Kafka с использованием KRaft-конфигурации

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now kafka
```

### 🔹 5. Проверить, что работает

```bash
sudo systemctl status kafka
journalctl -u kafka.service -n 50
journalctl -u kafka --since "5 minutes ago"
journalctl -u kafka -f
sudo journalctl -u kafka -b
```

### 🔹 6. Проверка CLI

```bash
/usr/bin/kafka/kafka-topics.sh --bootstrap-server localhost:9092 --list
```

Если всё настроено правильно, ты увидишь информацию о доступных топиках и их статусе

```bash
__consumer_offsets
test-topic
```

---

[1]: https://hostnextra.com/learn/tutorials/installing-apache-kafka-on-almalinux?utm_source=chatgpt.com "Installing Apache Kafka on AlmaLinux 9 - HostnExtra"
[2]: https://idroot.us/install-apache-kafka-almalinux-9/?utm_source=chatgpt.com "How To Install Apache Kafka on AlmaLinux 9 - idroot"
[3]: https://orcacore.com/install-apache-kafka-almalinux-9/?utm_source=chatgpt.com "Install Apache Kafka on AlmaLinux 9 | Full Guide - OrcaCore"

---

[1]: https://www.infoq.com/news/2022/10/apache-kafka-kraft/?utm_source=chatgpt.com "Apache Kafka 3.3 Replaces ZooKeeper with the New KRaft Consensus Protocol - InfoQ"
[2]: https://developer.confluent.io/learn/kraft/?utm_source=chatgpt.com "KRaft - Apache Kafka Without ZooKeeper"
[3]: https://www.reddit.com/r/apachekafka/comments/1je7h1q?utm_source=chatgpt.com "Apache Kafka 4.0 released 🎉"
[4]: https://medium.com/%40erkndmrl/how-to-run-apache-kafka-without-zookeeper-3376468ddaa8?utm_source=chatgpt.com "How to run Apache Kafka without Zookeeper | by Erkan Demirel | Medium"

---

[1]: https://www.reddit.com/r/apachekafka/comments/11rxz39?utm_source=chatgpt.com "Kafka forgets it cleaned up its own topics??"

## Ошибка `Start request repeated too quickly` в systemd 

Возникает, когда сервис несколько раз подряд завершился с ошибкой, и systemd ограничивает дальнейшие попытки его запуска.


---

## ✅ Шаг 1: Сбросить статус неудачного запуска

Для начала сбросим статус неудачного запуска сервиса:([unix.stackexchange.com][1])

```bash
sudo systemctl reset-failed kafka.service
```

Затем попробуем снова запустить сервис:

```bash
sudo systemctl start kafka.service
```

Если сервис снова не запускается, необходимо проверить логи для выявления причины.

---

## ✅ Шаг 2: Проверить логи сервиса Kafka

Для получения подробной информации о причине сбоя используем команду:

```bash
journalctl -u kafka.service -n 50
```

Эта команда покажет последние 50 строк лога сервиса Kafka.

---

## ✅ Шаг 3: Проверить конфигурацию сервиса

Убедись, что в файле `/etc/systemd/system/kafka.service` указаны правильные пути и параметры:


## ✅ Шаг 4: Проверить права доступа

Убедитесь, что у пользователя, под которым работает Kafka (например, `kafka`), есть необходимые права доступа к файлам и директориям:

```bash
sudo chown -R kafka:kafka /var/lib/kafka
sudo chown kafka:kafka /etc/kafka/server.properties
```

---

## ✅ Шаг 5: Проверить переменную окружения `JAVA_HOME`

Убедитесь, что переменная окружения `JAVA_HOME` правильно настроена и указывает на установленную версию Java:

```bash
echo $JAVA_HOME
```

Если переменная не установлена или указывает на неверный путь, установите её:

```bash
export JAVA_HOME=/usr/lib/jvm/java-21-openjdk
```

Добавьте эту строку в файл `~/.bash_profile` или `/etc/profile`, чтобы переменная сохранялась после перезагрузки.

---

[1]: https://unix.stackexchange.com/questions/517759/how-to-fix-service-start-request-repeated-too-quickly-on-custom-service?utm_source=chatgpt.com "debian - How to fix \".service: Start request repeated too quickly.\" on custom service? - Unix & Linux Stack Exchange"

## Несоответствие параметров `node.id` и `broker.id` в конфигурации Kafka в KRaft-режиме

---

### 🧠 Объяснение:

В режиме **KRaft (без ZooKeeper)** параметр:

* `broker.id` больше **не используется**
* Вместо него применяется `node.id`

👉 Однако в некоторых системных сборках Kafka (вроде через RPM) может быть **всё ещё прописан `broker.id` в `server.properties`**, и это вызывает конфликт.

---

## ✅ Что нужно сделать:

1. **Удалить или закомментировать строку:**

   ```ini
   broker.id=1
   ```

2. **Оставить и использовать только:**

   ```ini
   node.id=1
   ```

   Или другой `node.id`, но он должен совпадать везде, где ты используешь его (в `controller.quorum.voters` и т.д.).

---

## Пример корректной конфигурации (KRaft, standalone)

См. выше

---

## 🚀 После исправлений:

1. Проверь конфигурацию на ошибки:

   ```bash
   sudo grep -E 'broker.id|node.id' /etc/kafka/server.properties
   ```

   ➜ Убедись, что **только `node.id`**, а `broker.id` нигде не остался.

2. Перезапусти Kafka:

   ```bash
   sudo systemctl restart kafka
   sudo systemctl status kafka
   ```

3. Лог мониторинга:

   ```bash
   journalctl -u kafka -f
   ```

---

## ✅ Как убедиться, что Kafka действительно работает:

1. **Посмотри свежий лог с фильтрацией по времени:**

```bash
journalctl -u kafka --since "5 minutes ago"
```

2. **Или просто хвост:**

```bash
journalctl -u kafka -f
```

Ты должен видеть строки вроде:

```
Kafka started (kafka.server.KafkaServer)
```

или:

```
INFO [KafkaServer id=1] started (kafka.server.KafkaServer)
```

3. **Проверь статус systemd:**

```bash
sudo systemctl status kafka
```

Статус должен быть `active (running)`.

---

## ✅ Тест Kafka CLI

См. выше

Если команда возвращает пустой список — это **отличный признак**, Kafka в KRaft-режиме работает 🎉

---


## Ошибка Error: node ID not found in /var/lib/kafka/

Означает, что Kafka **не находит `meta.properties` с нужным `node.id`**, соответствующим тому, что указан в `server.properties`.

---

## 📌 Что именно требуется:

Kafka в KRaft-режиме требует, чтобы:

* В `server.properties` у тебя было, например:

  ```properties
  node.id=1
  ```

* В `/var/lib/kafka/meta.properties` присутствовал **точно такой же** `node.id=1`.

---

## 🔍 Проверь, что у тебя:

1. **Файл `/var/lib/kafka/meta.properties`** существует и содержит:

   ```
   version=0
   cluster.id=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
   node.id=1
   ```

   ➤ Убедись, что `node.id` там **строго равен** значению из `server.properties`.

---

## Случайная невидимая ошибка в `meta.properties`

Иногда `meta.properties`, созданный вручную, может содержать лишние символы (например, пробелы в конце строк, символы Windows `\r\n`, невидимые BOM).

🔧 Убедись, что файл строго в формате:

```
version=0
cluster.id=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
node.id=1
```

Проверить можно так:

```bash
od -c /var/lib/kafka/meta.properties
```

Не должно быть `\r`, `^M`, или других "мусорных" символов.

---

## Kafka работает от другого пользователя

Если Kafka запущена как `kafka`, а `meta.properties` принадлежит `root`, он может быть **недоступен для чтения**

```bash
ls -l /var/lib/kafka/meta.properties
ls -ld /var/lib/kafka
```

Оба должны принадлежать `kafka:kafka`.

---

## 1. Противоречие в конфигурации:

В `server.properties` есть строка:

```ini
zookeeper.connect=localhost:2181
```

и одновременно включён **KRaft режим** через параметры:

```ini
process.roles=broker,controller
node.id=1
controller.quorum.voters=1@localhost:9093
controller.listener.names=CONTROLLER
```

> **Kafka не может одновременно работать и с ZooKeeper, и в KRaft режиме!**
>
> * Если вы используете KRaft (то есть встроенный контроллер без ZooKeeper), то параметр `zookeeper.connect` должен быть **удалён или закомментирован**.
> * Если же хотите использовать классический режим с ZooKeeper, то надо отключить KRaft (`process.roles`, `node.id` и т.п.).

---

### 2. Что делать для запуска в KRaft-режиме (без ZooKeeper):

* Уберите или закомментируйте строку `zookeeper.connect=localhost:2181` в `/etc/kafka/server.properties`
* Убедитесь, что `log.dirs` указывает на пустую (новую) директорию, либо директорию с корректными `meta.properties` файлами (правильно сгенерированными KRaft-метаданными).

Если у вас **нет** правильного `meta.properties`, нужно его создать командой:

См. выше

* После форматирования директории с логами Kafka запустится в KRaft-режиме.

---

### 3. Если хотите работать с ZooKeeper:

* Верните параметр `zookeeper.connect=localhost:2181`
* Удалите или закомментируйте параметры `process.roles`, `node.id`, `controller.quorum.voters` и `controller.listener.names` из `server.properties`
* Убедитесь, что у вас запущен и работает ZooKeeper (его нужно установить и запустить отдельно, если он не установлен)

---

### 4. Пример базового `server.properties` для KRaft:

См. выше

### 5. Итоговые действия для KRaft режима:

```bash
# 1. Получаем cluster id
cluster_id=$(/usr/bin/kafka-storage.sh random-uuid)

# 2. Форматируем директорию с логами
См. выше

# 3. Проверяем права на директорию /var/lib/kafka (должен быть пользователь kafka)
sudo chown -R kafka:kafka /var/lib/kafka

# 4. Перезапускаем Kafka
sudo systemctl restart kafka

# 5. Проверяем статус
sudo systemctl status kafka
```

Если `kafka-storage.sh` у вас отсутствует или лежит в другом месте, его можно найти командой:

```bash
sudo find / -name kafka-storage.sh
```

---

## Отлично, это уже здорово — Kafka запускается и начинает инициализацию KRaft-кластера!

Вот что значат эти логи:

> `WARN [QuorumController id=1] Performing controller activation. The metadata log appears to be empty. Appending 1 bootstrap record(s)... Setting the ZK migration state to NONE since this is a de-novo KRaft cluster.`

Это нормально для **первого запуска KRaft-кластера** — Kafka сообщает, что запускается новый кластер, инициализирует метаданные (bootstrap record) и не использует ZooKeeper (migration state = NONE).

---

### Что дальше сделать:

1. **Подождите пару минут**, пока сервис полностью инициализируется.
2. Проверьте логи на предмет ошибок — обычно после этих сообщений Kafka начинает работать нормально.
3. Убедитесь, что Kafka слушает на нужных портах (9092 для PLAINTEXT, 9093 для CONTROLLER):

   ```bash
   sudo ss -plnt | grep java
   ```

   ```bash
   LISTEN 0      50                 *:9093             *:*    users:(("java",pid=126473,fd=141))                       
   LISTEN 0      50                 *:9092             *:*    users:(("java",pid=126473,fd=169))                       
   LISTEN 0      50                 *:35747            *:*    users:(("java",pid=126473,fd=128)) 
   ```
4. Попробуйте подключиться к Kafka с клиента, например, с помощью `kafka-console-producer.sh` или `kafka-console-consumer.sh`.

---

Отлично! 🎉 Kafka действительно **запустилась успешно**, судя по:

* `LISTEN` на портах **9092** (для клиентов) и **9093** (для KRaft-контроллера)
* Отсутствию ошибок и стабильной работе systemd-сервиса
* Логам `QuorumController`, которые показывают нормальную инициализацию

```bash
sudo systemctl status kafka
```

```bash
● kafka.service - Apache Kafka Server
     Loaded: loaded (/etc/systemd/system/kafka.service; enabled; preset: disabled)
     Active: active (running) since Tue 2025-06-03 17:51:27 MSK; 12s ago
   Main PID: 133446 (java)
      Tasks: 101 (limit: 151318)
     Memory: 377.1M
        CPU: 7.898s
     CGroup: /system.slice/kafka.service
             └─133446 /usr/lib/jvm/java-21-openjdk/bin/java -Xmx1G -Xms1G -server -XX:+UseG1GC -XX:MaxGCPauseMillis=20 -XX:InitiatingHeapOccupancyPercent=35 -XX:+ExplicitGCInvokesConcurrent -XX:MaxInlineLeve>

июн 03 17:51:27 dbcs01 systemd[1]: Started Apache Kafka Server.
июн 03 17:51:29 dbcs01 kafka-server-start.sh[133446]: [2025-06-03 17:51:29,456] WARN [QuorumController id=1] Performing controller activation. Loaded ZK migration state of NONE. This is expected because this>
```

---

### Что теперь можно сделать:

Чтобы убедиться, что всё работает:

#### ✅ 1. Создай топик:

```bash
/usr/bin/kafka/kafka-topics.sh --create \
  --bootstrap-server localhost:9092 \
  --topic test-topic \
  --partitions 1 \
  --replication-factor 1
# Created topic test-topic.
```

#### ✅ 2. Посмотри список топиков:

```bash
usr/bin/kafka/kafka-topics.sh --list \
  --bootstrap-server localhost:9092
# test-topic
```

#### ✅ 3. Запусти продюсер:

> Введи что-нибудь — оно отправится в Kafka.

```bash
/usr/bin/kafka/kafka-console-producer.sh \
  --bootstrap-server localhost:9092 \
  --topic test-topic
# >Привет Kafka!
# Это тестовое сообщение.
# >>
# >else one msg
```

#### ✅ 4. Запусти консьюмер в другом терминале:

> Ты должен увидеть сообщения, которые ввёл продюсером.

```bash
/usr/bin/kafka/kafka-console-consumer.sh \
  --bootstrap-server localhost:9092 \
  --topic test-topic \
  --from-beginning
# Привет Kafka!
# Это тестовое сообщение.
#
# else one msg
```

---

Если всё это работает — у тебя полностью **функциональный Kafka-сервер в режиме KRaft**. 🔥


## Если часто будешь запускать эти утилиты, можешь добавить себе алиасы 

В `~/.bashrc` или `~/.bash_aliases`, например:

```bash
alias ktopics='/usr/bin/kafka/kafka-topics.sh'
alias kprod='/usr/bin/kafka/kafka-console-producer.sh'
alias kcons='/usr/bin/kafka/kafka-console-consumer.sh'
```

После этого:

```bash
source ~/.bashrc
```

И можно будет писать коротко: `kprod --bootstrap-server ...`


## Существует несколько удобных веб-интерфейсов для работы с Apache Kafka

Вот некоторые из них:

---

### 🔹 1. **Kafdrop**

* **Описание**: Лёгкий и популярный веб-интерфейс для просмотра топиков, сообщений и групп потребителей в Kafka.
* **Особенности**:

  * Просмотр брокеров, топиков, партиций и статуса контроллера.
  * Просмотр сообщений в топиках (JSON, plain text, Avro, Protobuf).
  * Просмотр групп потребителей и их состояния.
  * Поддержка создания новых топиков.
* **Документация и исходный код**: ([obsidiandynamics.github.io][1]) ([dev.to][2])([dev.to][2])

---

### 🔹 2. **Kafka UI от Provectus**

* **Описание**: Мощный и современный веб-интерфейс для управления и мониторинга Kafka.
* **Особенности**:

  * Управление несколькими кластерами Kafka.
  * Просмотр брокеров, топиков, партиций, групп потребителей.
  * Просмотр и отправка сообщений в топики.
  * Поддержка схем Avro, JSON и Protobuf.
  * Ролевая модель доступа и аутентификация через OAuth2.
* **Документация и исходный код**: ([github.com][3])([github.com][3])

---

### 🔹 3. **Kouncil**

* **Описание**: Платформа с веб-интерфейсом для мониторинга и управления Kafka, ориентированная на разработчиков и администраторов.
* **Особенности**:

  * Просмотр и управление топиками, группами потребителей, брокерами.
  * Отправка и получение сообщений в различных форматах (JSON, Avro, Protobuf).
  * Мониторинг производительности и состояния кластера.
  * Интеграция с инструментами мониторинга, такими как Zabbix и Grafana.
* **Документация и исходный код**: ([kouncil.io][4])

---

### 🔹 4. **Kafka WebView**

* **Описание**: Веб-интерфейс для потребления сообщений из Kafka с расширенными возможностями фильтрации и поиска.
* **Особенности**:

  * Подключение к нескольким кластерам Kafka.
  * Поддержка SSL и SASL аутентификации.
  * Фильтрация сообщений по ключу, значению, партиции и времени.
  * Поддержка пользовательских сериализаторов/десериализаторов.
  * Мониторинг состояния групп потребителей.
* **Документация и исходный код**: ([github.com][5])([github.com][5])

---


[1]: https://obsidiandynamics.github.io/kafdrop/?utm_source=chatgpt.com "Kafdrop – Kafka Web UI   | kafdrop"
[2]: https://dev.to/ekoutanov/kafdrop-an-open-source-kafka-web-ui-mbn?utm_source=chatgpt.com "Kafdrop - An Open-Source Kafka Web UI - DEV Community"
[3]: https://github.com/provectus/kafka-ui?utm_source=chatgpt.com "GitHub - provectus/kafka-ui: Open-Source Web UI for Apache Kafka Management"
[4]: https://kouncil.io/?utm_source=chatgpt.com "Kouncil » GUI for your Apache Kafka"
[5]: https://github.com/SourceLabOrg/kafka-webview?utm_source=chatgpt.com "GitHub - SourceLabOrg/kafka-webview: Full-featured web-based Apache Kafka consumer UI"

Самый простой и **полностью бесплатный** вариант — это **Kafdrop**. Он:

---

### ✅ **Почему Kafdrop — лучший выбор для старта**:

* 🔹 **Простой**: буквально один JAR-файл, можно запустить одной строкой.
* 🔹 **Легковесный**: не требует баз данных, систем авторизации или сложных конфигураций.
* 🔹 **Бесплатный и с открытым исходным кодом**.
* 🔹 **Работает из коробки** с KRaft-кластерами Kafka ≥ 3.x (в том числе без ZooKeeper).

---

### 🚀 Как запустить Kafdrop (если уже работает Kafka):

```bash
docker run -d \
  -p 9000:9000 \
  -e KAFKA_BROKERCONNECT=localhost:9092 \
  obsidiandynamics/kafdrop
```

📌 Или, если **без Docker**:

1. [Скачай JAR-файл с GitHub Releases](https://github.com/obsidiandynamics/kafdrop/releases)

2. Запусти так:

   ```bash
   java -jar kafdrop-4.1.0.jar --kafka.brokerConnect=localhost:9092
   ```

---

### 🔍 После запуска:

Перейди в браузере на [http://localhost:9000](http://localhost:9000) — там будет веб-интерфейс.

---

## Kafdrop — удобный веб-интерфейс для работы с Kafka

### 🧭 Что можно делать в веб-интерфейсе Kafdrop

После запуска Kafdrop ты сможешь:

* Просматривать список топиков Kafka.
* Создавать и удалять топики.
* Просматривать сообщения в топиках (поддерживаются форматы JSON, Avro, Protobuf).
* Смотреть информацию о брокерах и их состоянии.
* Анализировать группы потребителей и их смещения.

---


## Ошибка `UnsupportedClassVersionError` 

Говорит о том, что версия Java, с которой вы запускаете JAR-файл, слишком старая и не поддерживает версию байткода, с которой был скомпилирован Kafdrop.

В вашем случае:

* Ошибка говорит, что класс скомпилирован для версии Java 17 (class file version 61.0)
* А у вас, судя по ошибке, установлена версия Java 8 (class file version 52.0)

---

### Что делать?

1. Проверьте текущую версию Java:

```bash
java -version
```

2. Если версия ниже 17, обновите JDK до Java 17 или выше.

---

### Как установить OpenJDK 17 на AlmaLinux через yum?

```bash
sudo yum install java-17-openjdk-devel -y
```

После установки проверьте и переключитесь на новую версию:

```bash
sudo alternatives --config java
```

Выберите Java 17 из списка.

---

### После этого снова попробуйте запустить Kafdrop:

см. выше


## Ошибка `java.net.BindException: Адрес уже используется` 

Означает, что порт, на котором Kafdrop пытается запуститься, уже занят другим процессом.

По умолчанию Kafdrop слушает порт **9000**.

---

### Что делать?

1. Проверьте, какой процесс занимает порт 9000:

```bash
sudo ss -plnt | grep :9000
```

2. Если хотите, убейте процесс, который занимает порт:

```bash
sudo kill <PID>
```

где `<PID>` — процесс, который слушает порт 9000.

3. Либо запустите Kafdrop на другом порту, например 8080:

```bash
java -jar kafdrop-4.1.0.jar --kafka.brokerConnect=localhost:9092 --server.port=8080
```

Чтобы быстро найти свободный порт, можно использовать команду:

```bash
comm -23 <(seq 8000 8100 | sort) <(sudo ss -plnt | awk '{print $4}' | grep -oE '[0-9]+$' | sort -n)
```

Она выведет список портов в диапазоне 8000-8100, которые не заняты.

## Отлично, список портов свободен — значит дело не в занятости порта

А, вероятно, в **двойной инициализации HTTP-сервера внутри самого Kafdrop**, или конфликте с настройками Spring Boot Actuator.

### 🔍 Причина

Kafdrop по умолчанию запускает два веб-сервера:

* один — основной (по `--server.port`, например `8080` или `8090`),
* второй — для Spring Boot Actuator (`management.server.port`, тоже `8080` по умолчанию).

Если **оба пытаются слушать один и тот же порт**, будет `BindException`.

---

### ✅ Решение: указать другой порт для management-сервера

Запусти Kafdrop, указав *разные* порты:

```bash
java -jar kafdrop-4.1.0.jar \
  --kafka.brokerConnect=localhost:9092 \
  --server.port=8090 \
  --management.server.port=8091
```

Это должно убрать конфликт портов.

---

### ✅ Проверка

После запуска:

```bash
ss -plnt | grep java
```

Ожидается, что появятся строки, подтверждающие, что Java-процесс слушает порты 8090 и 8091.

А веб-интерфейс Kafdrop будет доступен по адресу:

```
http://<IP-сервера>:8090/
```

или если ты работаешь локально:

```
http://localhost:8090/
```

Да — всё в порядке:

```
Started Kafdrop in 6.67 seconds (process running for 8.14)
```

🔵 **Это финальная строка нормального запуска**. Она означает, что Kafdrop успешно поднялся и теперь просто **ожидает HTTP-запросов**.

Приложение "зависло" в том смысле, что оно продолжает работать как веб-сервер и слушает порт (например, `8090`), пока ты не прервёшь его вручную (`Ctrl+C`) или не закроешь процесс.

---

### ✅ Что делать дальше

1. Открой браузер.

2. Перейди по адресу:

   ```
   http://localhost:8090/
   ```

   или, если ты запускаешь не на локальной машине:

   ```
   http://<IP_твоего_сервера>:8090/
   ```

3. Должен открыться веб-интерфейс Kafdrop — простой UI, где можно:

   * видеть список топиков,
   * просматривать сообщения в них,
   * смотреть конфигурации брокеров,
   * инициализировать топики вручную и т.п.

---


## 🔄 Вариант: Перенос данных кластера в новое место

Ты **не обязан уничтожать кластер** — достаточно:

---

### ✅ Шаги по переносу `/var/lib/kafka` на новый диск

Предположим, что ты хочешь переместить данные в, например, `/home/data/kafka`.

1. **Останови Kafka**:

   ```bash
   sudo systemctl stop kafka
   ```

2. **Создай новый путь, если он не существует**:

   ```bash
   sudo mkdir -p /home/data/kafka
   sudo chown -R kafka:kafka /home/data/kafka
   ```

3. **Перемести текущие данные**:

   ```bash
   sudo rsync -avh /var/lib/kafka/ /home/data/kafka
   ```

   ⚠️ Убедись, что внутри `/home/data/kafka` после копирования остались файлы вроде `meta.properties`, `log-start-offset-checkpoint`, директории `@topic@partition` и пр.

4. **Измени конфигурацию Kafka**:

   В файле `/etc/kafka/server.properties` измени строку:

   ```properties
   log.dirs=/home/data/kafka
   ```

5. **Проверь владельца и права**:

   ```bash
   sudo chown -R kafka:kafka /home/data/kafka
   ```

6. **Запусти Kafka заново**:

   ```bash
   sudo systemctl start kafka
   ```

7. **Проверь, что всё поднялось без ошибок**:

   ```bash
   sudo systemctl status kafka
   journalctl -u kafka -n 50
   ```

---

## 🧪 Проверь через Kafdrop

После запуска снова зайди в Kafdrop ([http://localhost:8090/](http://localhost:8090/)) и убедись, что топики и сообщения на месте.

---

### 💡 Альтернатива: Симлинк

Если **не хочешь править конфиг**, можно сделать так:

```bash
sudo systemctl stop kafka
sudo mv /var/lib/kafka /data/kafka
sudo ln -s /data/kafka /var/lib/kafka
sudo systemctl start kafka
```

Но это скорее временное решение — лучше всё же указать путь явно через `log.dirs`.

