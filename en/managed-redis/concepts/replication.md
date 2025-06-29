---
title: '{{ VLK }} replication and fault tolerance'
description: '{{ mrd-name }} uses native {{ VLK }} replication and provides high availability of cluster data using {{ VLK }} Sentinel.'
keywords:
  - Valkey replication
  - Valkey
  - Valkey DBMS
---

# Replication and fault tolerance

{{ mrd-name }} uses native {{ VLK }} replication and provides high availability of cluster data using [rdsync](https://github.com/yandex/rdsync), a host status management agent.

## Replication {#replication}

{{ mrd-name }} clusters use asynchronous replication, i.e., the result of a write request is committed to the master host, which then forwards the data to the cluster replicas. The replication process does not affect the master availability in any way, but it can make replicas temporarily unavailable (for up to a few seconds for large databases) when loading new data into memory.

Since the replication is asynchronous, the data on replicas may be out of date: while a replica is processing updates from the master, it continues sending the existing data in response to requests, as the [replica-serve-stale-data](https://github.com/valkey-io/valkey/blob/unstable/valkey.conf) flag is set to `yes`.

Due to limited resources, **b1**, **b2**, and **b3** class hosts are not replicated.

For more information about how replication works in {{ VLK }}, read the [relevant documentation](https://valkey.io/topics/replication/).

## Fault tolerance {#availability}

To ensure fault tolerance, [rdsync](https://github.com/yandex/rdsync), a host status management agent from Yandex, was integrated into the {{ mrd-name }} architecture.

Host status is stored in the distributed configuration management system. If the connection to the DCS (distributed configuration store, e.g., {{ ZK }}, etcd, or Consul) is lost, the agent switches the host to [protected mode](https://valkey.io/topics/security/#protected-mode) and terminates client connections. When selecting a new master, if the replica host with the highest priority requires full data resync, the agent stops the replication process and selects a host with the least lag behind the master.

Thanks to the `rdsync` agent in {{ mrd-name }} cluster:

* Configurations that consist of an even number of hosts (for non-sharded clusters) or one or two shards (for sharded clusters) are fault-tolerant.

* The [client request](https://valkey.io/topics/sentinel-clients/) for the name of the host available for writes is processed in alignment with the `rdsync` agent and provides up-to-date information to clients, because the statuses of all hosts are known.

* The risk of losing data can be reduced by using the `WAIT` command with `N/2` available replicas, where `N` is the number of cluster hosts.

Sharded clusters with the **local-ssd** disk type and only one host per shard are not considered fault-tolerant. You cannot create such a cluster.

### Assigning a different host as a master if the primary master fails {#master-failover}

If the master host fails, a host with the least lag behind the master will become a new master.

You can influence master selection in a {{ VLK }} cluster by [configuring priorities](../operations/hosts.md#update) for cluster hosts. The host with the highest priority will become a new master. If the replica host with the highest priority requires full data resync, the priority value will be ignored and a host with the least lag behind the master will become a new master.

You can set host priority:

* When [creating a cluster](../operations/cluster-create.md) or [a host in a cluster](../operations/hosts.md#add).
* When [changing the host settings](../operations/hosts.md#update).

Minimum value (lowest priority): `0`. A host with such priority value can become a master only if there are no other hosts suitable for the role. Default priority value: `100`. You can specify a value higher than `100`.

A master host can be changed either automatically, as a result of a failure, or [manually](../operations/failover.md). Manual master switching is available both for [sharded](./sharding.md#failover) and unsharded clusters.

## Persistence {#persistence}

{{ mrd-name }} clusters use data persistence presets. You can disable persistence if needed to improve server throughput, as the DBMS will stop writing updates to disk.

{% note warning %}

Disable persistence only if data integrity is not critical for your application, e.g., when using {{ mrd-name }} as a cache. This is because, in this case, the most recent data captured in {{ VLK }} will only be stored in RAM and may be lost if a server crashes.

{% endnote %}

### Persistence settings {#persistence-on}

By default, cluster persistence is enabled and uses the following {{ VLK }} settings:

* **save ""**{#setting-save-rdb}

  Regular RDB file saving is disabled. [AOF](#setting-appendonly) mode is used instead.

* **appendonly yes**{#setting-appendonly}

  AOF (Append Only File) mode is enabled. In this mode, {{ VLK }} logs every write operation without changing already written data.

* **no-appendfsync-on-rewrite yes**{#setting-no-appendfsync}

  The AOF `fsync` policy being set to `everysec`, the AOF log's background save (`BGSAVE`) or rewrite (`BGREWRITEAOF`) process performs many disk I/O operations. {{ VLK }} may block calling `fsync()` for too long in some Linux configurations.

  The setting prevents calling `fsync()` within the main system process when running `BGSAVE` or `BGREWRITEAOF`.

  When you run `BGREWRITEAOF`, `fsync()` is in progress. {{ VLK }} writes the shortest sequence of commands needed to rebuild the current dataset in memory. You can manage data size using the [aof-rewrite-incremental-fsync](#setting-rewrite-incremental) setting.

* **auto-aof-rewrite-percentage 100**{#setting-rewrite-percentage}

  The AOF log size must exceed 100% for the AOF file to be automatically rewritten. This setting depends on the log file's [**auto-aof-rewrite-min-size**](#setting-rewrite-size) setting.

* **auto-aof-rewrite-min-size 64mb**{#setting-rewrite-size}

  Auto-aof-rewrite-min-size 64mb

* **aof-load-truncated yes**{#setting-load-truncated}

  Allows loading a truncated AOF file if the system crashes. The log notifies the user on loading the truncated file.

* **aof-rewrite-incremental-fsync yes**{#setting-rewrite-incremental}

  Enables AOF file synchronization every time 32 MB of data is generated.

* **aof-use-rdb-preamble yes**{#setting-rdb-preamble}

  Enables using the RDB file as a prefix to the AOF file when rewriting or restoring it.

For more information on {{ VLK }} persistence mechanisms, refer to the [DBMS documentation](https://valkey.io/topics/persistence) and the [valkey.conf](https://github.com/valkey-io/valkey/blob/unstable/valkey.conf) config file description.

### Disabling persistence {#persistence-off}

With persistence disabled, the following {{ VLK }} settings take effect:

* **save ""**

  Regular RDB file saving is disabled.

* **appendonly no**

  AOF (Append Only File) mode is disabled.
