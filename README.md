# Лабораторная №2
## Выполнили: Давиденко Дарья, Нечухаева Яна, Туреев Содном, группа Б9118-09.03.04прогин

---

## Установка и настройка Squid proxy
Сначала устанавливаем squid для дальнейшего использования в качестве прокси-сервера с помощью следующей команды:

```properties
sudo apt install squid
```

После установки, squid необходимо настроить. Для этого открывем файл .conf с помощью следующей команды:

```properties
sudo nano /etc/squid/squid.conf
```

В открытом файле необходимо нажимаем Ctrl+W для того, чтобы чтобы найти необходимое место и прописать:
```properties
include /etc/squid/conf.d/*
```

В найденном месте ниже добавим строчки и поле сохраняем измененный файл:
 ```properties
 auth_param basic program /usr/lib/squid3/basic_ncsa_auth /etc/squid/passwords`
 auth_param basic realm proxy`
 acl authenticated proxy_auth REQUIRED`
 acl localnet src  176.59.142.172
 http_access allow authenticated
 ```
 176.59.142.172 - ip адрес локальной машины.
 ![squid](https://user-images.githubusercontent.com/91045595/166097976-d1335246-0b82-45aa-888f-a9216703c2da.jpg)


### Установка Apache 
Устанавливаем Apache для дальнейшего осуществления передачи данных при запросе и настраиваем аутентифиуацию по паролю
```properties
sudo apt install apache2-utils
sudo htpasswd -c /etc/squid/passwords *squid_username*
```

### Запуск сервера
По умолчанию squid слушает 3128 порт
```properties
sudo systemctl start squid
sudo systemctl enable squid
sudo ufw allow 3128
```
Видим, что прокси успешно работает:
![Сингапур](https://user-images.githubusercontent.com/91045595/166098143-ee66369c-ccdb-4f36-8e35-db8b2d6537e1.jpg)


## Elasticsearch

### Установка Java
Установим Java с помощью следующих команд:
```properties
sudo apt update
sudo apt install default-jre
sudo apt install default-jdk
```

### Установка Elasticsearch
Elasticsearch используется для хранения, анализа, поиска по логам.
Для начала используем cURL, инструмент командной строки для передачи данных с помощью URL, для импорта открытого ключа Elasticsearch GPG в APT. Так же используем аргументы -fsSL для подавления всех текущих и возможных ошибок (кроме сбоя сервера), а также, чтобы разрешить cURL подать запрос на другой локации при переадресации. Выводим результаты команды cURL в программу apt-key, которая добавит открытый ключ GPG в APT

```properties
curl -fsSL https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
```

Добавим список источников Elastic в каталог sources.list.d, где APT будет искать новые источники:

```properties
echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list
```

Обновляем список пакетов, чтобы APT мог прочитать новый источник Elastic и устанавливаем Elasticksearch с помощью следующей команды:

```properties
sudo apt update
sudo apt install elasticsearch
```

### Настройка Elasticsearch
Для настройки Elasticsearch мы отредактируем файлы конфигурации. В Elasticsearch имеется три файла конфигурации:

* elasticsearch.yml для настройки Elasticsearch, главный файл конфигурации
* jvm.options для настройки виртуальной машины Elasticsearch Java Virtual Machine (JVM)
* log4j2.properties для настройки журнала Elasticsearch

Открываем файл elasticsearch.yml для изменения конфигураций. Файл elasticsearch.yml предоставляет варианты конфигурации для вашего кластера, узла, пути, памяти, сети, обнаружения и шлюза. Нам необходимо изменить настройки только для хоста сети.
```properties
sudo nano /etc/elasticsearch/elasticsearch.yml
```
Для ограничения доступа и повышения безопасности находим строку с указанием network.host и убераем с нее значок комментария, после чего заменяем значение на 0.0.0.0, чтобы она выглядела следующим образом: "network.host: 0.0.0.0".
Elasticsearch формирует одноузловой кластер: discovery.type: single-node.
Включаем функции безопасности Elasticsearch на узле: xpack.security.enabled: true.

### Запуск Elasticsearch
Запукаем Elasticsearch с помощью следующих команд:
```properties
sudo systemctl start elasticsearch
sudo systemctl enable elasticsearch
```

![image](https://user-images.githubusercontent.com/91045595/166140286-d3e0eb64-d0ba-4ffc-9d88-602c9c973ec1.png)

### Создание пользователей

```properties
sudo -u root /usr/share/elasticsearch/bin/elasticsearch-setup-passwords auto
```

Сохраняем вывод

```
Changed password for user apm_system
PASSWORD apm_system = Ff7GWmJSS4X45NewBNZN

