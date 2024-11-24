# Cassandra Cluster Deployment with Docker Compose

## Описание

Этот проект развертывает кластер из трех инстансов Cassandra с использованием Docker Compose. Каждый инстанс доступен по отдельному IP-адресу в локальной сети.

## Требования
- ubuntu 22.04 lts 
На машине А:
- Docker
- Docker Compose
На машине B:
- cqlsh

## Установка

1. Установите Docker и Docker Compose на машине А:

   ```bash
   sudo apt update
   sudo apt install docker.io docker-compose -y

2. Создать на машине А файл docker-compose.yml, содержимое можно взять из репозитоиря.

   ```yml
   version: '3.7'

   services:
   cassandra1:
      image: cassandra:latest
      container_name: inst_cassandra1
      networks:
         cassandra_network:
         ipv4_address: 192.168.1.200
      volumes:
         - cassandra_data1:/var/lib/cassandra
      ports:
         - "9042:9042"

   cassandra2:
      image: cassandra:latest
      container_name: inst_cassandra2
      networks:
         cassandra_network:
         ipv4_address: 192.168.1.201
      volumes:
         - cassandra_data2:/var/lib/cassandra
      ports:
         - "9042:9042"

   cassandra3:
      image: cassandra:latest
      container_name: inst_cassandra3
      networks:
         cassandra_network:
         ipv4_address: 192.168.1.202
      volumes:
         - cassandra_data3:/var/lib/cassandra
      ports:
         - "9042:9042"

   networks:
   cassandra_network:
      driver: macvlan
      driver_opts:
         parent: eth1
      ipam:
         config:
         - subnet: "192.168.1.0/24"

   volumes:
   cassandra_data1:
   cassandra_data2:
   cassandra_data3:


3. Выполняем из рабочей директории, где находится файл docker-compose.yml.

   ```bash
   docker-compose up -d

   Проверяем работу контейнеров.

   ```bash
   docker ps

4. Произвести установку Cassandra на машине B и проверить подключение.

   Установка.

   ```bash
   apt update
   apt install cassandra

   Если будет ошибка об отсувствии пакета выполнить команды.

   ```bash
   echo "deb https://debian.cassandra.apache.org 40x main" > /etc/apt/sources.list.d/cassandra.list
   curl https://downloads.apache.org/cassandra/KEYS | sudo apt-key add -
   apt update
   apt install cassandra

   Проверка доступности контейнеров.
   ```bash
   cqlsh 192.168.1.200
   cqlsh 192.168.1.201
   cqlsh 192.168.1.202


## Примичания

   При недостатке оперативной памяти потребуется создать swap.

   Для доступа к контейнерам на хостовой машине создать виртуальный сетевонй интерфейс, который будет работать как мост.
   ```bash
   ip link add macvlan0 link eth1 type macvlan mode bridge
   ip addr add 192.168.1.197/24 dev macvlan0
   ip link set macvlan0 up

