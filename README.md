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

В найденном месте необходимо ниже добавить строчки и поле сохранить измененный файл:
 ```properties
 auth_param basic program /usr/lib/squid3/basic_ncsa_auth /etc/squid/passwords`
 auth_param basic realm proxy`
 acl authenticated proxy_auth REQUIRED`
 acl localnet src *ip_adress* 
 http_access allow authenticated
 ```
 *ip_adress* - ip адрес локальной машины.
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
Для начала необходимо использовать cURL, инструмент командной строки для передачи данных с помощью URL, для импорта открытого ключа Elasticsearch GPG в APT. Так же используем аргументы -fsSL для подавления всех текущих и возможных ошибок (кроме сбоя сервера), а также, чтобы разрешить cURL подать запрос на другой локации при переадресации. Выведите результаты команды cURL в программу apt-key, которая добавит открытый ключ GPG в APT

```properties
curl -fsSL https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
```

Затем добавьте список источников Elastic в каталог sources.list.d, где APT будет искать новые источники:

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
Для ограничения доступа и повышения безопасности находим строку с указанием network.host и убераем с нее значок комментария, после чего заменяем значение на 0.0.0.0, чтобы она выглядела следующим образом: "network.host: 0.0.0.0"
Elasticsearch формирует одноузловой кластер: discovery.type: single-node
Включаем функции безопасности Elasticsearch на узле: xpack.security.enabled: true

### Запуск Elasticsearch
Запукаем Elasticsearch с помощью следующих команд:
```properties
sudo systemctl start elasticsearch
sudo systemctl enable elasticsearch
```
### Создание пользователей

```properties
sudo -u root /usr/share/elasticsearch/bin/elasticsearch-setup-passwords auto
```
Сохраняем вывод

![elastic](https://user-images.githubusercontent.com/91045595/166099919-21b65f88-cd66-4dcc-9669-dd402a8be0a1.jpg)


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

### Добавление пароля

```properties
sudo -u root /usr/share/kibana/bin/kibana-keystore create
sudo -u root /usr/share/kibana/bin/kibana-keystore add elasticsearch.password
```

Вписываем пароль, который получили на создания пользователей
```properties
Changed password for user kibana_system
PASSWORD kibana_system = OZ7w09XTeHBg3rfihAsm
```


## Logstash

### Установка Logstash
Установим Logstash- сервис для сбора логов и отправки их в Elasticsearch.
```properties
sudo apt install logstash
```

### Настройка Logstash
Откроем файл конфигураций:
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
    hosts => ["_elk_ip_:9200"]
    manage_template => false 
    index => "%{[@metadata][beat]}-%{[@metadata[version]}-%{+YYYY.MM.dd}"
    user => "_username_"
    password => "_password_"
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

Раскомментируем строчи "output.logstash" и "hosts: ["localhost:5044"]"

Изменим localhost на ip нашего сервера с ELK
![elasticOutput](https://user-images.githubusercontent.com/91045595/166100621-15ed5fd9-3402-4d3e-a141-859b971a96cd.jpg)
![LogstashOutput](https://user-images.githubusercontent.com/91045595/166100623-b61d5d44-15e1-4d57-8c75-1a994f44d3a5.jpg)


Проверим конфигурацию "sudo filebeat -e test output"
Так как мы отправляем события в Logstash, то необходимо вручную загрузить конвейеры загрузки. Для этого запустим команду:
```properties
sudo filebeat setup --pipelines --modules system
```
Установим index templates для Elasticsearch вручную.

Чтобы загрузить шаблон индекса вручную, запустим команду установки. Требуется подключение к Elasticsearch, но т.к. включен другой вывод, необходимо временно отключить этот вывод и включить Elasticsearch с помощью параметра -E. Отключаем выходные данные Logstash.
```properties
sudo filebeat setup --index-management -E output.logstash.enabled=false -E 'output.elasticsearch.hosts=["139.59.247.231:9200"]' -E 'output.elasticsearch.username="elastic"' -E 'output.elasticsearch.password="nrRO28jYIplnaEP3JBou"'
sudo filebeat setup --index-management -E output.logstash.enabled=false -E 'output.elasticsearch.hosts=["139.59.247.231:9200"]' -E 'output.elasticsearch.username="elastic"' -E 'output.elasticsearch.password="nrRO28jYIplnaEP3JBou"' -E setup.kibana.host=__ELK_IP__:5601
```
139.59.247.231 elk ip
elastic password- nrRO28jYIplnaEP3JBou

### Запуск Filebeat
Запускаем Filebeat с помощью следующих команд:

```properties
sudo systemctl start filebeat
sudo systemctl enable filebeat
curl -u _username_:_password -XGET 'http://139.59.247.231:9200/filebeat-*/_search?pretty'
```
![ElasticLogs](https://user-images.githubusercontent.com/91045595/166101316-8f53ed7b-950a-4b8f-9291-33a0f3301e93.jpg)


## GROK
Grok-это фильтр внутри Logstash, который используется для разбора неструктурированных данных на что-то структурированное и подлежащее запросу. Он находится поверх регулярного выражения (regex) и использует текстовые шаблоны для сопоставления строк в файлах журналов.

### Правило 1

Полученные логи

```json
{
  "_index": "filebeat-7.17.2-2022.04.26",
  "_type": "_doc",
  "_id": "9GE9ZIABAwPEDzUuLjxD",
  "_version": 41,
  "_score": 1,
  "_source": {
    "ecs": {
      "version": "1.12.0"
    },
    "fileset": {
      "name": "syslog"
    },
    "cloud": {
      "region": "fra1",
      "service": {
        "name": "Droplets"
      },
      "instance": {
        "id": "294566205"
      },
      "provider": "digitalocean"
    },
    "service": {
      "type": "system"
    },
    "log": {
      "file": {
        "path": "/var/log/syslog"
      },
      "offset": 1196759
    },
    "agent": {
      "id": "03d66c8a-f7bd-4e32-aee6-3df522db971d",
      "version": "7.17.2",
      "name": "proxySquid",
      "hostname": "proxySquid",
      "type": "filebeat",
      "ephemeral_id": "83f329a2-d14b-4a16-bc97-b34aff46e89b"
    },
    "event": {
      "dataset": "system.syslog",
      "module": "system",
      "timezone": "+00:00"
    },
    "@timestamp": "2022-04-26T04:59:26.695Z",
    "host": {
      "os": {
        "version": "20.04.3 LTS (Focal Fossa)",
        "platform": "ubuntu",
        "name": "Ubuntu",
        "kernel": "5.4.0-97-generic",
        "codename": "focal",
        "type": "linux",
        "family": "debian"
      },
      "id": "f39cad9bf667888807d67e1862511af0",
      "mac": [
        "e2:41:66:65:3d:cc",
        "8e:8c:93:42:9a:65"
      ],
      "name": "proxySquid",
      "ip": [
        "207.154.193.75",
        "10.19.0.5",
        "fe80::e041:66ff:fe65:3dcc",
        "10.114.0.2",
        "fe80::8c8c:93ff:fe42:9a65"
      ],
      "hostname": "proxySquid",
      "architecture": "x86_64",
      "containerized": false
    },
    "@version": "1",
    "tags": [
      "beats_input_codec_plain_applied"
    ],
    "message": "Apr 26 04:59:26 proxySquid kernel: [1466719.889974] [UFW BLOCK] IN=eth0 OUT= MAC=e2:41:66:65:3d:cc:fe:00:00:00:01:01:08:00 SRC=73.225.108.52 DST=207.154.193.75 LEN=232 TOS=0x00 PREC=0x00 TTL=47 ID=59840 PROTO=UDP SPT=59281 DPT=51820 LEN=212 ",
    "input": {
      "type": "log"
    }
  },
  "fields": {
    "host.os.name.text": [
      "Ubuntu"
    ],
    "host.hostname": [
      "proxySquid"
    ],
    "host.mac": [
      "e2:41:66:65:3d:cc",
      "8e:8c:93:42:9a:65"
    ],
    "service.type": [
      "system"
    ],
    "host.ip": [
      "207.154.193.75",
      "10.19.0.5",
      "fe80::e041:66ff:fe65:3dcc",
      "10.114.0.2",
      "fe80::8c8c:93ff:fe42:9a65"
    ],
    "cloud.instance.id": [
      "294566205"
    ],
    "agent.type": [
      "filebeat"
    ],
    "event.module": [
      "system"
    ],
    "host.os.version": [
      "20.04.3 LTS (Focal Fossa)"
    ],
    "host.os.kernel": [
      "5.4.0-97-generic"
    ],
    "@version": [
      "1"
    ],
    "host.os.name": [
      "Ubuntu"
    ],
    "agent.name": [
      "proxySquid"
    ],
    "host.name": [
      "proxySquid"
    ],
    "host.id": [
      "f39cad9bf667888807d67e1862511af0"
    ],
    "event.timezone": [
      "+00:00"
    ],
    "host.os.type": [
      "linux"
    ],
    "cloud.region": [
      "fra1"
    ],
    "fileset.name": [
      "syslog"
    ],
    "host.os.codename": [
      "focal"
    ],
    "input.type": [
      "log"
    ],
    "log.offset": [
      1196759
    ],
    "agent.hostname": [
      "proxySquid"
    ],
    "message": [
      "Apr 26 04:59:26 proxySquid kernel: [1466719.889974] [UFW BLOCK] IN=eth0 OUT= MAC=e2:41:66:65:3d:cc:fe:00:00:00:01:01:08:00 SRC=73.225.108.52 DST=207.154.193.75 LEN=232 TOS=0x00 PREC=0x00 TTL=47 ID=59840 PROTO=UDP SPT=59281 DPT=51820 LEN=212 "
    ],
    "tags": [
      "beats_input_codec_plain_applied"
    ],
    "host.architecture": [
      "x86_64"
    ],
    "cloud.provider": [
      "digitalocean"
    ],
    "@timestamp": [
      "2022-04-26T04:59:26.695Z"
    ],
    "agent.id": [
      "03d66c8a-f7bd-4e32-aee6-3df522db971d"
    ],
    "cloud.service.name": [
      "Droplets"
    ],
    "ecs.version": [
      "1.12.0"
    ],
    "host.containerized": [
      false
    ],
    "host.os.platform": [
      "ubuntu"
    ],
    "log.file.path": [
      "/var/log/syslog"
    ],
    "agent.ephemeral_id": [
      "83f329a2-d14b-4a16-bc97-b34aff46e89b"
    ],
    "agent.version": [
      "7.17.2"
    ],
    "host.os.family": [
      "debian"
    ],
    "event.dataset": [
      "system.syslog"
    ]
  }
}
```

Grok
```
%{SYSLOGTIMESTAMP:time} %{WORD:server_name} %{WORD:service}: \[%{DATA}\] \[UFW BLOCK\] IN=%{WORD:in} OUT=%{GREEDYDATA:out} MAC=%{GREEDYDATA:mac} SRC=%{IP:src_ip} DST=%{IP:dst_ip}
```

New Data

```json
{
  "_index": "filebeat-7.17.2-2022.04.27",
  "_type": "_doc",
  "_id": "KWFKaYABAwPEDzUuUWtZ",
  "_version": 1,
  "_score": 1,
  "_source": {
    "log": {
      "offset": 899071,
      "file": {
        "path": "/var/log/syslog"
      }
    },
    "event": {
      "dataset": "system.syslog",
      "timezone": "+00:00",
      "module": "system"
    },
    "host": {
      "id": "f39cad9bf667888807d67e1862511af0",
      "architecture": "x86_64",
      "ip": [
        "207.154.193.75",
        "10.19.0.5",
        "fe80::e041:66ff:fe65:3dcc",
        "10.114.0.2",
        "fe80::8c8c:93ff:fe42:9a65"
      ],
      "name": "proxySquid",
      "mac": [
        "e2:41:66:65:3d:cc",
        "8e:8c:93:42:9a:65"
      ],
      "hostname": "proxySquid",
      "containerized": false,
      "os": {
        "family": "debian",
        "version": "20.04.3 LTS (Focal Fossa)",
        "kernel": "5.4.0-97-generic",
        "name": "Ubuntu",
        "codename": "focal",
        "platform": "ubuntu",
        "type": "linux"
      }
    },
    "server_name": "proxySquid",
    "src_ip": "91.240.118.246",
    "@timestamp": "2022-04-27T04:31:58.713Z",
    "@version": "1",
    "input": {
      "type": "log"
    },
    "cloud": {
      "service": {
        "name": "Droplets"
      },
      "instance": {
        "id": "294566205"
      },
      "provider": "digitalocean",
      "region": "fra1"
    },
    "dst_ip": "207.154.193.75",
    "agent": {
      "id": "03d66c8a-f7bd-4e32-aee6-3df522db971d",
      "version": "7.17.2",
      "ephemeral_id": "83f329a2-d14b-4a16-bc97-b34aff46e89b",
      "name": "proxySquid",
      "hostname": "proxySquid",
      "type": "filebeat"
    },
    "service": {
      "type": "system"
    },
    "ecs": {
      "version": "1.12.0"
    },
    "fileset": {
      "name": "syslog"
    },
    "in": "eth0",
    "mac": "e2:41:66:65:3d:cc:fe:00:00:00:01:01:08:00",
    "tags": [
      "beats_input_codec_plain_applied"
    ]
  },
  "fields": {
    "server_name": [
      "proxySquid"
    ],
    "host.os.name.text": [
      "Ubuntu"
    ],
    "host.hostname": [
      "proxySquid"
    ],
    "host.mac": [
      "e2:41:66:65:3d:cc",
      "8e:8c:93:42:9a:65"
    ],
    "mac": [
      "e2:41:66:65:3d:cc:fe:00:00:00:01:01:08:00"
    ],
    "dst_ip": [
      "207.154.193.75"
    ],
    "src_ip": [
      "91.240.118.246"
    ],
    "service.type": [
      "system"
    ],
    "host.ip": [
      "207.154.193.75",
      "10.19.0.5",
      "fe80::e041:66ff:fe65:3dcc",
      "10.114.0.2",
      "fe80::8c8c:93ff:fe42:9a65"
    ],
    "cloud.instance.id": [
      "294566205"
    ],
    "agent.type": [
      "filebeat"
    ],
    "event.module": [
      "system"
    ],
    "host.os.version": [
      "20.04.3 LTS (Focal Fossa)"
    ],
    "host.os.kernel": [
      "5.4.0-97-generic"
    ],
    "@version": [
      "1"
    ],
    "host.os.name": [
      "Ubuntu"
    ],
    "agent.name": [
      "proxySquid"
    ],
    "host.name": [
      "proxySquid"
    ],
    "host.id": [
      "f39cad9bf667888807d67e1862511af0"
    ],
    "event.timezone": [
      "+00:00"
    ],
    "host.os.type": [
      "linux"
    ],
    "cloud.region": [
      "fra1"
    ],
    "in": [
      "eth0"
    ],
    "fileset.name": [
      "syslog"
    ],
    "host.os.codename": [
      "focal"
    ],
    "input.type": [
      "log"
    ],
    "log.offset": [
      899071
    ],
    "agent.hostname": [
      "proxySquid"
    ],
    "tags": [
      "beats_input_codec_plain_applied"
    ],
    "host.architecture": [
      "x86_64"
    ],
    "cloud.provider": [
      "digitalocean"
    ],
    "@timestamp": [
      "2022-04-27T04:31:58.713Z"
    ],
    "agent.id": [
      "03d66c8a-f7bd-4e32-aee6-3df522db971d"
    ],
    "cloud.service.name": [
      "Droplets"
    ],
    "ecs.version": [
      "1.12.0"
    ],
    "host.containerized": [
      false
    ],
    "host.os.platform": [
      "ubuntu"
    ],
    "log.file.path": [
      "/var/log/syslog"
    ],
    "agent.ephemeral_id": [
      "83f329a2-d14b-4a16-bc97-b34aff46e89b"
    ],
    "agent.version": [
      "7.17.2"
    ],
    "host.os.family": [
      "debian"
    ],
    "event.dataset": [
      "system.syslog"
    ]
  }
}
```


### Правило 2

Data

```json
{
  "_index": "filebeat-7.17.2-2022.04.26",
  "_type": "_doc",
  "_id": "-mHZZIABAwPEDzUua0Gc",
  "_version": 20,
  "_score": 1,
  "_source": {
    "service": {
      "type": "system"
    },
    "input": {
      "type": "log"
    },
    "host": {
      "hostname": "proxySquid",
      "architecture": "x86_64",
      "mac": [
        "e2:41:66:65:3d:cc",
        "8e:8c:93:42:9a:65"
      ],
      "id": "f39cad9bf667888807d67e1862511af0",
      "name": "proxySquid",
      "containerized": false,
      "os": {
        "version": "20.04.3 LTS (Focal Fossa)",
        "platform": "ubuntu",
        "family": "debian",
        "name": "Ubuntu",
        "kernel": "5.4.0-97-generic",
        "type": "linux",
        "codename": "focal"
      },
      "ip": [
        "207.154.193.75",
        "10.19.0.5",
        "fe80::e041:66ff:fe65:3dcc",
        "10.114.0.2",
        "fe80::8c8c:93ff:fe42:9a65"
      ]
    },
    "agent": {
      "hostname": "proxySquid",
      "version": "7.17.2",
      "id": "03d66c8a-f7bd-4e32-aee6-3df522db971d",
      "ephemeral_id": "83f329a2-d14b-4a16-bc97-b34aff46e89b",
      "name": "proxySquid",
      "type": "filebeat"
    },
    "@version": "1",
    "time": "Apr 26 07:49:56",
    "fileset": {
      "name": "auth"
    },
    "tags": [
      "beats_input_codec_plain_applied"
    ],
    "log": {
      "file": {
        "path": "/var/log/auth.log"
      },
      "offset": 470666
    },
    "event": {
      "dataset": "system.auth",
      "module": "system",
      "timezone": "+00:00"
    },
    "@timestamp": "2022-04-26T07:50:06.370Z",
    "message": "Apr 26 07:49:56 proxySquid sshd[127302]: Disconnected from authenticating user root 61.177.173.36 port 57131 [preauth]",
    "ecs": {
      "version": "1.12.0"
    },
    "cloud": {
      "provider": "digitalocean",
      "service": {
        "name": "Droplets"
      },
      "instance": {
        "id": "294566205"
      },
      "region": "fra1"
    }
  },
  "fields": {
    "host.os.name.text": [
      "Ubuntu"
    ],
    "host.hostname": [
      "proxySquid"
    ],
    "host.mac": [
      "e2:41:66:65:3d:cc",
      "8e:8c:93:42:9a:65"
    ],
    "service.type": [
      "system"
    ],
    "host.ip": [
      "207.154.193.75",
      "10.19.0.5",
      "fe80::e041:66ff:fe65:3dcc",
      "10.114.0.2",
      "fe80::8c8c:93ff:fe42:9a65"
    ],
    "cloud.instance.id": [
      "294566205"
    ],
    "agent.type": [
      "filebeat"
    ],
    "event.module": [
      "system"
    ],
    "host.os.version": [
      "20.04.3 LTS (Focal Fossa)"
    ],
    "host.os.kernel": [
      "5.4.0-97-generic"
    ],
    "@version": [
      "1"
    ],
    "host.os.name": [
      "Ubuntu"
    ],
    "agent.name": [
      "proxySquid"
    ],
    "host.name": [
      "proxySquid"
    ],
    "host.id": [
      "f39cad9bf667888807d67e1862511af0"
    ],
    "event.timezone": [
      "+00:00"
    ],
    "host.os.type": [
      "linux"
    ],
    "cloud.region": [
      "fra1"
    ],
    "fileset.name": [
      "auth"
    ],
    "host.os.codename": [
      "focal"
    ],
    "input.type": [
      "log"
    ],
    "log.offset": [
      470666
    ],
    "agent.hostname": [
      "proxySquid"
    ],
    "message": [
      "Apr 26 07:49:56 proxySquid sshd[127302]: Disconnected from authenticating user root 61.177.173.36 port 57131 [preauth]"
    ],
    "tags": [
      "beats_input_codec_plain_applied"
    ],
    "host.architecture": [
      "x86_64"
    ],
    "cloud.provider": [
      "digitalocean"
    ],
    "@timestamp": [
      "2022-04-26T07:50:06.370Z"
    ],
    "agent.id": [
      "03d66c8a-f7bd-4e32-aee6-3df522db971d"
    ],
    "cloud.service.name": [
      "Droplets"
    ],
    "ecs.version": [
      "1.12.0"
    ],
    "host.containerized": [
      false
    ],
    "host.os.platform": [
      "ubuntu"
    ],
    "log.file.path": [
      "/var/log/auth.log"
    ],
    "agent.ephemeral_id": [
      "83f329a2-d14b-4a16-bc97-b34aff46e89b"
    ],
    "agent.version": [
      "7.17.2"
    ],
    "time": [
      "Apr 26 07:49:56"
    ],
    "host.os.family": [
      "debian"
    ],
    "event.dataset": [
      "system.auth"
    ]
  }
}
```

Grok

```
%{SYSLOGTIMESTAMP:time} %{WORD:server_name} %{WORD:service_name}\[%{NUMBER}\]: %{DATA:msg} %{IP:ip} port %{NUMBER:port}
```

New Data

```json
{
  "_index": "filebeat-7.17.2-2022.04.26",
  "_type": "_doc",
  "_id": "tGHAZYABAwPEDzUuLEmu",
  "_version": 1,
  "_score": 1,
  "_source": {
    "cloud": {
      "service": {
        "name": "Droplets"
      },
      "instance": {
        "id": "294566205"
      },
      "provider": "digitalocean",
      "region": "fra1"
    },
    "agent": {
      "hostname": "proxySquid",
      "version": "7.17.2",
      "id": "03d66c8a-f7bd-4e32-aee6-3df522db971d",
      "name": "proxySquid",
      "type": "filebeat",
      "ephemeral_id": "83f329a2-d14b-4a16-bc97-b34aff46e89b"
    },
    "msg": "Disconnected from authenticating user root",
    "tags": [
      "beats_input_codec_plain_applied"
    ],
    "server_name": "proxySquid",
    "event": {
      "dataset": "system.auth",
      "timezone": "+00:00",
      "module": "system"
    },
    "service_name": "sshd",
    "port": "45236",
    "service": {
      "type": "system"
    },
    "log": {
      "file": {
        "path": "/var/log/auth.log"
      },
      "offset": 501687
    },
    "time": "Apr 26 12:01:58",
    "ip": "212.20.41.28",
    "@timestamp": "2022-04-26T12:02:08.722Z",
    "@version": "1",
    "fileset": {
      "name": "auth"
    },
    "input": {
      "type": "log"
    },
    "host": {
      "hostname": "proxySquid",
      "mac": [
        "e2:41:66:65:3d:cc",
        "8e:8c:93:42:9a:65"
      ],
      "id": "f39cad9bf667888807d67e1862511af0",
      "ip": [
        "207.154.193.75",
        "10.19.0.5",
        "fe80::e041:66ff:fe65:3dcc",
        "10.114.0.2",
        "fe80::8c8c:93ff:fe42:9a65"
      ],
      "architecture": "x86_64",
      "name": "proxySquid",
      "containerized": false,
      "os": {
        "version": "20.04.3 LTS (Focal Fossa)",
        "family": "debian",
        "name": "Ubuntu",
        "type": "linux",
        "kernel": "5.4.0-97-generic",
        "codename": "focal",
        "platform": "ubuntu"
      }
    },
    "ecs": {
      "version": "1.12.0"
    }
  },
  "fields": {
    "msg": [
      "Disconnected from authenticating user root"
    ],
    "server_name": [
      "proxySquid"
    ],
    "host.os.name.text": [
      "Ubuntu"
    ],
    "host.hostname": [
      "proxySquid"
    ],
    "host.mac": [
      "e2:41:66:65:3d:cc",
      "8e:8c:93:42:9a:65"
    ],
    "service.type": [
      "system"
    ],
    "host.ip": [
      "207.154.193.75",
      "10.19.0.5",
      "fe80::e041:66ff:fe65:3dcc",
      "10.114.0.2",
      "fe80::8c8c:93ff:fe42:9a65"
    ],
    "cloud.instance.id": [
      "294566205"
    ],
    "agent.type": [
      "filebeat"
    ],
    "event.module": [
      "system"
    ],
    "host.os.version": [
      "20.04.3 LTS (Focal Fossa)"
    ],
    "host.os.kernel": [
      "5.4.0-97-generic"
    ],
    "@version": [
      "1"
    ],
    "host.os.name": [
      "Ubuntu"
    ],
    "agent.name": [
      "proxySquid"
    ],
    "host.name": [
      "proxySquid"
    ],
    "host.id": [
      "f39cad9bf667888807d67e1862511af0"
    ],
    "event.timezone": [
      "+00:00"
    ],
    "host.os.type": [
      "linux"
    ],
    "cloud.region": [
      "fra1"
    ],
    "service_name": [
      "sshd"
    ],
    "ip": [
      "212.20.41.28"
    ],
    "fileset.name": [
      "auth"
    ],
    "host.os.codename": [
      "focal"
    ],
    "input.type": [
      "log"
    ],
    "log.offset": [
      501687
    ],
    "agent.hostname": [
      "proxySquid"
    ],
    "tags": [
      "beats_input_codec_plain_applied"
    ],
    "host.architecture": [
      "x86_64"
    ],
    "cloud.provider": [
      "digitalocean"
    ],
    "@timestamp": [
      "2022-04-26T12:02:08.722Z"
    ],
    "agent.id": [
      "03d66c8a-f7bd-4e32-aee6-3df522db971d"
    ],
    "cloud.service.name": [
      "Droplets"
    ],
    "port": [
      "45236"
    ],
    "ecs.version": [
      "1.12.0"
    ],
    "host.containerized": [
      false
    ],
    "host.os.platform": [
      "ubuntu"
    ],
    "log.file.path": [
      "/var/log/auth.log"
    ],
    "agent.ephemeral_id": [
      "83f329a2-d14b-4a16-bc97-b34aff46e89b"
    ],
    "agent.version": [
      "7.17.2"
    ],
    "time": [
      "Apr 26 12:01:58"
    ],
    "host.os.family": [
      "debian"
    ],
    "event.dataset": [
      "system.auth"
    ]
  }
}
```

### Правило 3

Data

```json
{
  "_index": "filebeat-7.17.2-2022.04.27",
  "_type": "_doc",
  "_id": "0mFiaYABAwPEDzUuDmu5",
  "_version": 1,
  "_score": 1,
  "_source": {
    "log": {
      "offset": 932761,
      "file": {
        "path": "/var/log/auth.log"
      }
    },
    "event": {
      "dataset": "system.auth",
      "timezone": "+00:00",
      "module": "system"
    },
    "host": {
      "id": "f39cad9bf667888807d67e1862511af0",
      "architecture": "x86_64",
      "ip": [
        "207.154.193.75",
        "10.19.0.5",
        "fe80::e041:66ff:fe65:3dcc",
        "10.114.0.2",
        "fe80::8c8c:93ff:fe42:9a65"
      ],
      "name": "proxySquid",
      "mac": [
        "e2:41:66:65:3d:cc",
        "8e:8c:93:42:9a:65"
      ],
      "hostname": "proxySquid",
      "os": {
        "family": "debian",
        "version": "20.04.3 LTS (Focal Fossa)",
        "kernel": "5.4.0-97-generic",
        "name": "Ubuntu",
        "codename": "focal",
        "platform": "ubuntu",
        "type": "linux"
      },
      "containerized": false
    },
    "@timestamp": "2022-04-27T04:57:49.532Z",
    "@version": "1",
    "cloud": {
      "service": {
        "name": "Droplets"
      },
      "instance": {
        "id": "294566205"
      },
      "provider": "digitalocean",
      "region": "fra1"
    },
    "input": {
      "type": "log"
    },
    "agent": {
      "id": "03d66c8a-f7bd-4e32-aee6-3df522db971d",
      "version": "7.17.2",
      "ephemeral_id": "83f329a2-d14b-4a16-bc97-b34aff46e89b",
      "name": "proxySquid",
      "hostname": "proxySquid",
      "type": "filebeat"
    },
    "service": {
      "type": "system"
    },
    "ecs": {
      "version": "1.12.0"
    },
    "fileset": {
      "name": "auth"
    },
    "message": "Apr 27 04:57:48 proxySquid su: FAILED SU (to root) brave on pts/1",
    "tags": [
      "beats_input_codec_plain_applied",
      "_grokparsefailure"
    ]
  },
  "fields": {
    "host.os.name.text": [
      "Ubuntu"
    ],
    "host.hostname": [
      "proxySquid"
    ],
    "host.mac": [
      "e2:41:66:65:3d:cc",
      "8e:8c:93:42:9a:65"
    ],
    "service.type": [
      "system"
    ],
    "host.ip": [
      "207.154.193.75",
      "10.19.0.5",
      "fe80::e041:66ff:fe65:3dcc",
      "10.114.0.2",
      "fe80::8c8c:93ff:fe42:9a65"
    ],
    "cloud.instance.id": [
      "294566205"
    ],
    "agent.type": [
      "filebeat"
    ],
    "event.module": [
      "system"
    ],
    "host.os.version": [
      "20.04.3 LTS (Focal Fossa)"
    ],
    "host.os.kernel": [
      "5.4.0-97-generic"
    ],
    "@version": [
      "1"
    ],
    "host.os.name": [
      "Ubuntu"
    ],
    "agent.name": [
      "proxySquid"
    ],
    "host.name": [
      "proxySquid"
    ],
    "host.id": [
      "f39cad9bf667888807d67e1862511af0"
    ],
    "event.timezone": [
      "+00:00"
    ],
    "host.os.type": [
      "linux"
    ],
    "cloud.region": [
      "fra1"
    ],
    "fileset.name": [
      "auth"
    ],
    "host.os.codename": [
      "focal"
    ],
    "input.type": [
      "log"
    ],
    "log.offset": [
      932761
    ],
    "agent.hostname": [
      "proxySquid"
    ],
    "message": [
      "Apr 27 04:57:48 proxySquid su: FAILED SU (to root) brave on pts/1"
    ],
    "tags": [
      "beats_input_codec_plain_applied",
      "_grokparsefailure"
    ],
    "host.architecture": [
      "x86_64"
    ],
    "cloud.provider": [
      "digitalocean"
    ],
    "@timestamp": [
      "2022-04-27T04:57:49.532Z"
    ],
    "agent.id": [
      "03d66c8a-f7bd-4e32-aee6-3df522db971d"
    ],
    "cloud.service.name": [
      "Droplets"
    ],
    "ecs.version": [
      "1.12.0"
    ],
    "host.containerized": [
      false
    ],
    "host.os.platform": [
      "ubuntu"
    ],
    "log.file.path": [
      "/var/log/auth.log"
    ],
    "agent.ephemeral_id": [
      "83f329a2-d14b-4a16-bc97-b34aff46e89b"
    ],
    "agent.version": [
      "7.17.2"
    ],
    "host.os.family": [
      "debian"
    ],
    "event.dataset": [
      "system.auth"
    ]
  }
}
```

Grok

```
%{SYSLOGTIMESTAMP} %{WORD:server_name} %{WORD:command}: %{WORD:status} %{WORD} \(%{GREEDYDATA:desc}\) %{USERNAME:username}
```

New Data

```json
{
  "_index": "filebeat-7.17.2-2022.04.27",
  "_type": "_doc",
  "_id": "rGF9aYABAwPEDzUusWyI",
  "_version": 1,
  "_score": 1,
  "_source": {
    "@timestamp": "2022-04-27T05:28:00.678Z",
    "fileset": {
      "name": "auth"
    },
    "server_name": "proxySquid",
    "ecs": {
      "version": "1.12.0"
    },
    "@version": "1",
    "cloud": {
      "instance": {
        "id": "294566205"
      },
      "region": "fra1",
      "provider": "digitalocean",
      "service": {
        "name": "Droplets"
      }
    },
    "desc": "to root",
    "agent": {
      "id": "03d66c8a-f7bd-4e32-aee6-3df522db971d",
      "version": "7.17.2",
      "hostname": "proxySquid",
      "ephemeral_id": "83f329a2-d14b-4a16-bc97-b34aff46e89b",
      "name": "proxySquid",
      "type": "filebeat"
    },
    "log": {
      "file": {
        "path": "/var/log/auth.log"
      },
      "offset": 935824
    },
    "input": {
      "type": "log"
    },
    "service": {
      "type": "system"
    },
    "tags": [
      "beats_input_codec_plain_applied"
    ],
    "command": "su",
    "username": "brave",
    "event": {
      "dataset": "system.auth",
      "module": "system",
      "timezone": "+00:00"
    },
    "host": {
      "os": {
        "version": "20.04.3 LTS (Focal Fossa)",
        "platform": "ubuntu",
        "family": "debian",
        "codename": "focal",
        "name": "Ubuntu",
        "type": "linux",
        "kernel": "5.4.0-97-generic"
      },
      "mac": [
        "e2:41:66:65:3d:cc",
        "8e:8c:93:42:9a:65"
      ],
      "ip": [
        "207.154.193.75",
        "10.19.0.5",
        "fe80::e041:66ff:fe65:3dcc",
        "10.114.0.2",
        "fe80::8c8c:93ff:fe42:9a65"
      ],
      "id": "f39cad9bf667888807d67e1862511af0",
      "hostname": "proxySquid",
      "containerized": false,
      "architecture": "x86_64",
      "name": "proxySquid"
    },
    "status": "FAILED"
  },
  "fields": {
    "server_name": [
      "proxySquid"
    ],
    "host.os.name.text": [
      "Ubuntu"
    ],
    "host.hostname": [
      "proxySquid"
    ],
    "host.mac": [
      "e2:41:66:65:3d:cc",
      "8e:8c:93:42:9a:65"
    ],
    "service.type": [
      "system"
    ],
    "host.ip": [
      "207.154.193.75",
      "10.19.0.5",
      "fe80::e041:66ff:fe65:3dcc",
      "10.114.0.2",
      "fe80::8c8c:93ff:fe42:9a65"
    ],
    "cloud.instance.id": [
      "294566205"
    ],
    "agent.type": [
      "filebeat"
    ],
    "event.module": [
      "system"
    ],
    "host.os.version": [
      "20.04.3 LTS (Focal Fossa)"
    ],
    "host.os.kernel": [
      "5.4.0-97-generic"
    ],
    "@version": [
      "1"
    ],
    "host.os.name": [
      "Ubuntu"
    ],
    "agent.name": [
      "proxySquid"
    ],
    "host.name": [
      "proxySquid"
    ],
    "host.id": [
      "f39cad9bf667888807d67e1862511af0"
    ],
    "event.timezone": [
      "+00:00"
    ],
    "host.os.type": [
      "linux"
    ],
    "cloud.region": [
      "fra1"
    ],
    "fileset.name": [
      "auth"
    ],
    "host.os.codename": [
      "focal"
    ],
    "input.type": [
      "log"
    ],
    "log.offset": [
      935824
    ],
    "agent.hostname": [
      "proxySquid"
    ],
    "command": [
      "su"
    ],
    "tags": [
      "beats_input_codec_plain_applied"
    ],
    "host.architecture": [
      "x86_64"
    ],
    "cloud.provider": [
      "digitalocean"
    ],
    "@timestamp": [
      "2022-04-27T05:28:00.678Z"
    ],
    "agent.id": [
      "03d66c8a-f7bd-4e32-aee6-3df522db971d"
    ],
    "cloud.service.name": [
      "Droplets"
    ],
    "ecs.version": [
      "1.12.0"
    ],
    "host.containerized": [
      false
    ],
    "host.os.platform": [
      "ubuntu"
    ],
    "log.file.path": [
      "/var/log/auth.log"
    ],
    "agent.ephemeral_id": [
      "83f329a2-d14b-4a16-bc97-b34aff46e89b"
    ],
    "agent.version": [
      "7.17.2"
    ],
    "host.os.family": [
      "debian"
    ],
    "event.dataset": [
      "system.auth"
    ],
    "status": [
      "FAILED"
    ],
    "username": [
      "brave"
    ],
    "desc": [
      "to root"
    ]
  }
}
```
