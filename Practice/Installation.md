Конечно, давай развернём сервер Apache Kafka на AlmaLinux с использованием пакетов RPM, доступных через YUM. Это позволит интегрировать Kafka в систему с минимальными усилиями и обеспечить удобное управление через systemd.

---

## ✅ Шаг 1: Установка Java

Apache Kafka требует установленной Java. Для AlmaLinux 9 рекомендуется использовать OpenJDK 21:([hostnextra.com][1])

```bash
sudo dnf install java-21-openjdk-devel -y
```



Проверь, что Java установлена:

```bash
java -version
```



---

## ✅ Шаг 2: Установка Apache Kafka

Скачай и установи Kafka с помощью RPM-пакета:

```bash
sudo rpm -ivh https://github.com/doraeven/kafka-server/releases/download/3.8.0-1.el9.x86_64.rpm
```



Это установит Kafka и необходимые зависимости.

---

## ✅ Шаг 3: Настройка Kafka

Создай необходимые каталоги для данных:

```bash
sudo mkdir -p /var/lib/kafka
sudo mkdir -p /var/lib/zookeeper
```



Скопируй пример конфигурации:

```bash
sudo cp /etc/kafka/server.properties /etc/kafka/server.properties.bak
```



Отредактируй файл конфигурации:

```bash
sudo nano /etc/kafka/server.properties
```



Убедись, что следующие строки настроены:

```properties
broker.id=0
listeners=PLAINTEXT://:9092
advertised.listeners=PLAINTEXT://your_server_ip:9092
log.dirs=/var/lib/kafka
zookeeper.connect=localhost:2181
```



Замените `your_server_ip` на IP-адрес вашего сервера.([idroot.us][2])

---

## ✅ Шаг 4: Настройка systemd для Kafka

Создай unit-файл для Kafka:([orcacore.com][3])

```bash
sudo nano /etc/systemd/system/kafka.service
```



Добавь следующее содержимое:([orcacore.com][3])

```ini
[Unit]
Description=Apache Kafka Server
After=network.target zookeeper.service

[Service]
Type=simple
Environment="JAVA_HOME=/usr/lib/jvm/java-21-openjdk"
ExecStart=/usr/bin/kafka-server-start.sh /etc/kafka/server.properties
ExecStop=/usr/bin/kafka-server-stop.sh
Restart=on-failure

[Install]
WantedBy=multi-user.target
```



Перезагрузи systemd:

```bash
sudo systemctl daemon-reload
```



Включи и запусти Kafka:

```bash
sudo systemctl enable --now kafka
```



---

## ✅ Шаг 5: Проверка работы Kafka

Проверь статус сервиса:

```bash
sudo systemctl status kafka
```



Если сервис работает корректно, ты увидишь сообщение о его активности.

---

## ✅ Шаг 6: Настройка Zookeeper (если требуется)

Если ты используешь Zookeeper, установи его:([hostnextra.com][1])

```bash
sudo dnf install zookeeper-server -y
```



Запусти и включи Zookeeper:

```bash
sudo systemctl enable --now zookeeper
```



Убедись, что Kafka настроен на использование Zookeeper, проверив строку `zookeeper.connect` в конфигурационном файле Kafka.

---

Если возникнут дополнительные вопросы или потребуется помощь с настройкой продакшн-окружения, мониторинга или безопасности, обращайся — я всегда готов помочь!

[1]: https://hostnextra.com/learn/tutorials/installing-apache-kafka-on-almalinux?utm_source=chatgpt.com "Installing Apache Kafka on AlmaLinux 9 - HostnExtra"
[2]: https://idroot.us/install-apache-kafka-almalinux-9/?utm_source=chatgpt.com "How To Install Apache Kafka on AlmaLinux 9 - idroot"
[3]: https://orcacore.com/install-apache-kafka-almalinux-9/?utm_source=chatgpt.com "Install Apache Kafka on AlmaLinux 9 | Full Guide - OrcaCore"

Ты абсолютно прав: начиная с версии Apache Kafka 3.3, был представлен новый консенсусный протокол **KRaft** (Kafka Raft), который позволяет Kafka работать без зависимости от ZooKeeper. Это упрощает архитектуру и повышает производительность. ([infoq.com][1], [developer.confluent.io][2])

---

## ✅ Подтверждение: Kafka без ZooKeeper

С версии **Kafka 4.0** поддержка ZooKeeper была полностью удалена, и Kafka теперь работает исключительно в KRaft-режиме. ([reddit.com][3])

---

## ✅ Настройка Kafka в KRaft-режиме

Если ты используешь Kafka версии 3.3 или выше, тебе не нужно устанавливать ZooKeeper. Просто настрой Kafka для работы в KRaft-режиме.

### Шаги настройки:

1. **Создание уникального идентификатора кластера:**

   ```bash
   ./bin/kafka-storage.sh random-uuid
   ```



2. **Форматирование данных с указанием идентификатора кластера:**

   ```bash
   ./bin/kafka-storage.sh format -t <uuid> -c ./config/kraft/server.properties
   ```



3. **Настройка конфигурации в `server.properties`:**

   ```properties
   process.roles=broker,controller
   node.id=1
   listeners=CONTROLLER://controller1.example.com:9093
   controller.quorum.voters=1@controller1.example.com:9093,2@controller2.example.com:9093,3@controller3.example.com:9093
   ```



4. **Запуск Kafka с использованием KRaft-конфигурации:**

   ```bash
   ./bin/kafka-server-start.sh ./config/kraft/server.properties
   ```



Обрати внимание, что в KRaft-режиме Kafka выполняет функции как брокера, так и контроллера.([medium.com][4])

---

## ✅ Проверка статуса

После запуска Kafka в KRaft-режиме, ты можешь проверить его статус с помощью команды:

```bash
./bin/kafka-topics.sh --bootstrap-server localhost:9092 --describe
```



