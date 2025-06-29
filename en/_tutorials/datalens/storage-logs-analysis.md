# Analyzing {{ objstorage-short-name }} logs in {{ datalens-short-name }}


For a {{ objstorage-full-name }} bucket, you can enable [action logging](../../storage/concepts/server-logs.md). The logs store info on operations involving a [bucket](../../storage/concepts/bucket.md) and the [objects](../../storage/concepts/object.md) in it. For example, analyzing bucket logs can help understand what caused a steep load increase or get the overall picture of traffic distribution.

You can create visualizations for your analysis using [{{ datalens-full-name }}](../../datalens/). You must transfer previously saved logs to the {{ CH }} database, which will be used as a source for {{ datalens-short-name }}.

To analyze logs and present the results in interactive charts:

1. [Get your cloud ready](#before-you-begin).
1. [Create a bucket for storing logs](#create-bucket).
1. [Enable log export](#logs-export).
1. [Get the data source ready](#prepare-origin).
1. [Create a connection in {{ datalens-short-name }}](#create-connection).
1. [Create a dataset in {{ datalens-short-name }}](#create-dataset).
1. [Create charts in {{ datalens-short-name }}](#create-charts).
1. [Create a dashboard in {{ datalens-short-name }}](#create-dashboard).

If you no longer need the resources you created, [delete them](#clear-out).

## Get your cloud ready {#before-you-begin}

{% include [before-you-begin](../_tutorials_includes/before-you-begin.md) %}


### Required paid resources {#paid-resources}

The cost includes:

* Fee for data storage in {{ objstorage-short-name }}, data operations, and outbound traffic (see [{{ objstorage-short-name }} pricing](../../storage/pricing.md)).
* Fee for a continuously running {{ mch-name }} cluster (see [{{ mch-name }} pricing](../../managed-clickhouse/pricing.md)).


## Create a bucket for storing logs {#create-bucket}

{% list tabs group=instructions %}

- Management console {#console}

  1. In the [management console]({{ link-console-main }}), select the folder where you want to create a bucket.
  1. From the list of services, select **{{ ui-key.yacloud.iam.folder.dashboard.label_storage }}**.
  1. Click **{{ ui-key.yacloud.storage.buckets.button_create }}**.
  1. In the **{{ ui-key.yacloud.storage.bucket.settings.field_name }}** field, enter a name for the bucket.
  1. In the **{{ ui-key.yacloud.storage.bucket.settings.field_access-read }}** and **{{ ui-key.yacloud.storage.bucket.settings.field_access-list }}** fields, select **{{ ui-key.yacloud.storage.bucket.settings.access_value_private }}**.
  1. Click **{{ ui-key.yacloud.storage.buckets.create.button_create }}**.

- AWS CLI {#cli}
  
  1. If you do not have the AWS CLI yet, [install and configure it](../../storage/tools/aws-cli.md).
  1. Create a bucket:
  
     ```bash
     aws --endpoint-url https://{{ s3-storage-host }} \
       s3 mb s3://<bucket_name>
     ```

     Result:
     
     ```text
     make_bucket: <bucket_name>
     ```

- {{ TF }} {#tf}

  {% include [terraform-role](../../_includes/storage/terraform-role.md) %}

  {% include [terraform-definition](../_tutorials_includes/terraform-definition.md) %}

  {% include [terraform-install](../../_includes/terraform-install.md) %}

  1. Describe the settings for creating a service account and access key in the configuration file:

     {% include [terraform-sa-key](../../_includes/storage/terraform-sa-key.md) %}

  1. Add the bucket properties to the configuration file:
  
     ```hcl
     resource "yandex_storage_bucket" "bucket-logs" {
       access_key = yandex_iam_service_account_static_access_key.sa-static-key.access_key
       secret_key = yandex_iam_service_account_static_access_key.sa-static-key.secret_key
       bucket     = "<bucket_name>"
     }
     ```

     For more information about the `yandex_storage_bucket` resource, see this [{{ TF }} overview article]({{ tf-provider-resources-link }}/storage_bucket).
     
  1. Make sure the settings are correct.

     {% include [terraform-validate](../../_includes/mdb/terraform/validate.md) %}

  1. Create a bucket.

     {% include [terraform-apply](../../_includes/mdb/terraform/apply.md) %}

- API {#api}

  Use the [create](../../storage/s3/api-ref/bucket/create.md) REST API method.
       
{% endlist %}

## Enable log export {#logs-export}

{% list tabs group=instructions %}

- Management console {#console}

  1. In the [management console]({{ link-console-main }}), select the bucket you want to enable logging for.
  1. In the left-hand panel, select **{{ ui-key.yacloud.storage.bucket.switch_settings }}**.
  1. Open the **{{ ui-key.yacloud.storage.bucket.switch_server-logs }}** tab.  
  1. Enable **{{ ui-key.yacloud.storage.server-logs.label_server-logs }}**.
  1. Select **{{ ui-key.yacloud.storage.server-logs.label_target-bucket }}**.
  1. In the **{{ ui-key.yacloud.storage.server-logs.label_prefix }}** field, specify the `s3-logs/` prefix.
  1. Click **{{ ui-key.yacloud.common.save }}**.

- AWS CLI {#cli}

  1. Create a file named `log-config.json` with the following contents:

     ```json
     {
      "LoggingEnabled": {
         "TargetBucket": "<bucket_name>",
         "TargetPrefix": "s3-logs/"
      }
     }
     ```

  1. Run this command:
     
     ```bash
     aws s3api put-bucket-logging \
       --endpoint-url https://{{ s3-storage-host }} \
       --bucket <bucket_name> \
       --bucket-logging-status file://log-config.json
     ```

     Where `--bucket` is the name of the bucket to enable action logging for.

- {{ TF }} {#tf}
  
  To enable logging for a bucket you want to track:

     1. Open the {{ TF }} configuration file and add the `logging` section to the bucket description fragment.

        ```hcl
        resource "yandex_storage_bucket" "bucket-logs" {
          access_key = "<static_key_ID>"
          secret_key = "<secret_key>"
          bucket     = "<name_of_bucket_to_store_logs>"
        }

        resource "yandex_storage_bucket" "bucket" {
          access_key = "<static_key_ID>"
          secret_key = "<secret_key>"
          bucket     = "<source_bucket_name>"
          acl        = "private"

          logging {
            target_bucket = yandex_storage_bucket.bucket-logs.id
            target_prefix = "s3-logs/"
          }
        }
        ```

        Where:
        * `access_key`: Static access key ID.
        * `secret_key`: Secret access key value.
        * `target_bucket`: Reference to the log storage bucket.
        * `target_prefix`: [Key prefix](../../storage/concepts/server-logs.md#key-prefix) for objects with logs.

        For more information about `yandex_storage_bucket` properties in {{ TF }}, see [this {{ TF }} article]({{ tf-provider-resources-link }}/storage_bucket#enable-logging).

        {% include [terraform-validate-plan-apply](../_tutorials_includes/terraform-validate-plan-apply.md) %}

        This will create all the resources you need in the specified folder. You can check the new resources and their settings using the [management console]({{ link-console-main }}).

- API {#api}

  Use the REST API [putBucketLogging](../../storage/s3/api-ref/bucket/putBucketLogging.md) method.

{% endlist %}

## Get the data source ready {#prepare-origin}

### Create a {{ CH }} cluster {#create-ch-cluster}

To create a {{ mch-name }} cluster, you will need the [{{ roles-vpc-user }}](../../vpc/security/index.md#vpc-user) and [{{ roles.mch.editor }} roles or higher](../../managed-clickhouse/security.md#roles-list). To learn more about assigning roles, see [this {{ iam-name }} article](../../iam/operations/roles/grant.md).

{% list tabs group=instructions %}

- Management console {#console}

  1. In the [management console]({{ link-console-main }}), select the folder where you want to create a cluster.
  1. From the list of services, select **{{ ui-key.yacloud.iam.folder.dashboard.label_managed-clickhouse }}**.
  1. In the window that opens, click **{{ ui-key.yacloud.mdb.clusters.button_create }}**.
  1. Specify the {{ CH }} cluster settings:

     1. Under **{{ ui-key.yacloud.mdb.forms.section_base }}**, specify `s3-logs` in the **{{ ui-key.yacloud.mdb.forms.base_field_name }}** field.

     1. Under **{{ ui-key.yacloud.mdb.forms.new_section_resource }}**, select `burstable` in the **{{ ui-key.yacloud.mdb.forms.resource_presets_field-type }}** field.

     1. Under **{{ ui-key.yacloud.mdb.forms.section_host }}**, click ![image](../../_assets/console-icons/pencil.svg) and enable the **{{ ui-key.yacloud.mdb.hosts.dialog.field_public_ip }}** option. Click **{{ ui-key.yacloud.mdb.hosts.dialog.button_choose }}**.

     1. Under **{{ ui-key.yacloud.mdb.forms.section_settings }}**:

        * In the **{{ ui-key.yacloud.mdb.forms.database_field_sql-user-management }}** field, select `{{ ui-key.yacloud.common.disabled }}`.
        * In the **{{ ui-key.yacloud.mdb.forms.database_field_user-login }}** field, specify `user`.
        * In the **{{ ui-key.yacloud.mdb.forms.database_field_user-password }}** field, set a password.
        * In the **{{ ui-key.yacloud.mdb.forms.database_field_name }}** field, specify `s3_data`.

        Memorize the database name.

     1. Under **{{ ui-key.yacloud.mdb.forms.section_service-settings }}**, enable these options:

        * **{{ ui-key.yacloud.mdb.forms.additional-field-datalens }}**.
        * **{{ ui-key.yacloud.mdb.forms.additional-field-websql }}**.

  1. Click **{{ ui-key.yacloud.mdb.forms.button_create }}**.

- {{ yandex-cloud }} CLI {#cli}

  {% include [cli-install](../../_includes/cli-install.md) %}

  {% include [default-catalogue](../../_includes/default-catalogue.md) %}

  To create a cluster:

  1. Check whether the folder has any subnets for the cluster hosts:

     ```bash
     yc vpc subnet list
     ```

     If there are no subnets in the folder, [create the required subnets](../../vpc/operations/subnet-create.md) in {{ vpc-short-name }}.

  1. Specify the cluster properties in the creation command:

     ```bash
     {{ yc-mdb-ch }} cluster create \
        --name s3-logs \
        --environment production \
        --network-name <network_name> \
        --host type=clickhouse,zone-id=<availability_zone>,subnet-id=<subnet_ID> \
        --clickhouse-resource-preset b2.medium \
        --clickhouse-disk-type {{ disk-type-example }} \
        --clickhouse-disk-size 10 \
        --user name=user,password=<user_password> \
        --database name=s3_data \
        --datalens-access=true \
        --websql-access=true
     ```

- {{ TF }} {#tf}

  1. Add descriptions of your cluster, database, and user to the configuration file:

     ```hcl
     resource "yandex_mdb_clickhouse_cluster" "s3-logs" {
       name                = "s3-logs"
       environment         = "PRODUCTION"
       network_id          = yandex_vpc_network.<network_name_in_{{ TF }}>.id

       clickhouse {
         resources {
           resource_preset_id = "b2.medium"
           disk_type_id       = "{{ disk-type-example }}"
           disk_size          = 10
         }
       }

       host {
         type      = "CLICKHOUSE"
         zone      = "<availability_zone>"
         subnet_id = yandex_vpc_subnet.<subnet_name_in_{{ TF }}>.id
       }

       access {
         datalens  = true
         web_sql   = true
       }

       lifecycle {
         ignore_changes = [database, user]
       }
     }

     resource "yandex_mdb_clickhouse_database" "s3-data" {
       cluster_id = yandex_mdb_clickhouse_cluster.s3-logs.id
       name       = "s3_data"
     }

     resource "yandex_mdb_clickhouse_user" "user1" {
       cluster_id = yandex_mdb_clickhouse_cluster.s3-logs.id
       name       = "user"
       password   = "<password>"
       permission {
         database_name = yandex_mdb_clickhouse_database.s3-data.name
       }
     }
     ```

     For more information about the resources you can create with {{ TF }}, see the [provider documentation]({{ tf-provider-mch }}).

  1. Make sure the settings are correct.

     {% include [terraform-validate](../../_includes/mdb/terraform/validate.md) %}

  1. Create a cluster:

     {% include [terraform-apply](../../_includes/mdb/terraform/apply.md) %}

- API {#api}
  
  Use the [create](../../managed-clickhouse/api-ref/Cluster/create.md) REST API method.

{% endlist %}

After creating the cluster, you will be automatically redirected to the **{{ ui-key.yacloud.clickhouse.switch_list }}** page.

Wait until the cluster status switches to `Alive`.

### Change the user settings {#user-settings}

{% list tabs group=instructions %}

- Management console {#console}

  1. Select the `s3-logs` cluster.
  1. Navigate to the **{{ ui-key.yacloud.clickhouse.cluster.switch_users }}** tab.
  1. Click ![image](../../_assets/console-icons/ellipsis.svg) and select **{{ ui-key.yacloud.mdb.cluster.users.button_action-update }}**.
  1. Click **{{ ui-key.yacloud.mdb.cluster.users.button_advanced-settings }}** → **Settings**.
  1. In the **Date time input format** field, select `best_effort`.
  1. Click **{{ ui-key.yacloud.mdb.cluster.users.popup-button_save }}**.

{% endlist %}

### Create a static key {#create-static-key}

You need a static key to create a table with access to {{ objstorage-name }}. [Create one](../../iam/operations/authentication/manage-access-keys.md#create-access-key) and save its ID and secret part.

### Create a table in the database {#create-table}

{% list tabs group=instructions %}

- Management console {#console}

  1. Select the `s3-logs` cluster.
  1. Navigate to the **SQL** tab.
  1. In the **{{ ui-key.yacloud.clickhouse.cluster.explore.label_password }}** field, enter the password.
  1. Click **{{ ui-key.yacloud.clickhouse.cluster.explore.button_submit-creds }}**.
  1. In the window on the right, write this SQL query:

     ```sql
     CREATE TABLE s3_data.s3logs
     (
        bucket String,              -- Bucket name.
        bytes_received Int64,       -- Request size in bytes.
        bytes_send Int64,           -- Response size in bytes.
        handler String,             -- Request method in this format: REST.<HTTP method>.<subject>.
        http_referer String,        -- Request source URL.
        ip String,                  -- User IP address.
        method String,              -- HTTP request method.
        object_key String,          -- Object key in URL-encoded format.
        protocol String,            -- Data transfer protocol version.
        range String,               -- HTTP header defining the byte range to load from the object.
        requester String,           -- User ID.
        request_args String,        -- Arguments of the URL request.
        request_id String,          -- Request ID.
        request_path String,        -- Full request path.
        request_time Int64,         -- Request processing time in milliseconds.
        scheme String,              -- Data transfer protocol type.
                                    -- The possible values are as follows:
                                    -- * http: Application layer protocol.
                                    -- * https: Application layer protocol with encryption support.
        ssl_protocol String,        -- Security protocol.
        status Int64,               -- HTTP response code.
        storage_class String,       -- Object storage class.
        timestamp DateTime,         -- Date and time of the bucket operation in the YYYY-MM-DDTHH:MM:SSZ format.
        user_agent String,          -- Client app (user agent) that runs the request.
        version_id String,          -- Object version.
        vhost String                -- Virtual host of the request.
                                    -- The possible values are as follows:
                                    -- * {{ s3-storage-host }}.
                                    -- * <bucket_name>.{{ s3-storage-host }}.
                                    -- * {{ s3-web-host }}.
                                    -- * <bucket_name>.{{ s3-web-host }}.
     )
     ENGINE = S3(
           'https://{{ s3-storage-host }}/<bucket_name>/s3-logs/*',
           '<key_ID>',
           '<secret_key>',
           'JSONEachRow'
        )
     SETTINGS date_time_input_format='best_effort';
     ```

  1. Click **{{ ui-key.yacloud.clickhouse.cluster.explore.button_execute }}**.

{% endlist %}

## Create a connection in {{ datalens-short-name }} {#create-connection}

{% list tabs group=instructions %}

- Management console {#console}

  1. Select the `s3-logs` cluster.
  1. Navigate to the **{{ ui-key.yacloud.clickhouse.cluster.switch_datalens }}** tab.
  1. In the window that opens, click **{{ ui-key.yacloud.mdb.datalens.button-action_new-connection }}**.
  1. Fill in the connection settings:

     1. Add a connection name: `s3-logs-con`.
     1. In the **Cluster** field, select `s3-logs`.
     1. In the **Host name** field, select the {{ CH }} host from the drop-down list. 
     1. Enter the DB username and password.

  1. Click **Confirm connection**.
  1. After checking the connection, click **Create connection**.
  1. In the window that opens, enter a name for the connection and click **Create**.

{% endlist %}

## Create a dataset in {{ datalens-short-name }} {#create-dataset}

1. Click **Create dataset**.
1. In the new dataset, move the `s3_data.s3logs` table to the workspace.
1. Navigate to the **Fields** tab.
1. Click ![image](../../_assets/console-icons/plus.svg)**Add field**.
1. Create a calculated field with the file type:
   
   * Field name: `object_type`
   * Formula: `SPLIT([object_key], '.', -1)`

1. Click **Create**.
1. In the top-right corner, click **Save**.
1. Enter the `s3-dataset` name for the dataset and click **Create**.
1. Once the dataset is saved, click **Create chart** in the top-right corner.

## Create charts in {{ datalens-short-name }} {#create-charts}

### Create the first chart {#create-pie-chart}

To visualize the number of requests to a bucket via different methods, create a pie chart:

1. Select `Pie chart` as the visualization type.
1. Drag the `method` field from the **Dimensions** section to the **Colors** section.
1. Drag the `request_id` field from the **Dimensions** section to the **Measures** section.
1. In the top-right corner, click **Save**.
1. In the window that opens, enter the `S3 - Method pie` name for the new chart and click **Save**.

### Create the second chart {#create-column-chart}

To visualize the number of requests ratio by object type, create a bar chart:

1. Copy the chart from the previous step:

   1. In the top-right corner, click the check mark next to the **Save** button.
   1. Click **Save as**.
   1. In the window that opens, enter the `S3 - Object type bars` name for the new chart and click **Save**.

1. Select **Bar chart** as the visualization type. The `method` and `request_id` fields will automatically appear in the **X** and **Y** sections, respectively.
1. Delete the `method` field from the **X** section and drag the `object_type` field there.
1. In the top-right corner, click **Save**.

### Create the third chart {#create-column-chart-2}

To visualize the distribution of outbound traffic by day, create a bar chart:

1. Copy the chart from the previous step:

   1. In the top-right corner, click the check mark next to the **Save** button.
   1. Click **Save as**.
   1. In the window that opens, enter the `S3 - Traffic generated by days` name for the new chart and click **Save**.

1. Drag the `object_type` field from the **X** section to the **Filters** section.
1. In the window that opens, select the types of objects to display in the chart and click **Apply filters**.
1. Drag the `timestamp` field from the **Dimensions** section to the **X** section.
1. Delete the `request_id` field from the **Y** section and drag the `bytes_send` field there.
1. In the top-right corner, click **Save**.

## Create a dashboard in {{ datalens-short-name }} and add charts to it {#create-dashboard}

1. Go to the {{ datalens-short-name }} [home page]({{ link-datalens-main }}).
1. Click **Create dashboard**.
1. Enter the `S3 Logs Analysis` name for the dashboard and click **Create**.
1. In the top-right corner, click **Add** and select `Chart`.
1. In the **Chart** field, click **Select** and select the `S3 - Method pie` chart from the list.
1. Click **Add**. You will now see the chart on the dashboard.
1. Repeat the previous steps for the `S3 - Object type bars` and `S3 - Traffic generated by days` charts.

## How to delete the resources you created {#clear-out}

Delete the resources you no longer need to avoid paying for them:

* [Delete the bucket](../../storage/operations/buckets/delete.md).
* [Delete the `s3-logs` cluster](../../managed-clickhouse/operations/cluster-delete.md).
