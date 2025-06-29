# Interactive debugging of {{ sf-full-name }} functions


In this tutorial, you will set up a system to interactively debug [functions](../../functions/concepts/function.md) in {{ sf-full-name }} by redirecting requests to a local server. For more information about this solution, see the [yc-serverless-live-debug](https://github.com/yandex-cloud/yc-serverless-live-debug) repository.

To set up the interactive function debugging system:

1. [Get your cloud ready](#prepare-cloud).
1. [Install the required utilities](#install-utilities).
1. [Create a cloud administrator service account](#create-account).
1. [Deploy your resources](#create-resources).
1. [Run the debugging service](#run-client).

If you no longer need the resources you created, [delete them](#clear-out).
 
## Get your cloud ready {#prepare-cloud}

{% include [before-you-begin](../../_tutorials/_tutorials_includes/before-you-begin.md) %}

### Required paid resources {#paid-resources}

The infrastructure support costs include:

* Fee for function invocations and computing resources allocated to run the functions (see [{{ sf-full-name }} pricing](../../functions/pricing.md)).
* Fee for the number of requests to the API gateway (see [{{ api-gw-full-name }} pricing](../../api-gateway/pricing.md)).
* Fee for {{ ydb-short-name }} operations and data storage (see [{{ ydb-full-name }} pricing](../../ydb/pricing/serverless.md)).
* Fee for logging operations and log storage (see [{{ cloud-logging-full-name }} pricing](../../logging/pricing.md)).

## Install the required tools {#install-utilities}

1. [Install {{ TF }}](../../tutorials/infrastructure-management/terraform-quickstart.md#install-terraform).
1. Create a folder named `live-debug-test` and open it:

    ```
    mkdir live-debug-test
    cd live-debug-test
    ```

1. Install the `yc-serverless-live-debug` package:

    ```
    npm i -D @yandex-cloud/serverless-live-debug
    ```

## Create a cloud administrator service account {#create-account}

1. Create a [service account](../../iam/concepts/users/service-accounts.md):
   
    {% list tabs group=instructions %}

    - Management console {#console}

      1. In the [management console]({{ link-console-main }}), select the folder where you want to create a service account.
      1. In the list of services, select **{{ ui-key.yacloud.iam.folder.dashboard.label_iam }}**.
      1. Click **{{ ui-key.yacloud.iam.folder.service-accounts.button_add }}**.
      1. Enter a name for the service account, e.g., `sa-live-debug`.

          The naming requirements are as follows:

          {% include [name-format](../../_includes/name-format.md) %}

      1. Click **{{ ui-key.yacloud.iam.folder.service-account.popup-robot_button_add }}**.

    - CLI {#cli}

      {% include [cli-install](../../_includes/cli-install.md) %}

      {% include [default-catalogue](../../_includes/default-catalogue.md) %}

      To create a service account, run the following command:

      ```bash
      yc iam service-account create --name sa-live-debug
      ```

      Where `name` is the service account name in the following format:

      {% include [name-format](../../_includes/name-format.md) %}

      Result:

      ```text
      id: ajehr0to1g8b********
      folder_id: b1gv87ssvu49********
      created_at: "2023-03-04T09:03:11.665153755Z"
      name: sa-live-debug
      ```

    - {{ TF }} {#tf}

      {% include [terraform-install](../../_includes/terraform-install.md) %}

      1. In the configuration file, describe the resources you want to create:
    
          ```hcl
          resource "yandex_iam_service_account" "sa" {
            name        = "sa-live-debug"
            description = "<service_account_description>"
            folder_id   = "<folder_ID>"
          }
          ```

          Where:

          * `name`: Service account name. This is a required parameter.
          * `description`: Service account description. This is an optional parameter.
          * `folder_id`: [Folder ID](../../resource-manager/operations/folder/get-id.md). This is an optional parameter. It defaults to the value specified in the provider settings.

          For more information about `yandex_iam_service_account` properties, see [this {{ TF }} article]({{ tf-provider-resources-link }}/iam_service_account).
    
      1. Make sure the configuration files are correct.

          1. In the command line, navigate to the directory where you created the configuration file.
          1. Run a check using this command:

              ```bash
              terraform plan
              ```

          If the configuration description is correct, the terminal will display information about the service account. If the configuration contains any errors, Terraform will point them out. 

      1. Deploy the cloud resources.

          1. If the configuration does not contain any errors, run this command:

              ```bash
              terraform apply
              ```

    - API {#api}

      To create a service account, use the [create](../../iam/api-ref/ServiceAccount/create.md) REST API method for the [ServiceAccount](../../iam/api-ref/ServiceAccount/index.md) resource or the [ServiceAccountService/Create](../../iam/api-ref/grpc/ServiceAccount/create.md) gRPC API call.

    {% endlist %}

1. Assign the service account the `{{ roles-admin }}` [role](../../iam/concepts/access-control/roles.md) for the cloud:

    {% list tabs group=instructions %}

    - Management console {#console}

      1. On the management console [home page]({{ link-console-main }}), select the cloud.
      1. Navigate to the **{{ ui-key.yacloud.common.resource-acl.label_access-bindings }}** tab.
      1. Find the `sa-live-debug` account in the list and click ![image](../../_assets/console-icons/ellipsis.svg).
      1. Click **{{ ui-key.yacloud.common.resource-acl.button_assign-binding }}**.
      1. Click **{{ ui-key.yacloud_components.acl.action.add-role }}** in the window that opens and select the `{{ roles-admin }}` role.
      1. Click **{{ ui-key.yacloud.common.save }}**.

    - CLI {#cli}

      Run this command:

      ```
      yc resource-manager cloud add-access-binding <cloud_ID> \
         --role {{ roles-admin }} \
         --subject serviceAccount:<service_account_ID>
      ```

      Result:
      ```
      done (1s)
      ```

    - {{ TF }} {#tf}

      1. In the configuration file, describe the resources you want to create:

          ```
          resource "yandex_resourcemanager_cloud_iam_member" "{{ roles-admin }}" {
            cloud_id = "<cloud_ID>"
            role     = "{{ roles-admin }}"
            member   = "serviceAccount:<service_account_ID>"
          }
          ```

          Where:

          * `cloud_id`: [Cloud ID](../../resource-manager/operations/cloud/get-id.md). This is a required parameter.
          * `role`: Role being assigned. This is a required parameter.
          * `member`: User or service account getting the role. Specify it as `userAccount:<user_ID>` or `serviceAccount:<service_account_ID>`. This is a required parameter.

          For more information about `yandex_resourcemanager_folder_iam_member` properties, see [this {{ TF }} article]({{ tf-provider-resources-link }}/iam_service_account_iam_member).

      1. Make sure the configuration files are correct.

          1. In the command line, navigate to the directory where you created the configuration file.
          1. Run a check using this command:

              ```
               terraform plan
              ```

              If the configuration description is correct, the terminal will display a list of the resources being created and their settings. If the configuration contains any errors, {{ TF }} will point them out.

      1. Deploy the cloud resources.

          1. If the configuration does not contain any errors, run this command:

              ```
              terraform apply
              ```

    - API {#api}

      To assign a role for a cloud to a service account, use the [setAccessBindings](../../iam/api-ref/ServiceAccount/setAccessBindings.md) REST API method for the [ServiceAccount](../../iam/api-ref/ServiceAccount/index.md) resource or the [ServiceAccountService/SetAccessBindings](../../iam/api-ref/grpc/ServiceAccount/setAccessBindings.md) gRPC API call.

    {% endlist %}

## Deploy your resources {#create-resources}

1. Set up the CLI profile to run operations under the service account:

    {% list tabs group=instructions %}

    - CLI {#cli}

      1. Create an [authorized key](../../iam/concepts/authorization/key.md) for the service account and save it to the file:

          ```
          yc iam key create \
            --service-account-id <service_account_ID> \
            --folder-id <folder_ID> \
            --output key.json
          ```

          Where:
          * `--service-account-id`: `sa-live-debug` service account ID.
          * `--folder-id`: Service account folder ID.
          * `--output`: Authorized key file name.

          Result:

          ```
          id: aje8nn871qo4********
          service_account_id: ajehr0to1g8********
          created_at: "2023-03-04T09:16:43.479156798Z"
          key_algorithm: RSA_2048
          ```

      1. Create a CLI profile to run operations under the service account:

          ```
          yc config profile create sa-live-debug
          ```

          Result:

          ```
          Profile 'sa-live-debug' created and activated
          ```

      1. Configure the profile:

          ```
          yc config set service-account-key key.json
          yc config set cloud-id <cloud_ID>
          ```

          Where:
          * `service-account-key`: Service account authorized key file.
          * `cloud-id`: [Cloud ID](../../resource-manager/operations/cloud/get-id.md).

      1. Add your credentials to the environment variables:

          ```
          export YC_TOKEN=$(yc iam create-token)
          export YC_CLOUD_ID=$(yc config get cloud-id)
          ```

    {% endlist %}

1. Deploy the resources in the cloud by running this command:

    ```
    npx serverless-live-debug deploy
    ```

    As a result, the command will create a folder named `live-debug` in the cloud and deploy all the required resources there.

## Run the debugging service {#run-client}

1. In the `live-debug-test` folder, create a file named `live-debug.config.ts`:

    ```
    nano live-debug.config.ts
    ```

1. Copy the code with the following configuration to the `live-debug.config.ts` file:

    ```
    import { defineConfig } from '@yandex-cloud/serverless-live-debug';
    import { Handler } from '@yandex-cloud/function-types';

    export default defineConfig({
      handler: <Handler.Http>(event => {
        console.log('got request', event);
        return {
          statusCode: 200,
          body: `Hello from local code!`,
        };
      })
    });
    ```

1. Start the debugging service by running the following command:

    ```
    npx serverless-live-debug run
    ```

    Result:

    ```
    Using config: live-debug.config.ts
    Running local client...
    Starting child...
    Child started
    Watching changes in: live-debug.config.ts
    WS connection opened
    Local client ready.
    Check url: https://{{ api-host-apigw }}
    Waiting requests...
    ```

    Where `Check url` is the public address of the [{{ api-gw-name }}](../../api-gateway/concepts/index.md).

1. Make sure the debug code is working properly by opening another terminal and running this command:

    ```
    curl https://{{ api-host-apigw }}
    ```

    Result:

    ```
    Hello from local code!
    ```

For more information about usage examples, see the [yc-serverless-live-debug](https://github.com/yandex-cloud/yc-serverless-live-debug) repository. 

## How to delete the resources you created {#clear-out}

Delete the folder with the resources required for interactive debugging of {{ sf-name }}:

{% list tabs group=instructions %}

- Management console {#console}

  1. In the [management console]({{ link-console-cloud }}), select `live-debug`. 
  1. Click ![image](../../_assets/console-icons/ellipsis.svg) next to the folder and select **{{ ui-key.yacloud.common.delete }}**.
  1. In the **{{ ui-key.yacloud.component.iam-delete-folder-or-cloud-dialog.field_folder-delete-after }}** field, select `{{ ui-key.yacloud_billing.component.iam-delete-folder-or-cloud-dialog.label_delete-now }}`.
  1. Click **{{ ui-key.yacloud.common.delete }}**.

- API {#api}

  To delete a folder, use the [delete](../../resource-manager/api-ref/Folder/delete.md) REST API method for the [Folder](../../resource-manager/api-ref/Folder/index.md) resource or the [FolderService/Delete](../../resource-manager/api-ref/grpc/Folder/delete.md) gRPC API call.

{% endlist %}