Если всё настроено правильно, ты увидишь информацию о доступных топиках и их статусе.

---

Если потребуется помощь с настройкой systemd для автоматического запуска Kafka в KRaft-режиме или с другими аспектами, дай знать — я готов помочь!

[1]: https://www.infoq.com/news/2022/10/apache-kafka-kraft/?utm_source=chatgpt.com "Apache Kafka 3.3 Replaces ZooKeeper with the New KRaft Consensus Protocol - InfoQ"
[2]: https://developer.confluent.io/learn/kraft/?utm_source=chatgpt.com "KRaft - Apache Kafka Without ZooKeeper"
[3]: https://www.reddit.com/r/apachekafka/comments/1je7h1q?utm_source=chatgpt.com "Apache Kafka 4.0 released 🎉"
[4]: https://medium.com/%40erkndmrl/how-to-run-apache-kafka-without-zookeeper-3376468ddaa8?utm_source=chatgpt.com "How to run Apache Kafka without Zookeeper | by Erkan Demirel | Medium"

Отлично! У тебя установлена **Apache Kafka 3.9.0**, а это значит — можно (и нужно) запускать Kafka **в режиме KRaft (без ZooKeeper)**. Эта версия уже по умолчанию использует KRaft как рекомендованный способ работы, и ты идёшь по правильному пути 💡

---

## ✅ Что делать дальше (Kafka 3.9, KRaft mode):

Вот пошаговая инструкция под твою систему (AlmaLinux, RPM):

---

### 🔹 1. Сгенерировать Cluster UUID

```bash
/opt/kafka/bin/kafka-storage.sh random-uuid
```

Сохрани UUID, он будет нужен дальше (например: `e7c3b893a1f34a8c913c9c4e248d233e`).

---

### 🔹 2. Отредактировать конфиг для KRaft

Файл обычно: `/etc/kafka/server.properties`
Если нет — найди путь:

```bash
rpm -ql kafka-3.9.0 | grep server.properties
```

Открой для редактирования:

```bash
sudo nano /etc/kafka/server.properties
```

Поменяй или добавь следующее (адаптируй под себя):

```properties
process.roles=broker,controller
node.id=1
controller.listener.names=CONTROLLER
listeners=PLAINTEXT://:9092,CONTROLLER://:9093
advertised.listeners=PLAINTEXT://your_host_or_ip:9092
log.dirs=/var/lib/kafka
controller.quorum.voters=1@localhost:9093
```

🔹 Не забудь заменить `your_host_or_ip` на IP или имя твоего сервера.

---

### 🔹 3. Отформатировать кластер хранилища

```bash
sudo /opt/kafka/bin/kafka-storage.sh format -t e7c3b893a1f34a8c913c9c4e248d233e -c /etc/kafka/server.properties
```

*(где `-t <UUID>` — тот, что ты получил на шаге 1)*

---

### 🔹 4. Создать systemd unit (если ещё нет)

```bash
sudo nano /etc/systemd/system/kafka.service
```

Пример:

```ini
[Unit]
Description=Apache Kafka (KRaft mode)
After=network.target

[Service]
Type=simple
Environment="JAVA_HOME=/usr/lib/jvm/java-21-openjdk"
ExecStart=/opt/kafka/bin/kafka-server-start.sh /etc/kafka/server.properties
ExecStop=/opt/kafka/bin/kafka-server-stop.sh
Restart=on-failure
User=kafka

[Install]
WantedBy=multi-user.target
```

Замените `/opt/kafka` на свой путь, если отличается.

---

### 🔹 5. Запустить Kafka

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now kafka
```

---

### 🔹 6. Проверить, что работает

```bash
sudo systemctl status kafka
journalctl -u kafka -f
```

---

### 🔹 7. Проверка Kafka (CLI)

```bash
/opt/kafka/bin/kafka-topics.sh --bootstrap-server localhost:9092 --list
```

Если видишь пустой список — всё ок, Kafka живёт 🎉

---

## 💬 Готов помочь:

* с настройкой Kafka topics и продюсеров/консьюмеров
* мониторингом через Prometheus или Kafka UI
* безопасным доступом (SASL, TLS)

Скажи, если хочешь продвинуться дальше!

Понимаю твою ситуацию. В RPM-пакете Kafka 3.9.0 для AlmaLinux скрипт `kafka-storage.sh` отсутствует, так как он предназначен для использования при ручной сборке из исходников. Вместо этого, для настройки KRaft-режима в твоём случае можно воспользоваться встроенными средствами Kafka.

---

## ✅ Настройка KRaft-режима без использования `kafka-storage.sh`

В Kafka 3.9.0, установленной через RPM, настройка KRaft-режима осуществляется следующим образом:

1. **Создание уникального идентификатора кластера:**

   Вместо использования `kafka-storage.sh random-uuid`, можно вручную сгенерировать UUID:

   ```bash
   uuidgen
   ```



Сохрани полученный UUID для дальнейшего использования.

2. **Форматирование хранилища:**

   Создай директорию для хранения данных Kafka:

   ```bash
   sudo mkdir -p /var/lib/kafka
   ```



Убедись, что у пользователя, под которым работает Kafka, есть права на эту директорию:

```bash
sudo chown kafka:kafka /var/lib/kafka
```



3. **Настройка конфигурации:**

   Отредактируй файл конфигурации Kafka:

   ```bash
   sudo nano /etc/kafka/server.properties
   ```



Добавь или измени следующие строки:

```properties
process.roles=broker,controller
node.id=1
controller.listener.names=CONTROLLER
listeners=PLAINTEXT://:9092,CONTROLLER://:9093
advertised.listeners=PLAINTEXT://your_host_or_ip:9092
log.dirs=/var/lib/kafka
controller.quorum.voters=1@localhost:9093
```



Замените `your_host_or_ip` на IP или имя твоего сервера.

4. **Запуск Kafka:**

   Перезагрузи systemd:

   ```bash
   sudo systemctl daemon-reload
   ```



Запусти Kafka:

```bash
sudo systemctl enable --now kafka
```



---

## 🔧 Дополнительные рекомендации

* **Проверка статуса:**

  Для проверки статуса Kafka:

```bash
  sudo systemctl status kafka
