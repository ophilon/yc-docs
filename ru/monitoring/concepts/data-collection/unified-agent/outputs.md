# Выходы

Выход можно описать в секции `routes` внутри элемента `channel`:`output`. Либо в секции именованных каналов `channels`.

Общий формат описания выхода:

```yaml
- output:
    plugin: ... # имя плагина
    id: ... # (рекомендуемый) идентификатор выхода, используется в метриках и логах работы агента
```

## Выход debug {#debug_output}

Отладочный выход — выводит поступающие сообщения в файл или в консоль.

Так как хранилище не поддерживает локальный просмотр данных, вы можете настроить выгрузку данных в файл через этот выход. Ротацию можно настроить через утилиту `logrotate`. Чтобы агент начал записывать логи в новый файл, воспользуйтесь методом `reopen_file`. 

Пример конфигурации:

```yaml
status:
  port: 16301

output:
  plugin: debug
  id: my_output_id
  config:
    file_name: ./data/output
    delimiter: "\n===\n"
    _test:
      register_test_handlers: true
```

Описание параметров:

```yaml
- channel:
    output:
        plugin: debug
        config:
            # Должно быть указано одно из свойств file_name или directory.

            # Имя файла, в который будут записываться сообщения.
            # Для вывода в консоль укажите значение "/dev/stdout".
            file_name: out.txt  # необязательный, по умолчанию не задан

            # Имя директории. Если указано, данные каждой сессии будут записываться в отдельный файл в этой директории с именем, равным идентификатору сессии.
            directory: output_directory  # необязательный, по умолчанию не задан

            # Разделитель сообщений в файле, например "\n".
            delimiter: null  # обязательный
```

## Выход dev_null {#dev_null_output}

Отладочный «пустой» выход — не сохраняет поступающие сообщения. Параметров нет.

## Выход yc_metrics {#yc_metrics_output}

Выход для записи метрик в {{ monitoring-full-name }} API.

Описание параметров:

```yaml
- channel:
    output:
        plugin: yc_metrics
        config:
        # URL, на который будут отправляться метрики.
        url: https://monitoring.{{ api-host }}/monitoring/v2/data/write  # необязательный, по умолчанию https://monitoring.{{ api-host }}/monitoring/v2/data/write

        folder_id: b1ge2vt0gml6********  # обязательный, идентификатор каталога

        # Настройки IAM-аутентификации.
        iam:  # обязательный
            # Должен быть указан один из элементов cloud_meta или jwt.

            # Если указан, IAM-токен берется из сервиса метаданных.
            cloud_meta: {}  #необязательный, по умолчанию не задан

            # Если указан, JWT обменивается на IAM-токен.
            jwt:  #необязательный, по умолчанию не задан
            # Имя файла с параметрами JWT в формате, который возвращает команда `yc iam key create`.
                file: "jwt_params.json"  # обязательный

                endpoint: iam.{{ api-host }}  # необязательный, по умолчанию iam.{{ api-host }}

                refresh_period: 1h  # необязательный, по умолчанию 1h

                request_timeout: 10s  # необязательный, по умолчанию 10s

        # Число повторных попыток, если запрос завершился с ошибкой.
        # Если за указанное число запрос так и не был выполнен успешно (то есть не был получен ответ со статусом 200), сообщение отбрасывается, то есть по нему формируется подтверждение в сторону агента.
        # Отброшенные таким образом сообщения учитываются в счетчике DroppedMessages этого плагина, а также отображаются в общих health-счетчиках MessagesLost и BytesLost.
        retry_count: inf  # необязательный, по умолчанию max_int, то есть сообщения отбрасываться не будут

        # Задержка между повторными попытками.
        retry_delay: 1s  # необязательный, по умолчанию 1 секунда

        # Таймаут запроса, включая все повторные попытки.
        timeout: inf  # необязательный, по умолчанию max_int секунд, то есть сообщения отбрасываться не будут

        # Таймаут на одну попытку.
        request_timeout: 1s  # необязательный, по умолчанию одна секунда
```


## Выход yc_logs {#yc_logs_output}

Выход для отправки логов в [{{ cloud-logging-full-name }}](../../../../logging/) по протоколу [grpc](../../../../logging/api-ref/grpc/LogIngestion/index.md).