Changed password for user kibana_system
PASSWORD kibana_system = OZ7w09XTeHBg3rfihAsm

Changed password for user kibana
PASSWORD kibana = OZ7w09XTeHBg3rfihAsm

Changed password for user logstash_system
PASSWORD logstash_system = AKtK2rmOmkAwWfL17jCh

Changed password for user beats_system
PASSWORD beats_system = ymq4MVVHCdDIVBohH5l7

Changed password for user remote_monitoring_user
PASSWORD remote_monitoring_user = RbkSKKbivgZOoq02plda

Changed password for user elastic
PASSWORD elastic = nrRO28jYIplnaEP3JBou
```

## Kibana

### Установка Kibana
Установим Kibana, которая представляет собой удобную и красивую web панель для работы с логами, с помощью следующей команды:
```properties
sudo apt install kibana
```

### Настройка Kibana
Откроем файл конфигурации:
```properties
sudo nano /etc/kibana/kibana.yml
```
Натроим username и укажем, что Elasticsearch должна работать на порту 9200, а Kibana на 5601:
```properties
elasticsearch.username: "kibana_system"
elasticsearch.host: "0.0.0.0:9200"
kibana.port: 5601
```
### Запуск Kibana
Запустим Kibana с помощью следующих команд:
```properties
sudo systemctl start kibana
sudo systemctl enable kibana
```

![image](https://user-images.githubusercontent.com/91045595/166140336-fbfebee1-7426-4aa8-9d95-4458964af2ce.png)

### Добавление пароля

```properties
sudo -u root /usr/share/kibana/bin/kibana-keystore create
sudo -u root /usr/share/kibana/bin/kibana-keystore add elasticsearch.password
```

Вписываем пароль, который получили при создании пользователей
```properties
Changed password for user kibana_system
PASSWORD kibana_system = OZ7w09XTeHBg3rfihAsm
```
Заходим в Elasticsearch, создаем суперпользователя

![elastic](https://user-images.githubusercontent.com/91045595/166099919-21b65f88-cd66-4dcc-9669-dd402a8be0a1.jpg)


## Logstash

### Установка Logstash
Устанавливаем Logstash- сервис для сбора логов и отправки их в Elasticsearch.
```properties
sudo apt install logstash
```

### Настройка Logstash
Открываем файл конфигураций:
```properties
sudo nano /etc/logstash/conf.d/logstash.conf
```

Указываем, что принимаем информацию на 5044 порт, формируем функцию, где в дальнейшем будут находиться правила,  описываем передачу данных в Elasticsearch и сохраняем.

```properties
input { 
    beats { 
        port => 5044 
    }
}

filter {
  **Место для правил Grok**
}

output {
  elasticsearch {
    hosts => ["139.59.247.231:9200"]
    manage_template => false 
    index => "%{[@metadata][beat]}-%{[@metadata[version]}-%{+YYYY.MM.dd}"
    user => "dasha"
    password => "123456"
  }
}

