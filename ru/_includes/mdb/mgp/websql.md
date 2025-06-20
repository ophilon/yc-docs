Вы можете отправлять запросы к базам данных в кластере {{ mgp-name }} с помощью сервиса [{{ websql-full-name }}](../../../websql).

{{ websql-name }} — это сервис {{ yandex-cloud }}, который позволяет подключаться к кластерам управляемых баз данных, работать с БД, таблицами и схемами и выполнять запросы. Сервис работает в браузере, не требует дополнительной авторизации и предлагает удобные подсказки для работы с командами {{ VLK }}.

Чтобы подключаться из {{ websql-name }} к кластеру {{ mgp-name }}, необходимо создать подключение:

1. Перейдите на страницу каталога и выберите сервис **{{ ui-key.yacloud.iam.folder.dashboard.label_managed-greenplum }}**.
1. Нажмите на имя нужного кластера.
1. [Включите опцию](../../../managed-greenplum/operations/update.md#change-additional-settings) **{{ ui-key.yacloud.mdb.forms.additional-field-websql-service }}** в настройках кластера, если она еще не включена.
1. Выберите вкладку **{{ ui-key.yacloud.mdb.cluster.switch_explore-websql }}**.
1. Нажмите кнопку **{{ ui-key.yacloud.mdb.cluster.websql-connections.action_create-connection }}** и укажите параметры подключения:
    * **Тип базы данных** — {{ GP }}.
    * **Каталог кластера** — будет автоматически выбран текущий каталог.
    * **Кластер** — будет автоматически выбран текущий кластер {{ GP }}.
    * **Имя пользователя**, от лица которого вы будете подключаться к базе данных в кластере.
    * **Пароль** пользователя.
    * **Базы данных**, к которым вы хотите подключиться. Вы можете подключиться только к существующим в этом кластере БД. У пользователя, которого вы указали, должен быть настроен доступ к ним.
1. Нажмите кнопку **Создать**.

Чтобы открыть редактор запросов, нажмите на созданное подключение на вкладке **{{ ui-key.yacloud.mdb.cluster.switch_explore-websql }}**.

Подробнее о работе с {{ websql-name }} см. в [документации сервиса](../../../websql/operations/index.md).
