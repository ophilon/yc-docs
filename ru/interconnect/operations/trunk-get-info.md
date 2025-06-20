---
title: Как получить информацию о транковых подключениях в {{ interconnect-name }}
description: Следуя этой инструкции, вы сможете получить информацию о транковых подключениях в {{ interconnect-name }}.
---

# Получить информацию о транковых подключениях

{% note info %}

Для выполнения операции необходима роль [cic.viewer](../security/index.md#cic-viewer).

{% endnote %}

{% list tabs group=instructions %}

- CLI {#cli}

  1. Посмотрите описание команды CLI для получения информации о [транковых подключениях](../concepts/trunk.md):

      ```bash
      yc cic trunk get --help
      ```

  1. Получите список транковых подключений в указанном каталоге:

      ```bash
      yc cic trunk list --folder-id b1gqf2hjizv2jw******
      ```

      Результат:

            
      ```text
      +----------------------+--------------------+----------------------------+------------------+----------+
      |          ID          |        NAME        | POINT OF PRESENCE ID (POP) | TRANSCEIVER TYPE | CAPACITY |
      +----------------------+--------------------+----------------------------+------------------+----------+
      | cf3td**********nufvr | customer-name-m9   | ru-msk-m9-0                | 10GBASE-LR       | 1 GBPS   |
      | euuvd**********jl5sh | customer-name-ost  | ru-msk-ost-0               | 10GBASE-LR       | 1 GBPS   |
      +----------------------+--------------------+----------------------------+------------------+----------+
      ```




  1. Получите информацию о транковом подключении, указав его идентификатор, полученный на предыдущем шаге:

      ```bash
      # yc cic trunk get <идентификатор-транка>
      yc cic trunk get cf3td**********nufvr
      ```

      Результат:

      
      ```yml
      - id: cf3td**********nufvr
        name: customer-name-m9
        description: Trunk M9
        cloud_id: b1g7a**********kd23p
        folder_id: b1gqf**********jiz2w
        region_id: {{ region-id }}
        created_at: "2025-03-25T10:54:46Z"
        single_port_direct_joint:
          transceiver_type: TRANSCEIVER_TYPE_10GBASE_LR
          port_name: 25GE1/0/12
        point_of_presence_id: ru-msk-m9-0
        capacity: CAPACITY_1_GBPS
        status: ACTIVE
      ```



      где,
      * `id` — идентификатор транкового подключения.
      * `name` — название транкового подключения.
      * `description` — описание транкового подключения.
      * `cloud_id` — идентификатор облака, в каталоге которого было создано транковое подключение.
      * `folder_id` — идентификатор облачного каталога в котором было создано транковое подключение.
      * `region_id` — регион облака в котором создано транковое подключение.
      * Тип транкового подключения:
        * `single_port_direct_joint` — прямое транковое подключение:
           * `transceiver_type` — тип используемого [трансивера](../concepts/transceivers.md).
           * `port_name` — номер порта (портов), которые выделены на сетевом устройстве для транкового подключения.
           * `access_device_name` — имя сетевого устройства на котором были выделены порты для транкового подключения.
        * `lag_direct_joint` — агрегированное (LAG) прямое транковое подключение:
           * `transceiver_type` — тип используемого [трансивера](../concepts/transceivers.md).
           * `lag_id` — идентификатор агрегированного подключения.
           * `port_names` — список физических портов в LAG.
        * `partner_joint_info` — транковое подключение через партнера:
           * `partner_id` — идентификатор партнера.
           * `service_key` — сервисный ключ для транкового подключения через партнера.
      * `point_of_presence_id` — идентификатор [точки присутствия](../concepts/pops.md).
      * `capacity` — величина [пакета трафика](../concepts/capacity.md) для данного транкового подключения. 
      * `status` — состояние транкового подключения. Целевое состояние — `ACTIVE`.
      * `created_at` — дата и время создания транкового подключения.

{% endlist %}