```



* **Просмотр логов:**

  Для просмотра логов Kafka:

```bash
  journalctl -u kafka -f
```



* **Проверка работы Kafka:**

  Для проверки работы Kafka:([reddit.com][1])

```bash
  /opt/kafka/bin/kafka-topics.sh --bootstrap-server localhost:9092 --list
```



Если команда возвращает пустой список, это означает, что Kafka работает корректно.

---

Если возникнут дополнительные вопросы или потребуется помощь с настройкой продакшн-окружения, мониторинга или безопасности, обращайся — я всегда готов помочь!

[1]: https://www.reddit.com/r/apachekafka/comments/11rxz39?utm_source=chatgpt.com "Kafka forgets it cleaned up its own topics??"

Понимаю твою ситуацию. Ошибка `Start request repeated too quickly` в systemd возникает, когда сервис несколько раз подряд завершился с ошибкой, и systemd ограничивает дальнейшие попытки его запуска.

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

```ini
[Unit]
Description=Apache Kafka Server
After=network.target

[Service]
Type=simple
Environment="JAVA_HOME=/usr/lib/jvm/java-21-openjdk"
ExecStart=/opt/kafka/bin/kafka-server-start.sh /etc/kafka/server.properties
ExecStop=/opt/kafka/bin/kafka-server-stop.sh
Restart=on-failure
User=kafka

[Install]
WantedBy=multi-user.target
```

Замените `/opt/kafka` на путь, где установлена ваша Kafka, если он отличается.

---

## ✅ Шаг 4: Проверить права доступа

Убедитесь, что у пользователя, под которым работает Kafka (например, `kafka`), есть необходимые права доступа к файлам и директориям:

```bash
sudo chown -R kafka:kafka /opt/kafka
sudo chown kafka:kafka /etc/kafka/server.properties
```

Замените `/opt/kafka` на путь, где установлена ваша Kafka, если он отличается.

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

Если после выполнения этих шагов сервис Kafka всё ещё не запускается, пожалуйста, предоставь вывод команды `journalctl -u kafka.service -n 50`, чтобы мы могли более точно определить причину сбоя.

[1]: https://unix.stackexchange.com/questions/517759/how-to-fix-service-start-request-repeated-too-quickly-on-custom-service?utm_source=chatgpt.com "debian - How to fix \".service: Start request repeated too quickly.\" on custom service? - Unix & Linux Stack Exchange"

Отлично, ты нашёл ключевую ошибку — она говорит о **несоответствии параметров `node.id` и `broker.id`** в конфигурации Kafka в **KRaft-режиме**.

---

## 🧠 Объяснение:

В режиме **KRaft (без ZooKeeper)** параметр:

* `broker.id` больше **не используется**
* Вместо него применяется `node.id`

👉 Однако в некоторых системных сборках Kafka (вроде твоей, через RPM) может быть **всё ещё прописан `broker.id` в `server.properties`**, и это вызывает конфликт.

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

## Пример корректной конфигурации (KRaft, standalone):

```properties
process.roles=broker,controller
node.id=1
controller.listener.names=CONTROLLER
listeners=PLAINTEXT://:9092,CONTROLLER://:9093
advertised.listeners=PLAINTEXT://localhost:9092
log.dirs=/var/lib/kafka
controller.quorum.voters=1@localhost:9093
```

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

Если всё сделано правильно — Kafka стартует в режиме KRaft без ошибок.

Готов помочь с проверкой `controller.quorum.voters` или созданием топиков после старта.

Да, ты абсолютно прав 👍

Такая строка:

```
июн 03 15:06:39 dbcs01 systemd[1]: Failed to start Apache Kafka Server.
```

— это просто **остаток от предыдущей неудачной попытки запуска**, которую systemd ещё не «забыл».

---

## ✅ Как убедиться, что Kafka действительно работает сейчас:

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

```bash
/opt/kafka/bin/kafka-topics.sh --bootstrap-server localhost:9092 --list
```

Если команда возвращает пустой список — это **отличный признак**, Kafka в KRaft-режиме работает 🎉

---

Если нужно, могу помочь:

* создать топики
* протестировать продюсер/консьюмер
* настроить запуск Kafka UI или мониторинг

Готов двигаться дальше, когда ты скажешь.

Понимаю твою ситуацию. Ошибка `Start request repeated too quickly` в systemd возникает, когда сервис несколько раз подряд завершился с ошибкой, и systemd ограничивает дальнейшие попытки его запуска.

---

## ✅ Шаг 1: Сбросить статус неудачного запуска

Для начала сбросим статус неудачного запуска сервиса:

```bash
sudo systemctl reset-failed kafka.service
```

Затем попробуем снова запустить сервис:

```bash
sudo systemctl start kafka.service
```

Если сервис снова не запускается, необходимо проверить логи для выявления причины.([reddit.com][1])

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

```ini
[Unit]
Description=Apache Kafka Server
After=network.target

[Service]
Type=simple
Environment="JAVA_HOME=/usr/lib/jvm/java-21-openjdk"
ExecStart=/opt/kafka/bin/kafka-server-start.sh /etc/kafka/server.properties
ExecStop=/opt/kafka/bin/kafka-server-stop.sh
Restart=on-failure
User=kafka

