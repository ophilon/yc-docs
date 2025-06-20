# Создание кластера {{ PG }} для «1С:Предприятия»


{{ mpg-name }} позволяет создавать отказоустойчивые кластеры {{ PG }}, оптимизированные для работы с системой «1С:Предприятие». Для этого в сервисе поддерживаются версии PostgreSQL {{ pg.versions.console.str-1c }}, в которых установлены все необходимые [расширения](#extensions) и изменена конфигурация менеджера подключений.

{% note warning %}

Систему «1С:Предприятие» можно подключить только к кластерам версии {{ pg.versions.console.str-1c }}.

{% endnote %}

При выборе [класса хоста](../../managed-postgresql/concepts/instance-types.md) ориентируйтесь на количество пользователей вашей инсталляции «1С:Предприятия». На хостах класса **s2.small** смогут одновременно работать до 50 пользователей. Класс **s2.medium** рекомендуется использовать, если с базой будут работать 50 и более пользователей. Размер хранилища следует выбирать исходя из размеров вашей информационной базы — учитывайте возможный рост объемов данных.


## Необходимые платные ресурсы {#paid-resources}

В стоимость поддержки описываемого решения входит:

* Плата за кластер {{ mpg-name }}: использование вычислительных ресурсов, выделенных хостам, и дискового пространства (см. [тарифы {{ mpg-name }}](../../managed-postgresql/pricing.md)).
* Плата за использование публичных IP-адресов, если для хостов кластера включен публичный доступ (см. [тарифы {{ vpc-name }}](../../vpc/pricing.md)).


## Создайте кластер {{ mpg-name }} {#create-cluster}

{% list tabs group=instructions %}

- Вручную {#manual}

    [Создайте](../../managed-postgresql/operations/cluster-create.md#create-cluster) кластер {{ mpg-name }} любой подходящей конфигурации со следующими настройками:

    * **{{ ui-key.yacloud.mdb.forms.base_field_environment }}** — `PRODUCTION`.
    * **{{ ui-key.yacloud.mdb.forms.base_field_version }}** — версия {{ PG }} для работы с системой «1С:Предприятия». Название таких версий заканчивается на `-1с`.
    * **{{ ui-key.yacloud.mdb.forms.section_resource }}** — не ниже `s2.small`.
    * **{{ ui-key.yacloud.mdb.forms.section_host }}** — добавьте не меньше двух дополнительных хостов, разместив их в разных зонах доступности. Это обеспечит отказоустойчивость кластера. Репликация между хостами будет настроена автоматически. Подробнее см. в разделе [{#T}](../../managed-postgresql/concepts/replication.md).

- {{ TF }} {#tf}

    1. {% include [terraform-install-without-setting](../../_includes/mdb/terraform/install-without-setting.md) %}
    1. {% include [terraform-authentication](../../_includes/mdb/terraform/authentication.md) %}
    1. {% include [terraform-setting](../../_includes/mdb/terraform/setting.md) %}
    1. {% include [terraform-configure-provider](../../_includes/mdb/terraform/configure-provider.md) %}

    1. Скачайте в ту же рабочую директорию файл конфигурации [postgresql-1c.tf](https://github.com/yandex-cloud-examples/yc-postgresql-1c/blob/main/postgresql-1c.tf).

        В этом файле описаны:

        * [сеть](../../vpc/concepts/network.md#network);
        * [подсеть](../../vpc/concepts/network.md#subnet);
        * [группа безопасности](../../vpc/concepts/security-groups.md) и правило, разрешающее подключение к кластеру;
        * кластер {{ mpg-name }} для «1С:Предприятия» с базой данных и пользователем.

    1. Укажите параметры инфраструктуры в файле конфигурации `postgresql-1c.tf` в блоке `locals`:

        * `cluster_name` — имя кластера.
        * `pg_version` — версия {{ PG }} для работы с системой «1С:Предприятия». Название таких версий заканчивается на `-1с`.
        * `db_name` — имя базы данных.
        * `username` и `password` — имя и пароль пользователя-владельца базы данных.

    1. Проверьте корректность файлов конфигурации {{ TF }} с помощью команды:

       ```bash
       terraform validate
       ```

       Если в файлах конфигурации есть ошибки, {{ TF }} на них укажет.

    1. Создайте необходимую инфраструктуру:

       {% include [terraform-apply](../../_includes/mdb/terraform/apply.md) %}

       {% include [explore-resources](../../_includes/mdb/terraform/explore-resources.md) %}

{% endlist %}

Создание кластера БД может занять несколько минут.

## Подключите базу к «1С:Предприятию» {#connect-to-1c}

Подключите созданную базу в качестве информационной базы «1С:Предприятия». При добавлении базы используйте следующие параметры:

* **Кластер серверов** — имя сервера «1С:Предприятия», на котором будет создана информационная база.
* **Имя информационной базы в кластере** — имя создаваемой информационной базы на сервере «1С:Предприятия».
* **Защищенное соединение** — отключено.
* **Тип СУБД** — `PostgreSQL`.
* **Сервер баз данных** — `с-<идентификатор_кластера>.rw.{{ dns-zone }} port={{ port-mpg }}`.
* **Имя базы данных** — имя базы данных, указанное при создании кластера {{ mpg-name }}.
* **Пользователь базы данных** — имя пользователя-владельца базы данных.
* **Пароль пользователя** — пароль пользователя-владельца базы данных.
* **Создать базу данных в случае ее отсутствия** — отключено.

### Расширения {{ PG }} для поддержки системы «1С:Предприятие» {#extensions}

Список расширений, которые установлены в кластерах PostgreSQL версии {{ pg.versions.console.str-1c }}:

* [online_analyze](https://postgrespro.ru/docs/postgrespro/10/online-analyze)

* [plantuner](https://postgrespro.ru/docs/postgrespro/10/plantuner)

* [fasttrun](https://postgrespro.ru/docs/postgrespro/10/fasttrun)

* [fulleq](https://postgrespro.ru/docs/postgrespro/10/fulleq)

* [mchar](https://postgrespro.ru/docs/postgrespro/10/mchar)

## Удалите созданные ресурсы {#clear-out}

Удалите ресурсы, которые вы больше не будете использовать, чтобы за них не списывалась плата:

{% list tabs group=instructions %}

- Вручную {#manual}

    [Удалите кластер {{ mpg-name }}](../../managed-postgresql/operations/cluster-delete.md).

- {{ TF }} {#tf}

    {% include [terraform-clear-out](../../_includes/mdb/terraform/clear-out.md) %}

{% endlist %}
