# yandex.practicum
Тестовое задание для Яндекс Практикум

## Задание:
Используя API [exchangerate.host](https://exchangerate.host/) подготовьте ETL процесс с помощью Airflow на python, для выгрузки данных по валютной паре BTC/USD (Биткоин к доллару), выгружать следует с шагом 3 часа и записывать данные в БД (валютная пара, дата, текущий курс).

В качестве задания со звёздочкой можете учесть вариант заполнения базы историческими данными (API предоставляет специальный endpoint для выгрузки исторических данных).

## Алгоритм запуска сервиса:
1. Необходимо склонировать репозиторий `git clone`
2. Запустить docker-compose командой `docker-compose up -d`

После запуска docker контейнеров, будут запущены сервисы airflow и clickhouse
- веб-сервер airflow - `http://localhost:8080/home` (необходимо подождать ~ 1 минуту для запуска airflow webserver)
- клиент tabix для clickhouse - `http://localhost:8124/`

## Порядок работы DAG's:
- *PARSE_PAIRS_FROM_EXCHANGERATE* - dag для выгрузки данных по валютной паре BTC/USD (Биткоин к доллару), которые выгружаются каждые 3 часа
- *HISTORY_PAIRS_FROM_EXCHANGERATE* - dag для выгрузки данных, за определенный период указанный пользователем в истории, по валютной паре BTC/USD (Биткоин к доллару), запускает вручную по необходимости

TASK'S:
1. create_stg_table_in_ch - таска для создания таблицы, в которую поступают данные 1:1 из первоисточника
2. create_core_table_in_ch - таска для создания таблицы, в которую поступают данные уже очищенные и преобразованы к необходимым типам данных
3. stg_load_to_ch - загрузка данных из первоисточника в таблицу из п.1
4. в зависимости от типа dag файла:
  - core_load_to_ch (PARSE_PAIRS_FROM_EXCHANGERATE) - тут происходит удаление партиции по дате указанной в параметрах к таске для избежания дублирования информации и загрузка в таргет таблицу из п.2
  - hs_load_to_ch (HISTORY_PAIRS_FROM_EXCHANGERATE) - тут происходит удаление партиции по датам указанной в параметрах к таске для избежания дублирования информации и загрузка в таргет таблицу из п.2
5. truncate_stg_table - очистка stg таблицы с сырыми данными, для того, чтоб избежать дублирования информации в дальнейшем