[Install]
WantedBy=multi-user.target
```

Замените `/opt/kafka` на путь, где установлена ваша Kafka, если он отличается.

---

## ✅ Шаг 4: Проверить права доступа

Убедитесь, что у пользователя, под которым работает Kafka (например, `kafka`), есть необходимые права доступа к файлам и директориям:

```bash
sudo chown -R kafka:kafka /opt/kafka
sudo chown kafka:kafka /etc/kafka/server.properties
```

Замените `/opt/kafka` на путь, где установлена ваша Kafka, если он отличается.

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

Если после выполнения этих шагов сервис Kafka всё ещё не запускается, пожалуйста, предоставь вывод команды `journalctl -u kafka.service -n 50`, чтобы мы могли более точно определить причину сбоя.

[1]: https://www.reddit.com/r/Fedora/comments/ee5134?utm_source=chatgpt.com "I can't get docker's service to start."

Отлично, теперь видно настоящую причину сбоя:

> **`java.lang.RuntimeException: No readable meta.properties files found.`**

Это значит, что Kafka **в KRaft-режиме** ожидает наличие файла `meta.properties` в папке `log.dirs` (обычно `/var/lib/kafka`), но не находит его.

---

## 🔧 Почему так происходит?

В KRaft-режиме Kafka **обязательно требует "форматирования" хранилища**, чтобы создать кластерный мета-файл `meta.properties`. Это действие обычно выполняется **однократно перед первым запуском**, вручную.

Раньше это делалось скриптом `kafka-storage.sh` — но в твоей RPM-инсталляции он может отсутствовать, и нужно выполнить эквивалент вручную.

---

## ✅ Решение: инициализация KRaft-хранилища вручную

1. **Сгенерируй UUID кластера:**

```bash
uuidgen
```

Скопируй этот UUID, он понадобится на следующем шаге.

---

2. **Создай `meta.properties` вручную:**

Создай директорию (если ещё нет):

```bash
sudo mkdir -p /var/lib/kafka
sudo chown kafka:kafka /var/lib/kafka
```

Создай файл `/var/lib/kafka/meta.properties` с содержимым:

```
version=0
cluster.id=<UUID>
node.id=1
```

👉 где `<UUID>` — это то, что сгенерировал `uuidgen`.

---

3. **Проверь разрешения:**

```bash
sudo chown kafka:kafka /var/lib/kafka/meta.properties
```

---

4. **Удалить мусор, если были старые попытки:**

```bash
sudo rm -rf /var/lib/kafka/*-log
```

---

5. **Перезапусти Kafka:**

```bash
sudo systemctl daemon-reexec
sudo systemctl restart kafka
sudo systemctl status kafka
```

---

## ✅ Проверка

Если всё прошло успешно, ты увидишь в логах строки вроде:

```
INFO KafkaServer started (kafka.server.KafkaServer)
```

И можно будет выполнять команды типа:

```bash
kafka-topics.sh --bootstrap-server localhost:9092 --list
```

---

Если хочешь — могу помочь тебе с созданием KRaft-кластера на 1 или более нод, с контроллерами и мониторингом.

Отлично, ты близко! Ошибка:

```
Error: node ID not found in /var/lib/kafka/
```

означает, что Kafka **не находит `meta.properties` с нужным `node.id`**, соответствующим тому, что указан в `server.properties`.

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

## 🧼 Если не совпадает или файл битый:

1. Удали старые данные:

   ```bash
   sudo rm -rf /var/lib/kafka/*
   ```

2. Пересоздай `meta.properties`:

   ```bash
   sudo nano /var/lib/kafka/meta.properties
   ```

   Вставь:

   ```
   version=0
   cluster.id=<твой-сгенерированный-UUID>
   node.id=1
   ```

3. Выстави права:

   ```bash
   sudo chown -R kafka:kafka /var/lib/kafka
   ```

---

## ✅ Проверка:

* `server.properties` должен содержать:

  ```properties
  node.id=1
  log.dirs=/var/lib/kafka
  ```

* `meta.properties` — **точно тот же `node.id=1`**.

---

После этого снова:

```bash
sudo systemctl restart kafka
sudo journalctl -u kafka -f
```

Если всё в порядке, Kafka взлетит. Хочешь — могу показать, как генерировать `meta.properties` автоматически, даже без `kafka-storage.sh`.

Хорошо, теперь мы знаем, что:

* `node.id=1` указан **и в `server.properties`**, и в **`/var/lib/kafka/meta.properties`**
* Права выставлены корректно
* Но Kafka всё ещё падает с ошибкой:

> `RuntimeException: Error: node ID not found in /var/lib/kafka/`

---

### 🔍 Возможные причины этой ошибки:

#### ✅ 1. **Неверный путь в `log.dirs`**

Убедись, что в `server.properties` путь в `log.dirs` **точно совпадает** с тем, где лежит `meta.properties`. Например:

```properties
log.dirs=/var/lib/kafka
```

📌 **Внимание**: если в `log.dirs` указан **другой путь** (например, `/tmp/kafka-logs` или `/var/log/kafka`), Kafka будет искать `meta.properties` там — и не найдет.

Проверь:

```bash
grep log.dirs /etc/kafka/server.properties
```

---

#### ✅ 2. **Остатки мусора в других лог-директориях**

Если в `log.dirs` указано **несколько директорий через запятую**, Kafka требует наличие корректного `meta.properties` в каждой. Это также может вызвать ошибку `node ID not found`.

Проверь:

```bash
cat /etc/kafka/server.properties | grep log.dirs
```

Если там что-то вроде:

```
log.dirs=/var/lib/kafka,/tmp/kafka-logs
```

→ это приведёт к падению.

🔧 Решение: оставь только одну папку:

```
log.dirs=/var/lib/kafka
```

---

#### ✅ 3. **Случайная невидимая ошибка в `meta.properties`**

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

#### ✅ 4. **Kafka работает от другого пользователя**

Если Kafka запущена как `kafka`, а `meta.properties` принадлежит `root`, он может быть **недоступен для чтения**. Хотя ты писал, что права стоят — стоит перепроверить:

```bash
ls -l /var/lib/kafka/meta.properties
ls -ld /var/lib/kafka
```

Оба должны принадлежать `kafka:kafka`.

---

### 🧪 Дополнительная проверка: логов больше

Получи **расширенные логи запуска**:

```bash
journalctl -u kafka -n 100 --no-pager
```

Если не поможет — покажи:

* Содержимое `log.dirs`
* Полный `meta.properties`
* Точную ошибку из свежего лога

Готов посмотреть и разобрать с тобой.

Спасибо, ты всё показал предельно чётко — и проблема действительно интересная. У тебя:

* `log.dirs=/var/lib/kafka/` (с **завершающим слэшем**)
* `meta.properties` лежит в `/var/lib/kafka/`, с правильным `node.id=1`
* Права и формат файла корректны

Но Kafka **всё равно говорит, что не может найти `node.id` в `/var/lib/kafka/`**, что довольно парадоксально.

---

### 💡 Что, скорее всего, происходит

Kafka **может быть чувствительна к завершающему слэшу `/`** в `log.dirs`. Такое случается с некоторыми путями в Java/Scala-проектах: путь `"/var/lib/kafka"` и `"/var/lib/kafka/"` **считаются разными**, и могут сравниваться дословно.

### ✅ Шаг 1: Убери завершающий `/` в `log.dirs`

Открой:

```bash
sudo nano /etc/kafka/server.properties
```

И замени:

```ini
log.dirs=/var/lib/kafka/
```

на:

```ini
log.dirs=/var/lib/kafka
```

Сохрани.

---

### ✅ Шаг 2: Перезапусти Kafka

```bash
sudo systemctl daemon-reexec
sudo systemctl restart kafka
```

Затем проверь лог:

```bash
journalctl -u kafka -f
```

---

### 📌 Почему это важно

Kafka в KRaft-режиме проверяет, что:

* Каталог `log.dirs` существует
* В нём есть файл `meta.properties`
* Этот файл имеет `node.id`, соответствующий `server.properties`

Если при сравнении путей участвует `Path.equals()` (в Java), то `"/var/lib/kafka"` и `"/var/lib/kafka/"` — **не эквивалентны**. Это может приводить к тому, что Kafka не находит нужную директорию с `meta.properties`.

---

Если это действительно решит проблему — будет очень ценный вывод для других, кто запускает Kafka RPM в KRaft-режиме под RHEL-подобными системами.

Понимаю, что ситуация остаётся непростой. Ошибка `No readable meta.properties files found` указывает на то, что Kafka не может найти или прочитать файл `meta.properties`, который необходим для работы в KRaft-режиме. Это может происходить по нескольким причинам, даже если файл существует и имеет правильные права доступа.

---

### 🔍 Возможные причины и решения

#### 1. **Проблемы с правами доступа**

Хотя вы уже проверяли права доступа, стоит убедиться, что процесс Kafka имеет необходимые разрешения для чтения файла `meta.properties`. Попробуйте временно установить более открытые права на файл и директорию:

```bash
sudo chmod -R 777 /var/lib/kafka
```



Затем перезапустите сервис Kafka:

```bash
sudo systemctl restart kafka
```



Если это решит проблему, возможно, потребуется настроить SELinux или AppArmor, чтобы разрешить доступ к этим файлам.

#### 2. **Проблемы с SELinux**

Если на вашей системе включен SELinux, он может блокировать доступ к файлу `meta.properties`. Проверьте логи SELinux:

```bash
sudo ausearch -m avc -ts recent
```



Если вы обнаружите соответствующие ошибки, можно временно отключить SELinux для диагностики:

```bash
sudo setenforce 0
```



Если это решит проблему, вам нужно будет настроить соответствующие правила SELinux.

#### 3. **Проблемы с файловой системой**

Убедитесь, что файловая система, на которой находится `/var/lib/kafka`, не имеет ошибок:

```bash
sudo dmesg | grep -i ext4
```



Если вы обнаружите ошибки, возможно, потребуется выполнить проверку и восстановление файловой системы.

#### 4. **Проблемы с версией Kafka**

Убедитесь, что используемая вами версия Kafka поддерживает KRaft-режим. Согласно [Stack Overflow](https://stackoverflow.com/questions/74223969/node-id-vs-broker-id-in-kraft-mode), начиная с версии Kafka 2.8.0, KRaft-режим стал доступен как экспериментальная функция, а с версии 3.0.0 — как стабильная.([stackoverflow.com][1])

#### 5. **Проблемы с конфигурацией**

Проверьте, что в файле `server.properties` указаны правильные параметры:

```ini
log.dirs=/var/lib/kafka
node.id=1
controller.quorum.voters=1@localhost:9093
listeners=PLAINTEXT://localhost:9092
log.dirs=/var/lib/kafka
```



Убедитесь, что параметр `log.dirs` указывает на правильный путь и что в этом каталоге находится файл `meta.properties`.

---

### 🔄 Дополнительные шаги

* Проверьте логи Kafka для получения более подробной информации о причине ошибки:

  ```bash
  sudo journalctl -u kafka -f
  ```



* Если возможно, попробуйте запустить Kafka в другом каталоге данных, чтобы исключить проблемы с текущим расположением.

* Если у вас есть возможность, попробуйте установить Kafka на другой машине или виртуальной машине, чтобы проверить, сохраняется ли проблема.

---

Если после выполнения этих шагов проблема сохраняется, пожалуйста, предоставьте дополнительные детали, такие как:

* Полный вывод команды `journalctl -u kafka -f`.

* Содержимое файлов `server.properties` и `meta.properties`.

* Вывод команды `ls -l /var/lib/kafka`.

С этими данными я смогу предоставить более точные рекомендации.

[1]: https://stackoverflow.com/questions/74223969/node-id-vs-broker-id-in-kraft-mode?utm_source=chatgpt.com "apache kafka - node.id vs broker.id in KRaft mode - Stack Overflow"

Спасибо за данные, ситуация понятнее.
Обращу внимание на пару важных моментов:

---

### 1. Противоречие в конфигурации:

У вас в `server.properties` есть строка:

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

```bash
sudo -u kafka /usr/bin/kafka-storage.sh format -t <cluster-id> -c /etc/kafka/server.properties
```

* `<cluster-id>` нужно получить с помощью команды:

```bash
/usr/bin/kafka-storage.sh random-uuid
```

* После форматирования директории с логами Kafka запустится в KRaft-режиме.

---

### 3. Если хотите работать с ZooKeeper:

* Верните параметр `zookeeper.connect=localhost:2181`
* Удалите или закомментируйте параметры `process.roles`, `node.id`, `controller.quorum.voters` и `controller.listener.names` из `server.properties`
* Убедитесь, что у вас запущен и работает ZooKeeper (его нужно установить и запустить отдельно, если он не установлен)

---

### 4. Пример базового `server.properties` для KRaft:

```ini
# listeners and advertised listeners
listeners=PLAINTEXT://:9092,CONTROLLER://:9093
advertised.listeners=PLAINTEXT://10.30.40.150:9092

# basic performance configs
num.network.threads=3
num.io.threads=8
socket.send.buffer.bytes=102400
socket.receive.buffer.bytes=102400
socket.request.max.bytes=104857600

# log dirs
log.dirs=/var/lib/kafka

# Kafka raft mode configs
process.roles=broker,controller
node.id=1
controller.quorum.voters=1@localhost:9093
controller.listener.names=CONTROLLER

# replication factors etc
num.partitions=1
offsets.topic.replication.factor=1
transaction.state.log.replication.factor=1
transaction.state.log.min.isr=1

# retention policies
log.retention.hours=168
log.retention.check.interval.ms=300000

# zookeeper should be removed or commented out here!
# zookeeper.connect=localhost:2181
```

---

### 5. Итоговые действия для KRaft режима:

```bash
# 1. Получаем cluster id
cluster_id=$(/usr/bin/kafka-storage.sh random-uuid)

# 2. Форматируем директорию с логами
sudo -u kafka /usr/bin/kafka-storage.sh format -t $cluster_id -c /etc/kafka/server.properties

# 3. Проверяем права на директорию /var/lib/kafka (должен быть пользователь kafka)
sudo chown -R kafka:kafka /var/lib/kafka

# 4. Перезапускаем Kafka
sudo systemctl restart kafka

# 5. Проверяем статус
sudo systemctl status kafka
```

---

Если `kafka-storage.sh` у вас отсутствует или лежит в другом месте, его можно найти командой:

```bash
sudo find / -name kafka-storage.sh
```

---

Если хотите, могу помочь с полным примером сервиса systemd и точными командами под AlmaLinux. Просто скажите!

---

Если будет интересно, могу рассказать, как запустить Kafka с ZooKeeper — это чуть проще, но без преимуществ KRaft.

Отлично, это уже здорово — Kafka запускается и начинает инициализацию KRaft-кластера!

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
4. Попробуйте подключиться к Kafka с клиента, например, с помощью `kafka-console-producer.sh` или `kafka-console-consumer.sh`.

---

### Если вдруг сервис продолжит падать, рекомендую:

* Заглянуть в логи полной глубиной:

  ```bash
  sudo journalctl -u kafka -b
  ```
* Проверить, что права на `/var/lib/kafka` и все вложенные файлы принадлежат пользователю `kafka` (который запускает процесс).
* Убедиться, что в `server.properties` нет конфликтующих параметров (особенно связанных с ZooKeeper).

---

### Подытоживая:

* Запуск с KRaft — это современный режим без ZooKeeper, и лог с `bootstrap record` — нормальный признак старта.
* Если нет ошибок после этого — кластер живой и работоспособный!
* Можно приступать к созданию топиков, публикации и потреблению сообщений.

---

Если хочешь — могу помочь с примером простых команд для проверки работы Kafka и настройки продюсеров/консьюмеров.
Напиши, если надо!

Отлично! 🎉 Kafka действительно **запустилась успешно**, судя по:

* `LISTEN` на портах **9092** (для клиентов) и **9093** (для KRaft-контроллера)
* Отсутствию ошибок и стабильной работе systemd-сервиса
* Логам `QuorumController`, которые показывают нормальную инициализацию

---

### Что теперь можно сделать:

Чтобы убедиться, что всё работает:

#### ✅ 1. Создай топик:

```bash
/usr/bin/kafka-topics.sh --create \
  --bootstrap-server localhost:9092 \
  --topic test-topic \
  --partitions 1 \
  --replication-factor 1
```

#### ✅ 2. Посмотри список топиков:

```bash
/usr/bin/kafka-topics.sh --list \
  --bootstrap-server localhost:9092
```

#### ✅ 3. Запусти продюсер:

```bash
/usr/bin/kafka-console-producer.sh --bootstrap-server localhost:9092 --topic test-topic
```

> Введи что-нибудь — оно отправится в Kafka.

#### ✅ 4. Запусти консьюмер в другом терминале:

```bash
/usr/bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test-topic --from-beginning
```

> Ты должен увидеть сообщения, которые ввёл продюсером.

---

Если всё это работает — у тебя полностью **функциональный Kafka-сервер в режиме KRaft**. 🔥
Дальше можно:

* Подключать Python-приложения (через `confluent-kafka`, `kafka-python`, `faust`, и т.д.)
* Добавлять ACL, REST-прокси, схематизацию (Confluent Schema Registry)
* Переходить к балансировке или настройке нескольких брокеров

Если захочешь — помогу и с этим.

Отличная работа!

Ты абсолютно прав — и хорошо подметил!

📦 При установке **Kafka из RPM** (как у тебя: `kafka-3.9.0-1.el8.x86_64`), все утилиты действительно кладутся в:

```
/usr/bin/kafka/
```

а не в `/usr/bin/` напрямую, как бывает при установке вручную из архивов с сайта Apache.

---

### Примеры с учетом твоего пути:

#### ✅ Создание топика:

```bash
/usr/bin/kafka/kafka-topics.sh --create \
  --bootstrap-server localhost:9092 \
  --topic test-topic \
  --partitions 1 \
  --replication-factor 1
```

#### ✅ Список топиков:

```bash
/usr/bin/kafka/kafka-topics.sh --list \
  --bootstrap-server localhost:9092
```

#### ✅ Продюсер:

```bash
/usr/bin/kafka/kafka-console-producer.sh \
  --bootstrap-server localhost:9092 \
  --topic test-topic
```

#### ✅ Консьюмер:

```bash
/usr/bin/kafka/kafka-console-consumer.sh \
  --bootstrap-server localhost:9092 \
  --topic test-topic \
  --from-beginning
```

---

Если часто будешь запускать эти утилиты, можешь добавить себе алиасы в `~/.bashrc` или `~/.bash_aliases`, например:

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

Если что — помогу с настройкой клиентских библиотек или с multi-broker конфигурацией!

Отличный вопрос! Давай пошагово, на практике. Сейчас ты на самом пороге — Kafka у тебя запущена, продюсер и консьюмер работают. Осталось только **отправить сообщение** и **получить его**.

---

## ✅ Что сделать:

### 1. **Открой два терминала**

#### 🟩 Терминал 1 — Consumer (слушатель)

Запусти консьюмера, чтобы он слушал `test-topic`:

```bash
/usr/bin/kafka/kafka-console-consumer.sh \
  --bootstrap-server localhost:9092 \
  --topic test-topic \
  --from-beginning
```

Он будет "висеть" в ожидании сообщений.

---

#### 🟦 Терминал 2 — Producer (отправитель)

Теперь открой второй терминал и запусти продюсера:

```bash
/usr/bin/kafka/kafka-console-producer.sh \
  --bootstrap-server localhost:9092 \
  --topic test-topic
```

Тебе откроется пустая строка. Начни вводить текст:

```
Привет Kafka!
Это тестовое сообщение.
```

После ввода каждого сообщения нажимай **Enter** — сообщение будет отправлено в Kafka.

---

### 🔄 Результат

В **первом терминале (консьюмер)** ты сразу же увидишь эти сообщения на экране:

```
Привет Kafka!
Это тестовое сообщение.
```

---

## ❗️ Если не видно сообщений:

* Убедись, что **топик существует** и правильно указан в обеих командах.
* Используй флаг `--from-beginning`, если хочешь читать и старые сообщения.
* Консьюмер должен быть запущен **до или во время** отправки, чтобы успеть "услышать".

---

## 📌 Резюме

Kafka уже работает. Сейчас ты:

* Создал топик
* Подключился как продюсер
* Получил сообщение как консьюмер

Поздравляю, ты сделал первый круг в Kafka 🎉

Если хочешь, могу показать как это делать программно — например, на Python с помощью `kafka-python` или `confluent-kafka`.

Да, ты прав: существует несколько удобных веб-интерфейсов для работы с Apache Kafka. Вот некоторые из них:

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

Если тебе нужно выбрать подходящий инструмент, я могу помочь с установкой и настройкой одного из них. Какой из этих интерфейсов тебе наиболее интересен?

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

### 🚀 Как запустить Kafdrop (если у тебя уже работает Kafka):

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
   java -jar kafdrop-<version>.jar --kafka.brokerConnect=localhost:9092
   ```

---

### 🔍 После запуска:

Перейди в браузере на [http://localhost:9000](http://localhost:9000) — там будет веб-интерфейс.

---

Если хочешь, могу помочь тебе прямо сейчас его поднять — с Docker или без. Скажи только, Kafka у тебя точно доступна на `localhost:9092`?

Отлично, что Kafka теперь работает и доступна на порту 9092. Давай теперь настроим Kafdrop — удобный веб-интерфейс для работы с Kafka.

---

### 🚀 Как запустить Kafdrop

#### Вариант 1: Через Docker (самый простой способ)

Если у тебя установлен Docker, запусти Kafdrop одной командой:

```bash
docker run -d --rm -p 9000:9000 \
  -e KAFKA_BROKERCONNECT=localhost:9092 \
  obsidiandynamics/kafdrop
```

После этого открой браузер и перейди по адресу: [http://localhost:9000](http://localhost:9000).

---

#### Вариант 2: Запуск через JAR-файл

Если Docker не установлен, можно запустить Kafdrop напрямую из JAR-файла:

1. Скачай JAR-файл с [GitHub Releases](https://github.com/obsidiandynamics/kafdrop/releases).

2. Запусти его командой:

   ```bash
   java -jar kafdrop-<version>.jar --kafka.brokerConnect=localhost:9092
   ```

3. Открой браузер и перейди по адресу: [http://localhost:9000](http://localhost:9000).

---

### 🧭 Что можно делать в веб-интерфейсе Kafdrop

После запуска Kafdrop ты сможешь:

* Просматривать список топиков Kafka.
* Создавать и удалять топики.
* Просматривать сообщения в топиках (поддерживаются форматы JSON, Avro, Protobuf).
* Смотреть информацию о брокерах и их состоянии.
* Анализировать группы потребителей и их смещения.

---

Если хочешь, я могу помочь тебе с созданием первого топика, отправкой и получением сообщений. Напиши, если нужна помощь!

[![Kafdrop — Kafka Web UI — Quick Start ...](https://images.openai.com/thumbnails/b30338276d8e1da5d8e029b79e2f3b01.png)](https://medium.com/%40jeyagan/kafdrop-kafka-web-ui-quick-start-example-42beb5efd7b7)

Конечно! Чтобы скачать JAR-файл Kafdrop с GitHub Releases, выполните следующие шаги:

1. **Перейдите на страницу релизов Kafdrop**:
   Откройте [https://github.com/obsidiandynamics/kafdrop/releases](https://github.com/obsidiandynamics/kafdrop/releases).

2. **Выберите нужную версию**:
   На странице релизов найдите и выберите версию Kafdrop, которую вы хотите скачать.

3. **Скачайте JAR-файл**:
   В разделе "Assets" найдите файл с именем вида `kafdrop-<version>.jar` и скачайте его.

   Например, для версии 4.0.0 файл будет называться `kafdrop-4.0.0.jar`.

4. **Запустите Kafdrop**:
   После скачивания JAR-файла, откройте терминал и выполните команду:

   ```bash
   java -jar kafdrop-<version>.jar --kafka.brokerConnect=localhost:9092
   ```


Замените `<version>` на номер версии JAR-файла, который вы скачали.

5. **Откройте веб-интерфейс**:
   После запуска Kafdrop, откройте веб-браузер и перейдите по адресу: [http://localhost:9000](http://localhost:9000).

Теперь вы можете использовать веб-интерфейс Kafdrop для управления и мониторинга вашего кластера Kafka.

Если у вас возникнут дополнительные вопросы или потребуется помощь, не стесняйтесь обращаться!

Ошибка `UnsupportedClassVersionError` говорит о том, что версия Java, с которой вы запускаете JAR-файл, слишком старая и не поддерживает версию байткода, с которой был скомпилирован Kafdrop.

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

```bash
java -jar kafdrop-4.1.0.jar --kafka.brokerConnect=localhost:9092
```

---

Если нужно — могу помочь с установкой и настройкой Java 17. Напишите!

Ошибка `java.net.BindException: Адрес уже используется` означает, что порт, на котором Kafdrop пытается запуститься, уже занят другим процессом.

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

---

Проверьте и отпишись, помогло ли!

Понятно, ClickHouse уже занял порт 9000, и убивать его нельзя.

У Kafdrop можно изменить порт, на котором он слушает, с помощью параметра `--server.port`.

Просто запусти Kafdrop с указанием другого порта, например 8080:

```bash
java -jar kafdrop-4.1.0.jar --kafka.brokerConnect=localhost:9092 --server.port=8080
```

После этого веб-интерфейс Kafdrop будет доступен по адресу: `http://localhost:8080`.

Если хочешь, можно выбрать любой свободный порт.

---

Если хочешь, могу помочь подобрать свободный порт или сделать проверку на твоей машине.

Ошибка `java.net.BindException: Адрес уже используется` при попытке запустить Kafdrop на порту 8080 означает, что этот порт уже занят другим процессом.

Проверим, что слушает 8080:

```bash
sudo ss -plnt | grep :8080
```

Если там что-то есть, нужно либо остановить этот процесс, либо выбрать свободный порт для Kafdrop.

Чтобы быстро найти свободный порт, можно использовать команду:

```bash
comm -23 <(seq 8000 8100 | sort) <(sudo ss -plnt | awk '{print $4}' | grep -oE '[0-9]+$' | sort -n)
```

Она выведет список портов в диапазоне 8000-8100, которые не заняты.

Попробуй запустить Kafdrop, например, на порту 8090:

```bash
java -jar kafdrop-4.1.0.jar --kafka.brokerConnect=localhost:9092 --server.port=8090
```

Если и там ошибка — повтори проверку занятого порта и выбери другой.

---

Если хочешь, помогу проверить и подобрать порт — скинь вывод `sudo ss -plnt`.

Отлично, список портов свободен — значит дело не в занятости порта, а, вероятно, в **двойной инициализации HTTP-сервера внутри самого Kafdrop**, или конфликте с настройками Spring Boot Actuator.

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

---

Хочешь, могу помочь сделать systemd unit-файл, чтобы запускать Kafdrop как сервис.

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

Если хочешь, могу помочь тебе:

* подключить его к реальным продюсерам/консьюмерам,
* или оформить запуск через `systemd`, чтобы всё автоматически поднималось при перезагрузке.

Отличный вопрос, и ты **очень вовремя** его задал — переместить кластерные данные Kafka **можно и безопасно**, но при соблюдении нескольких чётких шагов. Это особенно важно, если кластер уже запущен и инициализирован (как у тебя).

---

## 🔄 Вариант: Перенос данных кластера в новое место

Ты **не обязан уничтожать кластер** — достаточно:

---

### ✅ Шаги по переносу `/var/lib/kafka` на новый диск

Предположим, что ты хочешь переместить данные в, например, `/data/kafka`.

1. **Останови Kafka**:

   ```bash
   sudo systemctl stop kafka
   ```

2. **Создай новый путь, если он не существует**:

   ```bash
   sudo mkdir -p /data/kafka
   sudo chown -R kafka:kafka /data/kafka
   ```

3. **Перемести текущие данные**:

   ```bash
   sudo rsync -avh /var/lib/kafka/ /data/kafka/
   ```

   ⚠️ Убедись, что внутри `/data/kafka` после копирования остались файлы вроде `meta.properties`, `log-start-offset-checkpoint`, директории `@topic@partition` и пр.

4. **Измени конфигурацию Kafka**:

   В файле `/etc/kafka/server.properties` измени строку:

   ```properties
   log.dirs=/data/kafka
   ```

5. **Проверь владельца и права**:

   ```bash
   sudo chown -R kafka:kafka /data/kafka
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

---

Если хочешь — могу помочь оформить это в виде `systemd`-таймера или `pre-start` скрипта, если потребуется автоматизация в будущем.

Готов двигаться дальше к программному использованию, когда будешь!