{% note info %}

{{ unified-agent-short-name }} устраняет дубликаты сообщений в целевой системе, но не гарантирует их отсутствие. Дубликаты могут возникать при остановке и повторном запуске.

{% endnote %}

Описание параметров:

```yml

- channel:
    output:
      plugin: yc_logs
      config:
        # Необязательный. URL, на который будут отправляться логи
        url: "{{ logging-endpoint-ingester }}:443"

        # Необязательный. Использовать SSL-соединение.
        use_ssl: null # директива выключает SSL, по умолчанию SSL включен
          # Опционально можно явно задать список корневых сертификатов сервера
          # в виде пути к файлу в PEM-формате.
          # Содержимое файла передается в параметр pem_root_certs.
          root_certs_file: null  # необязательный, по умолчанию не задан

        # Настройки целевой лог-группы.
        # Должен быть указан один из элементов message_log_group_id, message_folder_id, log_group_id или folder_id.
        # Значение может быть задано на двух уровнях:
        # 1) на уровне сообщения для этого нужно воспользоваться !expr-синтаксисом для message_log_group_id и для message_folder_id:
        #    message_log_group_id: !expr "{message_my_log_group}"
        #    Имя log_group_id будет взято из метаданных сообщения с ключом 'message_my_log_group'.
        # 2) на уровне сессии для этого нужно воспользоваться !expr-синтаксисом для log_group_id и для folder_id:
        #    log_group_id: !expr "{my_log_group}"
        #    Имя log_group_id будет взято из метаданных сессии с ключом 'my_log_group'.
        # Приоритет выбора лог-группы следующий:
        # message_log_group_id > message_folder_id > log_group_id > folder_id
        # В первую очередь происходит поиск в метаданных сообщения лог-группы (если параметр задан),
        # затем в метаданных сообщения поиск папки, затем в метаданных сессии поиск лог-группы,
        # затем в метаданных сессии поиск папки. Будет выбрана та, которую удалось найти первой.
        log_group_id: "b1ge2vt0gml6********"  # по умолчанию не задан

        folder_id: "b1ge2vt0gml6********"  # по умолчанию не задан; если указан, будет использована лог-группа по умолчанию

        message_log_group_id: "b1ge2vt0gml6********" # по умолчанию не задан

        message_folder_id: "b1ge2vt0gml6********"  # по умолчанию не задан; если указан, будет использована лог-группа по умолчанию

        # Лимит на количество уникальных лог-групп, в которые будет происходить одновременная отправка в рамках одной сессии.
        # Если приходит сообщение с новой уникальной лог-группой, в которую не происходит отправка в этот момент
        # и лимит буферов исчерпан, то оно будет отброшено
        buffer_count_limit: 50 # необязательный, по умолчанию 50

        # Настройки батчинга.
        batch:
          # Лимит на размер результата.
          # Новое сообщение формируется, если размер поступивших достиг лимита.
          limit: # необязательный
            # Ограничение может быть указано в единицах сообщений или в байтах.
            # Если указаны оба, будет применено первое достигнутое.

            # Количество поступающих сообщений в штуках.
            count: 100  # необязательный, по умолчанию 100, возможные значения: [1-100]

            # Размер поступающих сообщений в байтах.
            # Учитывается только размер тела сообщения.
            bytes: null  # необязательный, по умолчанию не задан

          # Лимит на время ожидания.
          # Новое сообщение формируется, если с момента получения время ожидания достигло лимита.
          # Формируется, если с момента получения первого сообщения в батче прошло больше времени.
          flush_period: 1s  # необязательный, по умолчанию 1s

        # Таймаут запроса на одну попытку.
        grpc_call_timeout: 3s  # необязательный, по умолчанию 3s

        # Число повторных попыток, если запрос завершился с ошибкой.
        # Если за указанное значение запрос так и не был выполнен
        # успешно (то есть не был получен ответ со статусом 200),
        # сообщение отбрасывается, то есть по нему формируется ack в сторону агента.
        # Отброшенные таким образом сообщения учитываются в счетчике DroppedMessages этого плагина,
        # а так же отражаются в общих health-счетчиках MessagesLost и BytesLost.
        retry_count: inf  # необязательный, по умолчанию max_int — сообщения отбрасываться не будут

        # Если запрос не удалось выполнить за retry_count раз, повторные попытки будут
        # возобновлены с экспоненциально увеличивающейся задержкой.
        # Параметр задает минимальное значение задержки.
        backoff_min_delay: 1s  # необязательный, по умолчанию 1 секунда

        # Если запрос не удалось выполнить за retry_count раз, повторные попытки будут
        # возобновлены с экспоненциально увеличивающейся задержкой.
        # Параметр задает максимальное значение задержки.
        backoff_max_delay: 1m  # необязательный, по умолчанию 1 минута

        # Квота на скорость записи сообщений (сообщений в секунду)
        message_quota: 1000

        # Настройки IAM-аутентификации.
        iam:  # обязательный, по умолчанию не задан
          # Должен быть указан один из элементов: cloud_meta или jwt.

          # Если указан, IAM-токен берется из сервиса метаданных.
          cloud_meta:  #необязательный, по умолчанию не задан
            url:  # необязательный

            refresh_period: 1h  # необязательный, по умолчанию 1h

            request_timeout: 3s  # необязательный, по умолчанию 10s

          # Если указан, JWT-токен обменивается на IAM-токен.
          jwt:  # необязательный, по умолчанию не задан
            # Имя файла с параметрами JWT в формате, который возвращает команда `yc iam key create`
            file: "jwt_params.json"  # обязательный

            endpoint: iam.api.cloud.yandex.net  # необязательный, по умолчанию iam.api.cloud.yandex.net

            refresh_period: 1h  # необязательный, по умолчанию 1h

            request_timeout: 10s  # необязательный, по умолчанию 10s

            use_private_api: false # необязательный, по умолчанию false, если установлен в true, необходимо явно задать endpoint

        # Список экспортируемых метаданных сообщения. Метаданные с ключами входящими в этот список будут переданы в
        # сервис Yandex Cloud Logging в виде json_payload.
        # Список строк - ключей метаданных.
        export_message_meta_keys:  # необязательный, по умолчанию не задан
          - "_app"
          - "_random"

        # режим применения export_message_meta_keys — черный или белый список (значения white или black)
        # в режиме черного списка, экспортируются все метаданные сообщения, за исключением служебных и указанных в export_message_meta_keys
        message_meta_keys_list_mode: white # необязательный, по умолчанию white

        # Настройки парсинга тела сообщения.
        # Если тело сообщения – валидный JSON, и в нем будет найден новый payload, то:
        #   1. Тело сообщения будет заменено на найденный payload.
        #   2. Если будут найдены специальные поля (timestamp, priority), и их парсинг удастся, они заменят соответствующие поля сообщения.
        #   3. Все остальные поля будут переложены в json_payload.
        parse_json: # необязательный
          # Правило, откуда брать новое тело сообщения.
          payload: # обязательный
            # Перечень ключей первого уровня.
            # Будет выбрано значение, соответствующее первому найденному из списка ключу.
            keys: ["msg", "message"]

          # Ключи в JSON, соответствующие времени сообщения.
          # Если ключи в JSON отсутствуют, или парсинг значения не удастся, будет использовано время получения сообщения.
          # Допустимые значения:
          # - unix timestamp (в секундах или регулируется unix_timestamp_format, опционально с дробной частью)
          # - ISO 8601
          # - RFC 822
          # - log4j {"epochSecond":123,"nanoOfSecond":456}
          timestamp: # необязательный
            keys: ["ts", "timestamp", "instant"]

          # Единицы измерения timestamp: seconds, milliseconds, microseconds
          unix_timestamp_format: seconds # необязательный

          # Ключи в JSON, соответствующие priority.
          # Если ключи в JSON отсутствуют, или парсинг значения не удастся, будет использовано значение из метаинформации.
          # Допустимые значения: TRACE, DEBUG, INFO, WARN, ERROR, FATAL (регистр не учитывается).
          priority: # необязательный
            keys: ["level"]

        # Параметры для LogEntryResource https://cloud.yandex.ru/docs/logging/api-ref/grpc/log_ingestion_service#LogEntryResource
        resource_id: some_id # необязательный, по умолчанию не задан
        resource_type: some_type # необязательный, по умолчанию не задан
        stream_name: some_name # необязательный, по умолчанию не задан

```
