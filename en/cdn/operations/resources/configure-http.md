---
title: How to configure HTTP methods in {{ cdn-full-name }}
description: Follow this guide to configure HTTP methods.
---

# Configuring HTTP methods

To configure the allowed HTTP client requests methods for a resource:

{% list tabs group=instructions %}

- Management console {#console}

  1. In the [management console]({{ link-console-main }}), select the folder where your resource is located.

  1. Select **{{ ui-key.yacloud.iam.folder.dashboard.label_cdn }}**.

  1. Click the resource name.

  1. Navigate to the **{{ ui-key.yacloud.cdn.label_resource-http-headers }}** tab.

  1. In the top-right corner, click ![image](../../../_assets/console-icons/pencil.svg) **{{ ui-key.yacloud.common.edit }}**.

  1. Under **{{ ui-key.yacloud.cdn.label_resource-http-headers-http-methods }}**, select **{{ ui-key.yacloud.cdn.label_resource-http-headers-allowed-methods }}** from the drop-down list.

  1. Click **{{ ui-key.yacloud.common.save }}**.

- CLI {#cli}

  {% include [include](../../../_includes/cli-install.md) %}

  {% include [default-catalogue](../../../_includes/default-catalogue.md) %}

  1. View the description of the CLI command to update a resource:

      ```bash
      yc cdn resource update --help
      ```

  1. Get a list of all resources in the default folder:

      ```bash
      yc cdn resource list --format yaml
      ```

      Result:

      ```text
      id: s0me1dkfjq********
      folder_id: s0mef01der7p********
      cname: testexample.com
      created_at: "2022-01-19T09:23:57.921365Z"
      updated_at: "2022-01-19T10:55:30.305141Z"
      active: true
      options:
        edge_cache_settings:
          enabled: true
          default value: "345600"
        cache_http_headers:
          enabled: true
          value:
          - content-type
          - content-length
          - connection
          - server
          - date
          - test
        stale:
          enabled: true
          value:
          - error
          - updating
        allowed_http_methods:
          value:
          - GET
          - POST
          - HEAD
          - OPTIONS
      origin_group_id: "89783"
      origin_group_name: My origins group
      origin_protocol: HTTP
      ssl_certificate:
        type: DONT_USE
        status: READY
      ```

  1. Specify the allowed HTTP methods for client requests using the `--allowed-http-methods` parameter:

      ```bash
      yc cdn resource update <resource_ID> \
        --allowed-http-methods <allowed_methods>
      ```

      For more information about the `yc cdn resource update` command, see the [CLI reference](../../../cli/cli-ref/cdn/cli-ref/resource/update.md).

- {{ TF }} {#tf}

  {% include [terraform-install](../../../_includes/terraform-install.md) %}

  1. In the configuration file, describe the properties of the CDN resource to create:

      ```hcl
      terraform {
        required_providers {
          yandex = {
            source  = "yandex-cloud/yandex"
            version = "0.69.0"
          }
        }
      }

      provider "yandex" {
        token     = "<OAuth_token>"
        cloud_id  = "<cloud_ID>"
        folder_id = "<folder_ID>"
        zone      = "<availability_zone>"
      }

      resource "yandex_cdn_resource" "my_resource" {
          cname               = "cdn1.yandex-example.ru"
          active              = false
          origin_protocol     = "https"
          secondary_hostnames = ["cdn-example-1.yandex.ru", "cdn-example-2.yandex.ru"]
          origin_group_id     = yandex_cdn_origin_group.my_group.id
          options {
            allowed_http_methods = ["GET","PUT"]
          }

      }
      ```

      Where:

      * `cname`: Primary domain name used for content distribution. This is a required parameter.
      * `active`: Flag indicating content availability to end users. `True`: CDN content will be available to clients. This is an optional parameter. The default value is `true`.
      * `origin_protocol`: Protocol for origins. This is an optional parameter. The default value is `http`.
      * `secondary_hostnames`: Additional domain names. This is an optional parameter.
      * `origin_group_id`: [Origin group](../../concepts/origins.md) ID. This is a required parameter. Use the ID from the description of the origin group in the `yandex_cdn_origin_group` resource.
      * The `options` section contains additional parameters of CDN resources:
         * `allowed_http_methods`: HTTP methods allowed for your CDN content. By default, the following methods are allowed: `GET`, `HEAD`, `POST`, `PUT`, `PATCH`, `DELETE`, and `OPTIONS`. If the user is not allowed to use any method, they will get the `405` _Method Not Allowed_ response. For methods that are not supported, the user will get `501` (_Not Implemented_). This is an optional parameter. Its default values are `GET`, `HEAD`, `POST`, and `OPTIONS`.

      For more information about the `yandex_cdn_resource` properties in {{ TF }}, see the [provider documentation]({{ tf-provider-resources-link }}/cdn_resource).

  1. In the command line, go to the directory with the {{ TF }} configuration file.

  1. Check the configuration using this command:
     ```
     terraform validate
     ```

     If the configuration is correct, you will get this message:

     ```
     Success! The configuration is valid.
     ```

  1. Run this command:
     ```
     terraform plan
     ```

     You will see a detailed list of resources. No changes will be made at this step. If the configuration contains errors, {{ TF }} will show them.

  1. Apply the changes:
     ```
     terraform apply
     ```

  1. Type `yes` and press **Enter** to confirm the changes.

     You can check the CDN resource update in the [management console]({{ link-console-main }}) or using this [CLI](../../../cli/quickstart.md) command:

     ```
     yc cdn resource list
     ```

- API {#api}

  Use the [update](../../api-ref/Resource/update.md) REST API method for the [Resource](../../api-ref/Resource/index.md) resource or the [ResourceService/Update](../../api-ref/grpc/Resource/update.md) gRPC API call.

{% endlist %}

{% include [after-changes-tip](../../../_includes/cdn/after-changes-tip.md) %}

## Examples {#examples}

{% list tabs group=instructions %}

- CLI {#cli}

  Add the allowed GET method to the resource:

    ```bash
    yc cdn resource update s0me1dkfjq******** --allowed-http-methods GET
    ```
  
  Result:

    ```text
    id: s0me1dkfjq********

    ...

    cname: testexample.com
    active: true

    ...

    allowed_http_methods:
    enabled: true
    value:
    - GET
    ```

{% endlist %}

#### See also {#see-also}

* [{#T}](../../concepts/clients-to-servers.md)