---
title: Изменение кластера {{ TR }}
description: После создания кластера {{ TR }} вы можете изменить его основные и дополнительные настройки.
keywords:
  - изменение кластера {{ TR }}
  - кластер {{ TR }}
  - '{{ TR }}'
---

# Изменение кластера {{ TR }}

{% include [preview](../../_includes/managed-trino/note-preview.md) %}

После создания кластера вы можете изменить его основные и дополнительные настройки.

{% list tabs group=instructions %}

- Консоль управления {#console}

    Чтобы изменить настройки кластера:

    1. Перейдите на [страницу каталога]({{ link-console-main }}) и выберите сервис **{{ ui-key.yacloud.iam.folder.dashboard.label_managed-trino }}**.
    1. Выберите кластер и нажмите кнопку **{{ ui-key.yacloud.mdb.clusters.button_action-edit }}** на панели сверху.
    1. В блоке **{{ ui-key.yacloud.mdb.forms.section_base }}**:

        * Измените имя и описание кластера.
        * Удалите или добавьте новые [метки](../../resource-manager/concepts/labels.md).
        * Выберите сервисный аккаунт или [создайте новый](../../iam/operations/sa/create.md#create-sa) с ролями `managed-trino.integrationProvider` и `storage.editor`. Это даст кластеру нужные права для работы с пользовательскими ресурсами. Подробнее см. в разделе [Имперсонация](../concepts/impersonation.md).

            Для изменения сервисного аккаунта в кластере {{ mtr-name }} [убедитесь](../../iam/operations/roles/get-assigned-roles.md), что вашему аккаунту в {{ yandex-cloud }} назначена роль [iam.serviceAccounts.user](../../iam/security/index.md#iam-serviceAccounts-user) или выше.

            {% include [mdb-service-account-update](../../_includes/mdb/service-account-update.md) %}

    1. В блоке **{{ ui-key.yacloud.mdb.forms.section_network-settings }}** выберите группу безопасности для кластера.
    1. В блоке **Политика перезапросов** измените параметры [отказоустойчивого выполнения запросов](../concepts/retry-policy.md):
        * Измените **Тип объекта для перезапроса**.
        * Добавьте или удалите в поле **Параметры перезапросов** дополнительные параметры в формате `ключ: значение`. Подробнее о параметрах см. в [документации {{ TR }}](https://trino.io/docs/current/admin/fault-tolerant-execution.html#advanced-configuration).
        * Добавьте или удалите в поле **Параметры хранилища** дополнительные параметры хранилища Exchange Manager в формате `ключ: значение`. Подробнее о параметрах см. в [документации {{ TR }}](https://trino.io/docs/current/admin/fault-tolerant-execution.html#id1).
    1. Измените конфигурацию [координатора](../concepts/index.md#coordinator) и [воркеров](../concepts/index.md#workers).
    1. В блоке **{{ ui-key.yacloud.mdb.forms.section_additional }}**:
        * Установите или снимите защиту от удаления кластера.
        * Включите или выключите запись логов. Если логирование включено, выберите:

            * Место записи логов — пользовательская [лог-группа](../../logging/concepts/log-group.md) или лог-группа по умолчанию в выбранном каталоге.  
            * Минимальный уровень логирования.
    1. Нажмите кнопку **{{ ui-key.yacloud.mdb.forms.button_edit }}**.

- REST API {#api}

    Чтобы изменить настройки кластера:

    1. [Получите IAM-токен для аутентификации в API](../api-ref/authentication.md) и поместите токен в переменную среды окружения:

        {% include [api-auth-token](../../_includes/mdb/api-auth-token.md) %}

    1. Создайте файл `body.json` и добавьте в него следующее содержимое:

       {% note info %}

       В примере приведены не все доступные параметры. Список всех параметров см. в [документации по API](../api-ref/Cluster/update.md#yandex.cloud.trino.v1.UpdateClusterRequest).

       {% endnote %}

        ```json
        {
          "updateMask": "<список_изменяемых_параметров>",
          "name": "<имя_кластера>",
          "description": "<описание_кластера>",
          "labels": { <список_меток> },
          "trino": {
            "coordinatorConfig": {
              "resources": {
                "resourcePresetId": "<идентификатор_ресурсов>"
              }
            },
            "workerConfig": {
              "resources": {
                "resourcePresetId": "<идентификатор_ресурсов>"
              },
              "scalePolicy": {
                "autoScale": {
                  "minCount": "<минимальное_количество_экземпляров>",
                  "maxCount": "<максимальное_количество_экземпляров>"
                }
              }
            }
          },
          "retryPolicy": {
            "policy": "<тип_объекта_для_перезапроса>",
            "exchangeManager": {
              "storage": {
                "serviceS3": {}
              },
              "additionalProperties": {<дополнительные_параметры_хранилища>}
            },
            "additionalProperties": {<дополнительные_параметры_перезапросов>}
          },
          "networkSpec": {
            "securityGroupIds": [ <список_идентификаторов_групп_безопасности> ]
          },
          "deletionProtection": "<защита_от_удаления>",
          "serviceAccountId": "<идентификатор_сервисного_аккаунта>",
          "logging": {
            "enabled": "<использование_логирования>",
            "folderId": "<идентификатор_каталога>",
            "minLevel": "<уровень_логирования>"
          }
        }
        ```

        Где:

        * `updateMask` — перечень изменяемых параметров в одну строку через запятую.

            {% note warning %}

            При изменении кластера будут переопределены все параметры изменяемого объекта, которые не были явно переданы в запросе, на значения по умолчанию. Чтобы избежать этого, перечислите настройки, которые вы хотите изменить, в параметре `updateMask`.

            {% endnote %}

        * `name` — имя кластера.
        * `description` — описание кластера.
        * `labels` — список меток. Метки задаются в формате `"<ключ>": "<значение>"`.
        * `trino` — конфигурация [компонентов](../concepts/index.md#cluster-architecture) кластера {{ TR }}.

            * `coordinatorConfig` — конфигурация координатора.

               * `resources.resourcePresetId` — идентификатор вычислительных ресурсов координатора. Возможные значения:

                    * `c4-m16` — 4 vCPU, 16 ГБ RAM.
                    * `c8-m32` — 8 vCPU, 32 ГБ RAM.

            * `workerConfig` — конфигурация воркера.

               * `resources.resourcePresetId` — идентификатор вычислительных ресурсов воркера. Возможные значения:

                    * `c4-m16` — 4 vCPU, 16 ГБ RAM.
                    * `c8-m32` — 8 vCPU, 32 ГБ RAM.

               * `scalePolicy` — политика масштабирования воркеров:

                    * `fixedScale` — фиксированная политика масштабирования.

                       * `count` — количество воркеров.

                    * `autoScale` — автоматическая политика масштабирования.

                       * `minCount` — минимальное количество воркеров.
                       * `maxCount` — максимальное количество воркеров.

                    Укажите один из двух параметров: `fixedScale` либо `autoScale`. 

            * `retryPolicy` — параметры [отказоустойчивого выполнения запросов](../concepts/retry-policy.md).

               * `policy` – способ повторного выполнения запросов. Возможные значения:

                  * `TASK` — в рамках запроса повторно выполняется промежуточное задание, вызвавшее сбой воркера.
                  * `QUERY` – повторно выполняются все [этапы запроса](../concepts/index.md#query-execution), в котором произошел сбой воркера.

               * `exchangeManager.additionalProperties` – дополнительные параметры хранилища Exchange Manager в формате `ключ: значение`. Подробнее о параметрах см. в [документации {{ TR }}](https://trino.io/docs/current/admin/fault-tolerant-execution.html#id1).

               * `additionalProperties` – дополнительные параметры в формате `ключ: значение`. Подробнее о параметрах см. в [документации {{ TR }}](https://trino.io/docs/current/admin/fault-tolerant-execution.html#advanced-configuration).

        * `networkSpec` — сетевые настройки:

            * `securityGroupIds` — список идентификаторов групп безопасности.

        * `deletionProtection` — позволяет включить защиту кластера от непреднамеренного удаления. Возможные значения: `true` или `false`.

            Включенная защита от удаления не помешает подключиться к кластеру вручную и удалить его.

        * `serviceAccountId` — идентификатор сервисного аккаунта.
        * `logging` — параметры логирования:

            * `enabled` — позволяет включить логирование. Логи, сгенерированные компонентами {{ TR }}, будут отправляться в {{ cloud-logging-full-name }}. Возможные значения: `true` или `false`.
            * `minLevel` — минимальный уровень логирования. Возможные значения: `TRACE`, `DEBUG`, `INFO`, `WARN`, `ERROR` и `FATAL`.
            * `folderId` — идентификатор каталога. Логи будут записываться в [лог-группу](../../logging/concepts/log-group.md) по умолчанию для этого каталога.
            * `logGroupId` — идентификатор пользовательской лог-группы. Логи будут записываться в нее.

            Укажите один из двух параметров: `folderId` либо `logGroupId`.

    1. Воспользуйтесь методом [Cluster.update](../api-ref/Cluster/update.md) и выполните запрос, например, с помощью {{ api-examples.rest.tool }}:

        ```bash
        curl \
            --request PATCH \
            --header "Authorization: Bearer $IAM_TOKEN" \
            --url 'https://{{ api-host-trino }}/managed-trino/v1/clusters/<идентификатор_кластера>'
            --data '@body.json'
        ```

        Идентификатор кластера можно запросить со [списком кластеров в каталоге](cluster-list.md#list-clusters).

    1. Убедитесь, что запрос был выполнен успешно, изучив [ответ сервера](../api-ref/Cluster/update.md#yandex.cloud.operation.Operation).

- gRPC API {#grpc-api}

    Чтобы изменить настройки кластера:

    1. [Получите IAM-токен для аутентификации в API](../api-ref/authentication.md) и поместите токен в переменную среды окружения:

        {% include [api-auth-token](../../_includes/mdb/api-auth-token.md) %}

    1. {% include [grpc-api-setup-repo](../../_includes/mdb/grpc-api-setup-repo.md) %}

    1. Создайте файл `body.json` и добавьте в него следующее содержимое:

       {% note info %}

       В примере приведены не все доступные параметры. Список всех параметров см. в [документации по API](../api-ref/grpc/Cluster/update.md#yandex.cloud.trino.v1.UpdateClusterRequest).

       {% endnote %}

        ```json
        {
          "cluster_id": "<идентификатор_кластера>",
          "update_mask": "<список_изменяемых_параметров>",
          "name": "<имя_кластера>",
          "description": "<описание_кластера>",
          "labels": { <список_меток> },
          "trino": {
            "coordinator_config": {
              "resources": {
                "resource_preset_id": "<идентификатор_ресурсов>"
              }
            },
            "worker_config": {
              "resources": {
                "resource_preset_id": "<идентификатор_ресурсов>"
              },
              "scale_policy": {
                "auto_scale": {
                  "min_count": "<минимальное_количество_экземпляров>",
                  "max_count": "<максимальное_количество_экземпляров>"
                }
              }
            },
            "retry_policy": {
              "policy": "<тип_объекта_для_перезапроса>",
              "exchange_manager": {
                "storage": {
                  "service_s3": ""
                },
                "additional_properties": {<дополнительные_параметры_хранилища>}
              },
              "additional_properties": {<дополнительные_параметры_перезапросов>}
            }
          },
          "network_spec": {
            "security_group_ids": [ <список_идентификаторов_групп_безопасности> ]
          },
          "deletion_protection": "<защита_от_удаления>",
          "service_account_id": "<идентификатор_сервисного_аккаунта>",
          "logging": {
            "enabled": "<использование_логирования>",
            "folder_id": "<идентификатор_каталога>",
            "min_level": "<уровень_логирования>"
          }
        }
        ```

        Где:

        * `cluster_id` — идентификатор кластера. Его можно запросить со [списком кластеров в каталоге](cluster-list.md#list-clusters).
        * `update_mask` — перечень изменяемых параметров в виде массива строк `paths[]`.

            {% cut "Формат перечисления настроек" %}

            ```yaml
            "update_mask": {
                "paths": [
                    "<настройка_1>",
                    "<настройка_2>",
                    ...
                    "<настройка_N>"
                ]
            }
            ```

            {% endcut %}

            {% note warning %}

            При изменении кластера будут переопределены все параметры изменяемого объекта, которые не были явно переданы в запросе, на значения по умолчанию. Чтобы избежать этого, перечислите настройки, которые вы хотите изменить, в параметре `update_mask`.

            {% endnote %}

        * `name` — имя кластера.
        * `description` — описание кластера.
        * `labels` — список меток. Метки задаются в формате `"<ключ>": "<значение>"`.
        * `trino` — конфигурация [компонентов](../concepts/index.md#cluster-architecture) кластера {{ TR }}.

            * `coordinator_config` — конфигурация координатора.

               * `resources.resource_preset_id` — идентификатор вычислительных ресурсов координатора. Возможные значения:

                    * `c4-m16` — 4 vCPU, 16 ГБ RAM.
                    * `c8-m32` — 8 vCPU, 32 ГБ RAM.

            * `worker_config` — конфигурация воркера.

               * `resources.resource_preset_id` — идентификатор вычислительных ресурсов воркера. Возможные значения:

                    * `c4-m16` — 4 vCPU, 16 ГБ RAM.
                    * `c8-m32` — 8 vCPU, 32 ГБ RAM.

               * `scale_policy` — политика масштабирования воркеров:

                    * `fixed_scale` — фиксированная политика масштабирования.

                       * `count` — количество воркеров.

                    * `auto_scale` — автоматическая политика масштабирования.

                       * `min_count` — минимальное количество воркеров.
                       * `max_count` — максимальное количество воркеров.

                    Укажите один из двух параметров: `fixed_scale` либо `auto_scale`. 

            * `retry_policy` – параметры [отказоустойчивого выполнения запросов](../concepts/retry-policy.md).

               * `policy` – способ повторного выполнения запросов. Возможные значения:

                  * `TASK` — в рамках запроса повторно выполняется промежуточное задание, вызвавшее сбой воркера.
                  * `QUERY` – повторно выполняются все [этапы запроса](../concepts/index.md#query-execution), в котором произошел сбой воркера.

               * `exchange_manager.additional_properties` – дополнительные параметры хранилища Exchange Manager в формате `ключ: значение`. Подробнее о параметрах см. в [документации {{ TR }}](https://trino.io/docs/current/admin/fault-tolerant-execution.html#id1).

               * `additional_properties` – дополнительные параметры в формате `ключ: значение`. Подробнее о параметрах см. в [документации {{ TR }}](https://trino.io/docs/current/admin/fault-tolerant-execution.html#advanced-configuration).

        * `network_spec` — сетевые настройки:

            * `security_group_ids` — список идентификаторов групп безопасности.

        * `deletion_protection` — позволяет включить защиту кластера от непреднамеренного удаления. Возможные значения: `true` или `false`.

            Включенная защита от удаления не помешает подключиться к кластеру вручную и удалить его.

        * `service_account_id` — идентификатор сервисного аккаунта.
        * `logging` — параметры логирования:

            * `enabled` — позволяет включить логирование. Логи, сгенерированные компонентами {{ TR }}, будут отправляться в {{ cloud-logging-full-name }}. Возможные значения: `true` или `false`.
            * `min_level` — минимальный уровень логирования. Возможные значения: `TRACE`, `DEBUG`, `INFO`, `WARN`, `ERROR` и `FATAL`.
            * `folder_id` — идентификатор каталога. Логи будут записываться в [лог-группу](../../logging/concepts/log-group.md) по умолчанию для этого каталога.
            * `log_group_id` — идентификатор пользовательской лог-группы. Логи будут записываться в нее.

            Укажите один из двух параметров: `folder_id` либо `log_group_id`.

    1. Воспользуйтесь вызовом [ClusterService/Update](../api-ref/grpc/Cluster/update.md) и выполните запрос, например, с помощью {{ api-examples.grpc.tool }}:

        ```bash
        grpcurl \
            -format json \
            -import-path ~/cloudapi/ \
            -import-path ~/cloudapi/third_party/googleapis/ \
            -proto ~/cloudapi/yandex/cloud/trino/v1/cluster_service.proto \
            -rpc-header "Authorization: Bearer $IAM_TOKEN" \
            -d @ \
            {{ api-host-trino }}:{{ port-https }} \
            yandex.cloud.trino.v1.ClusterService.Update \
            < body.json
        ```

    1. Убедитесь, что запрос был выполнен успешно, изучив [ответ сервера](../api-ref/grpc/Cluster/create.md#yandex.cloud.operation.Operation).

{% endlist %}
