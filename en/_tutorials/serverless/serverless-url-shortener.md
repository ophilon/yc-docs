# URL shortener


In this tutorial, you will create a URL shortener using {{ yandex-cloud }} serverless technology.

This tool accepts user requests via a public [API gateway](../../api-gateway/concepts/index.md). The user receives an HTML page with a URL input field from [hosting](../../storage/concepts/hosting.md). The [function](../../functions/concepts/function.md) sends the entered URL for storage in a [serverless database](../../ydb/concepts/serverless-and-dedicated.md#serverless), shortens it, and returns it to the user. When the user follows the shortened URL, the function retrieves the full URL from the database and and performs a redirect.

To configure and test the tool:
1. [Get your cloud ready](#before-begin).
1. [Set up hosting for the URL shortener page](#object-storage).
1. [Create a service account](#service-account).
1. [Create a database in {{ ydb-full-name }}](#ydb).
1. [Set up a function in {{ sf-full-name }}](#function).
1. [Publish your URL shortener via {{ api-gw-full-name }}](#api-gw).
1. [Test the URL shortener](#test-shortener).

If you no longer need the resources you created, [delete them](#clear-out).

## Get your cloud ready {#before-begin}

{% include [before-you-begin](../_tutorials_includes/before-you-begin.md) %}

### Required paid resources {#paid-resources}

The cost of resources for our scenario includes:
* Fee for using the storage (see [{{ objstorage-full-name }} pricing](../../storage/pricing.md)).
* Fee for database queries (see [{{ ydb-name }} pricing](../../ydb/pricing/serverless.md)).
* Fee for function invocations (see [{{ sf-name }} pricing](../../functions/pricing.md)).
* Fee for API gateway requests (see [{{ api-gw-name }} pricing](../../api-gateway/pricing.md)).

## Set up hosting for the URL shortener page {#object-storage}

To create a bucket, upload your URL shortener's HTML page to it, and configure it for static website hosting:

{% list tabs group=instructions %}

- Management console {#console}

  1. In the [management console]({{ link-console-main }}), select your working folder.
  1. Select **{{ ui-key.yacloud.iam.folder.dashboard.label_storage }}**.
  1. Click **{{ ui-key.yacloud.storage.buckets.button_create }}**.
  1. On the bucket creation page:
     1. Enter a name for the bucket.

        {% note warning %}

        Bucket names are unique throughout {{ objstorage-name }}, which means you cannot create two buckets with the same name, even in different folders belonging to different clouds.

        {% endnote %}

     1. Set the maximum size to `1 {{ ui-key.yacloud.common.units.label_gigabyte }}`.
     1. Set object read access to `{{ ui-key.yacloud.storage.bucket.settings.access_value_public }}`.
     1. Click **{{ ui-key.yacloud.storage.buckets.create.button_create }}** to complete the operation.
  1. Copy the HTML code and paste it into `index.html`:

     {% cut "HTML code" %}

     ```html
     <!DOCTYPE html>
     <html lang="en">

     <head>
       <meta charset="UTF-8">
       <title>URL shortener</title>
       <!-- warns against sending unnecessary GET requests to /favicon.ico -->
       <link rel="icon" href="data:;base64,iVBORw0KGgo=">
     </head>

     <body>
       <h1>Welcome</h1>
       <form action="javascript:shorten()">
         <label for="url">Enter a link:</label><br>
         <input id="url" name="url" type="text"><br>
         <input type="submit" value="Shorten">
       </form>
       <p id="shortened"></p>
     </body>

     <script>
       function shorten() {
         const link = document.getElementById("url").value
         fetch("/shorten", {
           method: 'POST',
           headers: {
             'Content-Type': 'application/json'
           },
           body: link
         })
         .then(response => response.json())
         .then(data => {
           const url = data.url
           document.getElementById("shortened").innerHTML = `<a href=${url}>${url}</a>`
         })
         .catch(error => {
           document.getElementById("shortened").innerHTML = `<p>An error (${error}) occurred. Try again.</p>`
         })
       }
     </script>

     </html>
     ```

     {% endcut %}

  1. Click the name of the bucket you created.
  1. Click **{{ ui-key.yacloud.storage.bucket.button_empty-create }}**.
  1. Specify the `index.html` file you set up earlier.
  1. Click **{{ ui-key.yacloud.storage.button_upload }}**.
  1. In the left-hand panel, select **{{ ui-key.yacloud.storage.bucket.switch_settings }}**.
  1. On the **{{ ui-key.yacloud.storage.bucket.switch_website }}** tab:
        * Select `{{ ui-key.yacloud.storage.bucket.website.switch_hosting }}`.
        * Specify the website's home page: `index.html`.
  1. Click **{{ ui-key.yacloud.storage.bucket.website.button_save }}**.

{% endlist %}

## Create a service account {#service-account}

To create a service account for communication between the shortener components:

{% list tabs group=instructions %}

- Management console {#console}

  1. Navigate to your working folder.
  1. In the list of services, select **{{ ui-key.yacloud.iam.folder.dashboard.label_iam }}**.
  1. Click **{{ ui-key.yacloud.iam.folder.service-accounts.button_add }}**.
  1. Specify the service account name: `serverless-shortener`.
  1. Click **{{ ui-key.yacloud.iam.folder.service-account.label_add-role }}** and select `editor`.
  1. Click **{{ ui-key.yacloud.iam.folder.service-account.popup-robot_button_add }}**.
  1. Click the name of the service account you created.

     Save the service account ID, as you will need it later.

{% endlist %}

## Create a database in {{ ydb-name }} {#ydb}

To create a database in {{ ydb-name }} and configure it to store URLs:

{% list tabs group=instructions %}

- Management console {#console}

  1. Navigate to your working folder.
  1. In the list of services, select **{{ ui-key.yacloud.iam.folder.dashboard.label_ydb }}**.
  1. Click **{{ ui-key.yacloud.ydb.databases.button_create }}**.
  1. Enter the database name: `for-serverless-shortener`.
  1. Select **{{ ui-key.yacloud.ydb.forms.label_serverless-type }}** as the database type.
  1. Click **{{ ui-key.yacloud.ydb.forms.button_create-database }}**.
  1. Wait until the database is up and running.

     While being created, your database will have the `Provisioning` status. Once it is ready for use, its status will change to `Running`.
  1. Click the database name.

     Save the **{{ ui-key.yacloud.ydb.overview.label_endpoint }}** field value, as you will need it later.
  1. In the left-hand panel, select the **{{ ui-key.yacloud.ydb.database.switch_browse }}** tab.
  1. Click **{{ ui-key.yacloud.ydb.browse.button_sql-query }}**.
  1. Copy the following SQL query and paste it into the **{{ ui-key.yacloud.ydb.sql.label_query }}** field:

     ```sql
     CREATE TABLE links
     (
       id Utf8,
       link Utf8,
       PRIMARY KEY (id)
     );
     ```

  1. Click **{{ ui-key.yacloud.ydb.sql.button_run }}**.

{% endlist %}

## Set up a function in {{ sf-name }} {#function}

To create and set up a URL shortening function:

{% list tabs group=instructions %}

- Management console {#console}

  1. Navigate to your working folder.
  1. In the list of services, select **{{ ui-key.yacloud.iam.folder.dashboard.label_serverless-functions }}**.
  1. Click **{{ ui-key.yacloud.serverless-functions.list.button_create }}**.
  1. Enter the name: `for-serverless-shortener`.
  1. Click **{{ ui-key.yacloud.common.create }}**.
  1. From the **Python** drop-down list, select the `python312` runtime environment.
  1. Click **{{ ui-key.yacloud.serverless-functions.item.editor.button_action-continue }}**.
  1. Copy the function code and paste it into `index.py` under **{{ ui-key.yacloud.serverless-functions.item.editor.label_title-source }}**.

     {% cut "Function code" %}

     ```py
     import ydb

     import urllib.parse
     import hashlib
     import base64
     import json
     import os


     def decode(event, body):
       # The request body can be encoded.
       is_base64_encoded = event.get('isBase64Encoded')
       if is_base64_encoded:
         body = str(base64.b64decode(body), 'utf-8')
       return body

     def response(statusCode, headers, isBase64Encoded, body):
       return {
         'statusCode': statusCode,
         'headers': headers,
         'isBase64Encoded': isBase64Encoded,
         'body': body,
       }

     def get_config():
       endpoint = os.getenv("endpoint")
       database = os.getenv("database")
       if endpoint is None or database is None:
         raise AssertionError("You need to specify both environment variables")
       credentials = ydb.iam.MetadataUrlCredentials()
       return ydb.DriverConfig(endpoint, database, credentials=credentials)

     def execute(config, query, params):
       with ydb.Driver(config) as driver:
         try:
           driver.wait(timeout=5, fail_fast=True)
         except TimeoutError:
           print("Connect failed to YDB")
           print("Last reported errors by discovery:")
           print(driver.discovery_debug_details())
           return None

         session = driver.table_client.session().create()
         prepared_query = session.prepare(query)
         return session.transaction(ydb.SerializableReadWrite()).execute(
           prepared_query,
           params,
           commit_tx=True
         )


     def insert_link(id, link):
       config = get_config()
       query = """
         DECLARE $id AS Utf8;
         DECLARE $link AS Utf8;

         UPSERT INTO links (id, link) VALUES ($id, $link);
         """
       params = {'$id': id, '$link': link}
       execute(config, query, params)

     def find_link(id):
       print(id)
       config = get_config()
       query = """
         DECLARE $id AS Utf8;

         SELECT link FROM links where id=$id;
         """
       params = {'$id': id}
       result_set = execute(config, query, params)

       if not result_set or not result_set[0].rows:
         return None

       return result_set[0].rows[0].link

     def shorten(event):
       body = event.get('body')

       if body:
         body = decode(event, body)
         original_host = event.get('headers').get('Origin')
         link_id = hashlib.sha256(body.encode('utf8')).hexdigest()[:6]
         # The URL may contain encoded characters, e.g., %. This can interfere with API Gateway during redirection.
         # To fix this, decode the URL by invoking `urllib.parse.unquote`.
         insert_link(link_id, urllib.parse.unquote(body))
         return response(200, {'Content-Type': 'application/json'}, False, json.dumps({'url': f'{original_host}/r/{link_id}'}))
       return response(400, {}, False, 'The url parameter is missing in the request body')

     def redirect(event):
       link_id = event.get('pathParams').get('id')
       redirect_to = find_link(link_id)

       if redirect_to:
         return response(302, {'Location': redirect_to}, False, '')

       return response(404, {}, False, 'This link does not exist')

     # These checks are required because we have only one function.
     # Ideally, each path in API Gateway should have a function of its own.
     def get_result(url, event):
       if url == "/shorten":
         return shorten(event)
       if url.startswith("/r/"):
         return redirect(event)

       return response(404, {}, False, 'This path does not exist')

     def handler(event, context):
       url = event.get('url')
       if url:
         # The URL may come from API Gateway with a question mark at the end.
         if url[-1] == '?':
           url = url[:-1]
         return get_result(url, event)

       return response(404, {}, False, 'This function should be invoked using API Gateway.')
     ```

     {% endcut %}

  1. Under **{{ ui-key.yacloud.serverless-functions.item.editor.label_title-source }}**, create a file named `requirements.txt` and paste the following text into it:

     ```txt
     ydb
     ```

  1. Specify the entry point: `index.handler`.
  1. Set the timeout to `5`.
  1. Select the `serverless-shortener` service account.
  1. Add these environment variables:
     * `endpoint`: Enter the first part of the previously saved **{{ ui-key.yacloud.ydb.overview.label_endpoint }}** field value (preceding `/?database=`), e.g., `{{ ydb.ep-serverless }}`.
     * `database`: Enter the second part of the previously saved **{{ ui-key.yacloud.ydb.overview.label_endpoint }}** field value (following `/?database=`), e.g., `/{{ region-id }}/r1gra875baom********/g5n22e7ejfr1********`.
  1. Click **{{ ui-key.yacloud.serverless-functions.item.editor.button_deploy-version }}**.
  1. Under **{{ ui-key.yacloud.serverless-functions.item.overview.label_title }}**, enable **{{ ui-key.yacloud.serverless-functions.item.overview.label_all-users-invoke }}**.

  Save the function ID, as you will need it later.

{% endlist %}

## Publish your URL shortener via {{ api-gw-name }} {#api-gw}

To publish your URL shortener via {{ api-gw-name }}:

{% list tabs group=instructions %}

- Management console {#console}

  1. Navigate to your working folder.
  1. In the list of services, select **{{ ui-key.yacloud.iam.folder.dashboard.label_api-gateway }}**.
  1. Click **{{ ui-key.yacloud.serverless-functions.gateways.list.button_create }}**.
  1. In the **{{ ui-key.yacloud.common.name }}** field, enter `for-serverless-shortener`.
  1. Copy and paste the following code under **{{ ui-key.yacloud.serverless-functions.gateways.form.field_spec }}**:

     {% cut "Specification" %}

     ```yaml
     openapi: 3.0.0
     info:
       title: for-serverless-shortener
       version: 1.0.0
     paths:
       /:
         get:
           x-yc-apigateway-integration:
             type: object_storage
             bucket: <bucket_name> # <-- bucket name
             object: index.html # <-- HTML file name
             presigned_redirect: false
             service_account: <service_account_id> # <-- service account ID
           operationId: static
       /shorten:
         post:
           x-yc-apigateway-integration:
             type: cloud_functions
             function_id: <function_id> # <-- function ID
           operationId: shorten
       /r/{id}:
         get:
           x-yc-apigateway-integration:
             type: cloud_functions
             function_id: <function_id> # <-- function ID
           operationId: redirect
           parameters:
           - description: id of the url
             explode: false
             in: path
             name: id
             required: true
             schema:
               type: string
             style: simple
     ```

     {% endcut %}

     Edit the specification code as follows:
     * Replace `<service_account_id>` with the ID of the service account you created earlier.
     * Replace `<function_id>` with the ID of the function you created earlier.
     * Replace `<bucket_name>` with the name of the bucket you created earlier.
  1. Click **{{ ui-key.yacloud.serverless-functions.gateways.form.button_create-gateway }}**.
  1. Click the name of the API gateway you created.
  1. Copy the `url` value from the specification.

     Use this URL to access your URL shortener.

{% endlist %}

## Test the URL shortener {#test-shortener}

To make sure your shortener components communicate properly:
1. In your browser, open the previously copied URL.
1. In the input field, enter the URL you want to shorten.
1. Click **Shorten**.

   You will see the shortened URL below.
1. Open the shortened URL. It should lead to the same page as the original link.

## How to delete the resources you created {#clear-out}

To stop paying for the resources you created:
1. [Delete the API gateway](../../api-gateway/operations/api-gw-delete.md).
1. [Delete the function](../../functions/operations/function/function-delete.md).
1. [Delete the database](../../ydb/operations/manage-databases.md#delete-db).
1. [Delete the bucket](../../storage/operations/buckets/delete.md).
1. [Delete the service account](../../iam/operations/sa/delete.md).