```

Проверяем синтаксис, чтобы в дальнейшем не появилиь ошибки

```properties
sudo -u logstash /usr/share/logstash/bin/logstash --path.settings /etc/logstash -t
```

### Запуск Logstash
Запускаем Logstash с помощью следующих команд:

```properties
sudo systemctl start logstash
sudo systemctl enable logstash
```

## Filebeat

### Установка Filebeat

Заходим на прокси сервер

```properties
curl -fsSL https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list
sudo apt install filebeat
```

### Настройка Filebeat
Изменяем файл конфигураций:
```
sudo filebeat modules enable system
sudo nano /etc/filebeat/filebeat.yml
```

Комментируем строчки "output.elasticsearch" и "hosts: ["localhost:9200"]"

Раскомментируем строчки "output.logstash" и "hosts: ["localhost:5044"]"

Изменим localhost на ip нашего сервера с ELK
![elasticOutput](https://user-images.githubusercontent.com/91045595/166100621-15ed5fd9-3402-4d3e-a141-859b971a96cd.jpg)
![LogstashOutput](https://user-images.githubusercontent.com/91045595/166100623-b61d5d44-15e1-4d57-8c75-1a994f44d3a5.jpg)


Проверяем конфигурацию "sudo filebeat -e test output"
Так как мы отправляем события в Logstash, то необходимо вручную загрузить конвейеры загрузки. Для этого запустим команду:
```properties
sudo filebeat setup --pipelines --modules system
```
Установим index templates для Elasticsearch вручную.

Чтобы загрузить шаблон индекса вручную, запустим команду установки. Требуется подключение к Elasticsearch, но т.к. включен другой вывод, необходимо временно отключить этот вывод и включить Elasticsearch с помощью параметра -E. Отключаем выходные данные Logstash.
```properties
sudo filebeat setup --index-management -E output.logstash.enabled=false -E 'output.elasticsearch.hosts=["139.59.247.231:9200"]' -E 'output.elasticsearch.username="dasha"' -E 'output.elasticsearch.password="123456"'
sudo filebeat setup --index-management -E output.logstash.enabled=false -E 'output.elasticsearch.hosts=["139.59.247.231:9200"]' -E 'output.elasticsearch.username="dasha"' -E 'output.elasticsearch.password="123456"' -E setup.kibana.host=139.59.247.231:5601
```
139.59.247.231 elk ip

### Запуск Filebeat
Запускаем Filebeat с помощью следующих команд:

```properties
sudo systemctl start filebeat
sudo systemctl enable filebeat
curl -u dasha:123456 -XGET 'http://139.59.247.231:9200/filebeat-*/_search?pretty'
```
![ElasticLogs](https://user-images.githubusercontent.com/91045595/166101316-8f53ed7b-950a-4b8f-9291-33a0f3301e93.jpg)


## GROK
Grok-это фильтр внутри Logstash, который используется для разбора неструктурированных данных на что-то структурированное и подлежащее запросу. Он находится поверх регулярного выражения (regex) и использует текстовые шаблоны для сопоставления строк в файлах журналов.

### Правило 1

Полученные логи

```json
{
  "fields" : {
    "message": [
        "May  1 10:01:37 proxy sshd[87897]: Failed password for root from 49.88.112.76 port 14398 ssh2"
      ]
  }
}
```

Grok
```
%{SYSLOGTIMESTAMP} %{WORD:server_name} %{WORD:service_name}\[%{NUMBER:pid}\]: Failed password for %{WORD:user_name} from %{IP:source_ip} port %{NUMBER:port}
```

Новые полученные логи

```json
{
  "fields": {
    "server_name": [
      "proxy"
    ],
    "user_name": [
      "root"
    ],
    "pid": [
      "87897"
    ],
    "source_ip": [
      "49.88.112.76"
    ],
    "service_name": [
      "sshd"
    ],
    "message": [
      "May  1 10:01:37 proxy sshd[87897]: Failed password for root from 49.88.112.76 port 14398 ssh2"
    ],
    "tags": [
      "beats_input_codec_plain_applied"
    ],
    "port": [
      "14398"
    ]
  }
}
```


### Правило 2

Полученные логи

```json
{
  "fields": {
    "message": [
      "Apr 27 02:13:09 proxy sshd[5803]: Received disconnect from 124.156.222.134 port 46278:11: Bye Bye [preauth]"
    ]
  }
}
```

Grok
```
%{SYSLOGTIMESTAMP} %{WORD:server_name} sshd\[%{NUMBER}\]: Received disconnect from %{IP:source_ip} port %{NUMBER:port}:%{DATA}: %{GREEDYDATA:msg}
```

Новые полученные логи

```json
{
  "fields": {
    "msg": [
      "Bye Bye [preauth]"
    ],
    "server_name": [
      "proxy"
    ],
    "source_ip": [
      "14.224.148.16"
    ],
    "message": [
      "May  1 10:25:28 proxy sshd[88097]: Received disconnect from 14.224.148.16 port 36408:11: Bye Bye [preauth]"
    ],
    "tags": [
      "beats_input_codec_plain_applied"
    ],
    "port": [
      "36408"
    ]
  }
}
```

### Правило 3

Полученные логи

```json
{
  "fields": {
    "message": [
      "May  1 10:31:28 proxy sshd[88184]: Disconnected from authenticating user root 165.22.63.216 port 33010 [preauth]"
    ]
  }
}
```

Grok
```
%{SYSLOGTIMESTAMP} %{WORD:server_name} sshd\[%{NUMBER}\]: %{WORD:status} from authenticating user %{WORD:user_name} %{IP:source_ip} port %{NUMBER:port}
```

Новые полученные логи

```json
{
  "fields": {
    "server_name": [
      "proxy"
    ],
    "user_name": [
      "root"
    ],
    "source_ip": [
      "124.194.74.203"
    ],
    "message": [
      "May  1 10:39:30 proxy sshd[88242]: Disconnected from authenticating user root 124.194.74.203 port 40918 [preauth]"
    ],
    "tags": [
      "beats_input_codec_plain_applied"
    ],
    "port": [
      "40918"
    ],
    "status": [
      "Disconnected"
    ]
  }
}
```
