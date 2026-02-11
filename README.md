# Домашнее задание к занятию "`Резервное копирование баз данных`" - ``

---

### Задание 1

**1.1. Восстановление данных в полном объёме за предыдущий день**

**Выбранный вариант** - полное резервное копирование (full backup) ежедневно

- Бекапы выполняются вне пиковых часов (например, ночью).
- Выполняется обязательная проверка целостности бекапов (восстановление на тестовом стенде).
- Хранится несколько последних полных копий (например, за 7–30 дней).`

**1.2. Восстановление данных за час до поломки**

**Полное резервное копирование (еженедельно) + инкрементное/дифференциальное копирование (каждый час)**

Позволяет восстановить данные на момент максимально близкий к сбою. Для восстановления потребуется последний полный бекап + все инкрементные бекапы до нужного часа.

**1.3. Мгновенное переключение при поломке***

Решение, когда при поломке базы происходит моментальное переключение на работающую или починенную базу данных, возможно в следующем исполнении:

**1. Репликация баз данных**

Master-Slave — при падении мастера переключение на реплику.

Master-Master — более сложная, но позволяет распределять нагрузку.

**2. Кластеризация**

Active-Passive — одна нода активна, вторая в режиме горячего резерва.

Active-Active — все ноды работают, при падении одной нагрузка перераспределяется.

**3.Автоматический failover**

Использование встроенных механизмов СУБД (например, PostgreSQL с patroni, MySQL с group replication, MongoDB replica sets).

Или инструментов Kubernetes StatefulSets с операторами.

---

### Задание 2

2.1. Примеры команд резервирования данных и восстановления БД (pgdump/pgrestore)

*Резервное копирование*

```
pg_dump -U username -h localhost -d database_name -F c -b -v -f /backup/backup_file.dump

pg_dump -U username -d database_name | gzip > /backup/backup_file.sql.gz

pg_dump -U username -d database_name -s -f /backup/schema_only.sql

pg_dump -U username -d database_name -t table_name -f /backup/table_backup.sql

```
*Восстановление*

```
pg_restore -U username -h localhost -d new_database -v /backup/backup_file.dump

pg_restore -U username -d database_name -s -v /backup/backup_file.dump

pg_restore -U username -d database_name -t table_name -v /backup/backup_file.dump

psql -U username -d database_name -f /backup/backup_file.sql

```
2.1*. Автоматизировать резервирования данных и восстановления БД можно следующими способами:

1. Выполнение вышеуказанных команд через скрипт автоматического бекапа (bash) и настройку cron
2. Использование специализированных инструментов, таких как: Barman (Backup and Recovery Manager), pgBackRest
3. ИИспользование WAL (Write-Ahead Log) архивирования

### Задание 3

Примеры команд инкрементного резервного копирования базы данных MySQL

*Предварительные требования*
```
# my.cnf
[mysqld]
server-id = 1
log_bin = /var/log/mysql/mysql-bin.log
binlog_format = ROW
expire_logs_days = 7
max_binlog_size = 100M
```
*Создание бекапа*
```
mysqldump -u root -p --all-databases --flush-logs --master-data=2 \
  --single-transaction --routines --triggers --events \
  > /backup/full_backup_$(date +%Y%m%d).sql
```
*Создание инкрементного бекапа (копирование бинарных логов)*


