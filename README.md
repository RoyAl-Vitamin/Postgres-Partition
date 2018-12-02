# Postgres-Partition
Пример партиционирования в БД Postgres

## Настройка рабочего окружения

Что бы не устанавливать Postgres, поставим его с помощью Docker и запустим

```trerminal
docker pull postgres
docker run -p 5432:5432 --name some-postgres -e POSTGRES_PASSWORD=postgres -d postgres
```

То же самое сделаем с PGAdmin

```terminal
docker pull dpage/pgadmin4
docker run -p 81:80 --link some-postgres -e "PGADMIN_DEFAULT_EMAIL=email@domain.com" -e "PGADMIN_DEFAULT_PASSWORD=postgres" -d dpage/pgadmin4
```

## Пример работы:

Создадим таблицу и партиционируем её

```SQL
CREATE TABLE measurement (
city_id         int not null,
logdate         date not null,
peaktemp        int,
unitsales       int
) PARTITION BY RANGE (logdate);

CREATE TABLE measurement_y2006m02 PARTITION OF measurement
FOR VALUES FROM ('2006-02-01') TO ('2006-03-01');

CREATE TABLE measurement_y2006m03 PARTITION OF measurement
FOR VALUES FROM ('2006-03-01') TO ('2006-04-01');
```

Создадим индексы

```SQL
CREATE INDEX ON measurement_y2006m02 (logdate);
CREATE INDEX ON measurement_y2006m03 (logdate);
```

Вставим тестовые данные

```SQL
INSERT INTO measurement
(city_id, logdate, peaktemp, unitsales)
VALUES
(1, '2006-03-02', 12, 234);
INSERT INTO measurement
(city_id, logdate, peaktemp, unitsales)
VALUES
(1, '2006-03-04', 12, 234);
INSERT INTO measurement
(city_id, logdate, peaktemp, unitsales)
VALUES
(1, '2006-02-02', 12, 9999);
INSERT INTO measurement
(city_id, logdate, peaktemp, unitsales)
VALUES
(1, '2006-02-10', 12, 8888);
INSERT INTO measurement
(city_id, logdate, peaktemp, unitsales)
VALUES
(1, '2006-02-12', 12, 7777);
```

Удалим партицию

```SQL
DROP TABLE measurement_y2006m02;
```

Так же можно отсоединить раздел (сохранить в виде отдельной таблицы)

```SQL
ALTER TABLE measurement DETACH PARTITION measurement_y2006m02;
```
Примеры взяты с [сайта](https://www.postgresql.org/docs/10/static/ddl-partitioning.html).
