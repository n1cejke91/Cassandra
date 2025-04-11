#   Развертывание кластера Cassandra с использованием Docker Compose
Этот проект демонстрирует, как развернуть кластер Cassandra из 3 узлов с использованием Docker Compose. Он также включает инструкции по созданию keyspace и таблиц, вставке данных, выполнению запросов и созданию вторичного индекса.

##   Начало работы

1.  **Настройка Docker Compose** :

```sh
nano docker-compose.yml
```
```yaml
services:
  cassandra-seed:
    image: cassandra:latest
    volumes:
      - "cassandra_data_seed:/var/lib/cassandra"
    environment:
      - "CASSANDRA_SEEDS=cassandra-seed"
      - "CASSANDRA_CLUSTER_NAME=Test Cluster"
      - "CASSANDRA_DC=se1"
      - "CASSANDRA_ENDPOINT_SNITCH=GossipingPropertyFileSnitch"

  cassandra1:
    image: cassandra:latest
    volumes:
      - "cassandra_data_1:/var/lib/cassandra"
    environment:
      - "CASSANDRA_SEEDS=cassandra-seed"
      - "CASSANDRA_CLUSTER_NAME=Test Cluster"
      - "CASSANDRA_DC=se1"
      - "CASSANDRA_ENDPOINT_SNITCH=GossipingPropertyFileSnitch"
    depends_on:
      - cassandra-seed

  cassandra2:
    image: cassandra:latest
    volumes:
      - "cassandra_data_2:/var/lib/cassandra"
    environment:
      - "CASSANDRA_SEEDS=cassandra-seed"
      - "CASSANDRA_CLUSTER_NAME=Test Cluster"
      - "CASSANDRA_DC=se1"
      - "CASSANDRA_ENDPOINT_SNITCH=GossipingPropertyFileSnitch"
    depends_on:
      - cassandra-seed

volumes:
  cassandra_data_seed:
  cassandra_data_1:
  cassandra_data_2:
```

2.  **Запуск кластера:**

```bash
docker-compose up -d
```

##   Настройка Cassandra

1.  **Подключение к Cassandra:**

```bash
docker exec -it cassandra-seed cqlsh
```

2.  **Создание Keyspace:**

```cql
CREATE KEYSPACE my_keyspace WITH REPLICATION = { 'class' : 'SimpleStrategy', 'replication_factor' : 3 };
USE my_keyspace;
```

3.  **Создание таблицы:**

```cql
CREATE TABLE users (
    user_id UUID,
    item_id UUID,
    purchase_date TIMESTAMP,
    quantity INT,
    price DECIMAL,
    PRIMARY KEY ((user_id, item_id), purchase_date)
);

CREATE TABLE products (
    product_id UUID PRIMARY KEY,
    name TEXT,
    description TEXT
);
```

##   Работа с данными

1.  **Вставить данные:**

```cql
INSERT INTO users (user_id, item_id, purchase_date, quantity, price) VALUES (UUID(), UUID(), toTimestamp(now()), 2, 25.50);
INSERT INTO users (user_id, item_id, purchase_date, quantity, price) VALUES (UUID(), UUID(), toTimestamp(now()), 1, 10.00);
INSERT INTO users (user_id, item_id, purchase_date, quantity, price) VALUES (UUID(), UUID(), toTimestamp(now()), 3, 30.00);

INSERT INTO products (product_id, name, description) VALUES (UUID(), 'Laptop', 'High-performance laptop');
INSERT INTO products (product_id, name, description) VALUES (UUID(), 'Mouse', 'Wireless mouse');
```

2.  **Получить данные:**

```cql
SELECT * FROM users WHERE user_id = <user_id_значение> AND item_id = <item_id_значение> AND purchase_date = <purchase_date_значение>;
SELECT * FROM users WHERE user_id = <user_id_значение> AND item_id = <item_id_значение>;
SELECT * FROM products WHERE product_id = <product_id_значение>;
```

3.  **Создание вторичного индекса:**

```cql
CREATE INDEX ON products (name);
SELECT * FROM products WHERE name = 'Laptop';
```

##   Нагрузочное тестирование с Cassandra Stress Tool

Cassandra Stress Tool - это инструмент командной строки, который поставляется с Cassandra и предназначен для создания нагрузки на кластер, имитируя различные сценарии использования и измеряя производительность.

**Важные моменты при использовании с Docker:**

* Инструмент `cassandra-stress` находится внутри контейнера Cassandra. Необходимо запускать его внутри одного из работающих контейнеров.
* Убедиться, что инструмент может подключаться ко всем узлам Cassandra в кластере.
* Помнить об ограничениях ресурсов (CPU, память, ввод-вывод), потребляемых самим инструментом.

**Шаги для нагрузочного тестирования:**

1.  **Подключиться к контейнеру Cassandra:**

```bash
docker exec -it cassandra-seed bash
```

2.  **Найти `cassandra-stress`:**

Обычно находится в `/usr/bin/`.

```bash
which cassandra-stress
```

3.  **Базовый тест записи:**

```bash
cassandra-stress write n=10000 -rate threads=50
```

* `write`: Тип операции (запись).
* `n=10000`: Количество операций.
* `-rate threads=50`: Управление скоростью с помощью потоков.

4.  **Более сложные тесты:**

* **Смешанный тест чтения/записи:**

```bash
cassandra-stress mixedratio(write=90,read=10) n=10000 -rate threads=50
```

* **Пользовательские таблица и данные:**

Использовать `-schema` и `-pop` для определения схемы и данных.

* **Различная кардинальность ключей партиции:**

Использовать `cardinality` для управления количеством уникальных ключей партиции.

5.  **Мониторинг кластера:**

Во время теста использовать:

* `nodetool status`: Проверка состояния узлов.
* `nodetool tpstats`: Статистика пула потоков.
* `nodetool cfstats`: Статистика таблиц.
* `docker stats`: Потребление ресурсов контейнерами.

6.  **Анализ результатов:**

Инструмент предоставляет метрики, такие как скорость операций, задержка и ошибки.


