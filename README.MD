# Holistic.dev cloud function for SberCloud integration

Функция предназначена для регулярной автоматической отправки актуального состояния представления pg_stat_statements указанной базы PostgreSQL в сервис **[holistic.dev](https://holistic.dev)** для дальнейшего анализа.

Функция может быть использована для интеграции как managed PostgreSQL, так и для on-premise экземпляров PostgreSQL.

После окончания процесса настройки вы сможете увидеть все DML-запросы в соответствующем разделе клиентского кабинета.

# Общая информация о функции
Функция разработана для использования в окружении **SberCloud** в среде выполнения nodejs12.
Функция выполняет запрос в базу данных, параметры подключения к которой будут переданы в переменной окружения.
Функция выбирает исходный код и метрики всех записей из pg_stat_statements и отправляет на API **[holistic.dev](https://holistic.dev)**

# Начало работы
- зарегистрируйтесь в [личном кабинете](https://app.holistic.dev/)
- создайте новый проект
- загрузите [DDL проекта](https://docs.holistic.dev/#database-schema-ddl)
- из аккаунта настроек скопируйте API-key

# Network availability
Экземпляр базы данных должен быть доступен для подключения из функции, который происходит через внешний IP. Для того, чтобы иметь возможность подключаться к PostgreSQL из вне, необходимо привязать внешний IP в сервисе **[Elastic IP](https://support.hc.sbercloud.ru/en-us/usermanual/vpc/en-us_topic_0013748738.html)**

Убедитесь, что текущая Security Group экземпляра базы допускает подключение из внешних источников.


# pg_stat_statements extension
### Managed PostgreSQL
Для активации расширения pg_stat_statements требуется выполнить команду

```sql
select control_extension('create','pg_stat_statements');
```

Подробное описание в [документации](https://support.hc.sbercloud.ru/usermanual/rds/en-us_topic_0077893062.html)


### On-premise PostgreSQL
Для активации расширения выполните команду в базе, для которой производится интеграция

```sql
CREATE EXTENSION IF NOT EXISTS pg_stat_statements;
```

### Настройка расширения
pg_stat_statements имеет несколько настроек, с которыми можно ознакомиться в [документации к расширению](https://www.postgresql.org/docs/current/pgstatstatements.html#id-1.11.7.38.8)

# Установка
### Создание функции
- скачайте [архив из данного репозитория](https://github.com/holistic-dev/faas-sbercloud-pg/blob/master/faas-sbercloud-pg.zip) с исходным кодом функции
- в сервисе **FunctionGraph** перейдите к диалогу создания новой функцию
- укажите Runtime **Node.js 12**
- Code Entry Mode - **Upload ZIP file**, выберете скаченный архив с исходным кодом функции
- точка входа - **index.handler**
- после создания функции в закладке Configuration создайте переменные окружения:
  - **HOLISTICDEV_API_KEY** - API-key из настроек в личном кабинете
  - **HOLISTICDEV_PROJECT_NAME** - имя вашего проекта
  - **PG_CONNECTION_STRING** - строка подключения из настроек кластера managed PostgreSQL или on-premise экземпляра в формате ```postgres://<user>:<password>@<host>:<port>/<dbname>```.


### Тестирование функции 
- в верхнем меню нажмите **"Test"**
- в закладке **"Logs"** через несколько секунд появятся данные. Если все параметры функции указаны верно, в логах вы увидите сериализованный объект вида 
```{ status: 'OK', data: { pgss: { income: 100, new: 100 } } }```

### Настройка триггера
- в закладке **Triggers** создайте новый триггер с типом **Timer**
- в разделе **Rule** укажите **Fixed Rate** 15 минут. Это значение позволит запускать отправку каждые 15 минут. Не рекомендуется устанавливать данное значение для излишне частого срабатывания, т.к. история хранения метрик pg_stat_statements в сервисе **[holistic.dev](https://holistic.dev)** имеет конечную глубину.
