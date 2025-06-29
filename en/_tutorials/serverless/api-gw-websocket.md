# Working with an API gateway via WebSocket

To establish a WebSocket connection to an API gateway:

1. [Create an API gateway](#create).
1. [Establish a connection](#connect).
1. [Test the connection](#check).

If you no longer need the resources you created, [delete](#clear-out) them.

## Getting started {#before-you-begin}

{% include [before-you-begin](../_tutorials_includes/before-you-begin.md) %}

### Required paid resources {#paid-resources}

The cost of the resources includes the fee for the number of API gateway requests and outbound traffic (see [{{ api-gw-full-name }} pricing](../../api-gateway/pricing.md)).

## Create an API gateway {#create}

{% list tabs group=instructions %}

- Management console {#console}

    1. In the [management console]({{ link-console-main }}), select the folder where you want to create an API gateway.
    1. In the list of services, select **{{ ui-key.yacloud.iam.folder.dashboard.label_api-gateway }}**.
    1. Click **{{ ui-key.yacloud.serverless-functions.gateways.list.button_create }}**.
    1. In the **{{ ui-key.yacloud.common.name }}** field, enter `websocket`.
    1. Optionally, in the **{{ ui-key.yacloud.common.description }}** field, provide a description.
    1. Under **{{ ui-key.yacloud.serverless-functions.gateways.form.field_spec }}**, add the following specification:

        ```yaml
        openapi: 3.0.0
        info:
          title: Test API
          version: 1.0.0
        paths:
          /connections:
            x-yc-apigateway-websocket-message:
              summary: Get connection identifier
              operationId: getConnectionID
              parameters:
                - name: X-Yc-Apigateway-Websocket-Connection-Id
                  in: header
                  description: Websocket connection identifier
                  required: true
                  schema:
                    type: string
              responses:
                '200':
                  description: Connection identifier
                  content:
                    text/plain:
                      schema:
                        type: string
              x-yc-apigateway-integration:
                type: dummy
                http_code: 200
                http_headers:
                  Content-Type: application/json
                content:
                  text/plain: '{"connection_id":"{X-Yc-Apigateway-Websocket-Connection-Id}"}'
        ```

    1. Click **{{ ui-key.yacloud.serverless-functions.gateways.form.button_create-gateway }}**.

{% endlist %}

## Establish a connection {#connect}

1. Open the terminal and install [wscat](https://www.npmjs.com/package/wscat):

    ```bash
    npm install -g wscat
    ```

1. Connect to the API gateway. Replace `<API_gateway_domain>`, with the API gateway domain formatted as `{{ api-host-apigw }}`:

    ```bash
    wscat -c wss://<API_gateway_domain>/connections
    Connected (press CTRL+C to quit)
    ```

1. Type a message and press `Enter`.

The message will go to the API gateway over the established connection. The API gateway will call integration and return a response that you will see on your screen. The response will contain the established connection ID:

```bash
> Hello!
< {"connection_id":"<connection_ID>"}
```

## Test the connection {#check}

Open a new terminal window and test the connection using the connection ID you got.

{% list tabs group=instructions %}

- CLI {#cli}

    {% include [cli-install](../../_includes/cli-install.md) %}

    {% include [default-catalogue](../../_includes/default-catalogue.md) %}

    1. Retrieve information on the established connection:

        ```bash
        yc serverless api-gateway websocket get <connection_ID>
        ```

    1. Send a message to the client:

        ```bash
        yc serverless api-gateway websocket send <connection_ID> --data Hello!
        ```

    1. Terminate the connection:

        ```bash
        yc serverless api-gateway websocket disconnect <connection_ID>
        ```

    1. Go to the terminal window with the established connection. It should display the following information:

        ```bash
        wscat -c wss://<API_gateway_domain>/connections
        Connected (press CTRL+C to quit)
        > Hello!
        < {"connection_id":"<connection_ID>"}
        < Hello!
        Disconnected (code: 1000, reason: "")
        ```

        Where `API_gateway_domain` is a string formatted as `{{ api-host-apigw }}`.

{% endlist %}

## How to delete the resources you created {#clear-out}

To stop paying for the resources created, delete the [API gateway](../../api-gateway/operations/api-gw-delete.md).
