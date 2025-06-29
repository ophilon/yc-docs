---
title: Резервные копии {{ CH }}
description: '{{ mch-short-name }} обеспечивает автоматическое и ручное резервное копирование баз данных. Резервные копии занимают место в объеме хранилища, выделенном для кластера. Резервные копии автоматически создаются раз в день.'
keywords:
  - бэкап
  - бекап
  - backup
  - резервное копирование
  - резервное копирование {{ CH }}
  - backup {{ CH }}
  - '{{ CH }}'
---

# Резервные копии в {{ mch-name }}

{{ mch-short-name }} обеспечивает автоматическое и ручное резервное копирование баз данных.

Резервная копия автоматически создается раз в день. Отключить автоматическое создание резервных копий и изменить их срок хранения нельзя.

Чтобы восстановить кластер из резервной копии, [следуйте инструкциям](../operations/cluster-backups.md#restore).

## Создание резервной копии {#size}

Резервные копии могут быть созданы автоматически и вручную, в обоих случаях используется инкрементальная схема:

* При создании очередной резервной копии [куски данных]({{ ch.docs }}/engines/table-engines/mergetree-family/mergetree/#mergetree-data-storage) проверяются на уникальность.
* Если идентичные [куски данных]({{ ch.docs }}/engines/table-engines/mergetree-family/mergetree/#mergetree-data-storage) уже есть в одной из существующих резервных копий и они не старше {{ mch-dedup-retention }} дней, то они не дублируются. Для холодных данных [гибридного хранилища](storage.md#hybrid-storage-features) этот срок составляет {{ mch-backup-retention }} дней.

Резервная копия создается на весь кластер и содержит все [шарды](./sharding.md). Восстановить из резервной копии можно как отдельные шарды, так и весь кластер целиком.

{% note info %}

До 1 апреля 2025 года резервные копии создавались отдельно для каждого шарда кластера. Для таких резервных копий можно восстановить:

* одну или несколько резервных копий шардов в отдельный кластер;
* весь кластер целиком, указав резервные копии всех шардов кластера.

Восстановить несколько шардов в один кластер можно, если резервные копии создавались для одного и того же кластера.

{% endnote %}

В резервной копии хранятся данные только для движков семейства `MergeTree`. Для остальных движков хранятся только схемы таблиц. Подробнее про движки см. в [документации {{ CH }}]({{ ch.docs }}/engines/table-engines/).

Для создания резервной копии используется случайный хост-реплика. Поэтому, если между хостами кластера нет консистентности данных, то восстановление из резервной копии не гарантирует полного восстановления данных. Например, такое может произойти, если:

* [Таблицы реплицируются](replication.md#replicated-tables) не на все хосты шарда.
* Таблицы не реплицируются и размещены на части хостов шарда.

При [создании](../operations/cluster-create.md) или [изменении](../operations/update.md#change-additional-settings) кластера можно задать промежуток времени, в течение которого начинается резервное копирование. По умолчанию — `22:00 - 23:00` UTC (Coordinated Universal Time).

Резервные копии создаются только на работающих кластерах. Если вы используете кластер {{ mch-short-name }} не круглосуточно, проверьте [настройки времени начала резервного копирования](../operations/update.md#change-additional-settings).

О том, как вручную создать резервную копию, читайте в разделе [Управление резервными копиями](../operations/cluster-backups.md).

## Хранение резервной копии {#storage}

* Резервные копии данных [локального](storage.md) и [сетевых](storage.md) хранилищ содержатся в отдельном бакете {{ objstorage-name }} и не занимают место в хранилище кластера. При этом, если в кластере есть N свободных гигабайт места, то хранение первых N гигабайт резервных копий не тарифицируется.

* Резервные копии холодных данных [гибридного хранилища](storage.md#hybrid-storage-features) хранятся в том же бакете {{ objstorage-name }}, что и сами данные. Объем, который занимают резервные копии, учитывается при расчете стоимости использования {{ objstorage-name }} так же, как и объем самих данных.

    Подробнее см. в разделе [Правила тарификации](../pricing.md#rules-storage).

* Резервные копии хранятся в виде бинарных файлов и шифруются с помощью [GPG](https://ru.wikipedia.org/wiki/GnuPG). У каждого кластера свои ключи шифрования.

* Срок хранения резервных копий существующего кластера зависит от способа их создания:

    * Автоматические резервные копии по умолчанию хранятся {{ mch-backup-retention }} дней. При [создании](../operations/cluster-create.md) кластера или [изменении](../operations/update.md#change-additional-settings) его настроек можно задать другой срок хранения в диапазоне от 3 до 60 дней.

    * Резервные копии, созданные вручную, хранятся бессрочно.

* После удаления кластера все его резервные копии хранятся {{ mch-backup-retention }} дней.

* {% include [no-quotes-no-limits](../../_includes/mdb/backups/no-quotes-no-limits.md) %}

## Проверка восстановления из резервной копии {#capabilities}

Для проверки возможностей резервного копирования [восстановите кластер из резервной копии](../operations/cluster-backups.md) и проверьте целостность ваших данных.

## Удаление резервной копии {#deletion}

Удалить можно только резервные копии, созданные вручную. Чтобы удалить такую резервную копию, [следуйте инструкции](../operations/cluster-backups.md#delete-backup).

## Примеры использования {#examples}

* [{#T}](../operations/cluster-backups.md)

{% include [clickhouse-disclaimer](../../_includes/clickhouse-disclaimer.md) %}