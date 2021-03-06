# Webhooks Documentation (Developer Preview)

## Introduction  

Webhooks let you build custom applications or workflows that react
to events that happen in Tableau Online. For example, you could use
webhooks to send an SMS or Slack notification any time
a datasource refresh fails, or fire off a confetti cannon when a new
workbook is created.  For the initial release of this developer preview,
webhooks are supported for datasource and workbook events only.  

You configure each webhook to subscribe to an event in Tableau. Then,
when the event occurs, an HTTP POST request will be sent to the public
URL you specified. This POST request includes a JSON payload that
includes information about the event. The payload includes the ID of the
object in question so that the Tableau REST API can be used to get
additional information or take further action.  

## Prerequisites  

To use Tableau webhooks, you must:

- enroll in the developer preview of the webhooks feature.  

- connect to the Tableau Online instance where the webhooks feature is enabled, [https://10ax.online.tableau.com](https://10ax.online.tableau.com/).  

- be a site administrator.

## Set Up a Webhook Using Postman

Here is an example of setting up a webhook with the REST API
using Postman.

1. Download the file [Postman-Collection-Tableau-Webhooks.json](Postman-Collection-Tableau-Webhooks.json)

1. Download Postman from [https://www.getpostman.com/](https://www.getpostman.com/)

1. Launch Postman.

1. Click **File** \> **Import** and choose the Postman collection you downloaded. The collection appears on the left.

1. To configure the variables in the collection, click the ellipsis beside the collection name in the left sidebar.

1. Click **Edit**.

1. Click the **Variables** tab, change the Tableau\_Server variable to the correct server. Click **Update**.

1. In the list of requests, click **Sign-in** and then click **Send**. The response body contains the site id and a token.  

1. Click the **Variables** tab and set the Site\_ID and Tableau\_Auth\_Token variables.

1. In the list of requests, click **Create a webhook**. In the request body, change the webhook **name** and **url**. The webhook destination URL must be https and have a valid certificate.

1. Click **Send**. The ID of the new webhook is returned in the response body.

1. In the list of requests, click **Test a webhook**. Set the webhook ID to test in the variables or in the URI and then click **Send**. Testing the webhook sends an empty payload to the configured destination URL of the webhook and returns the response from the server. This is useful for testing, to ensure that things are being sent from Tableau and received back as expected.

## Set Up a Webhook Using cURL

Here is an example of setting up a webhook with the REST API
using cURL.

### Sign In

`curl "http://<server>/api/3.1/auth/signin" -X POST -d @signin.xml`

Content of signin.xml:

```
<tsRequest>

  <credentials name="<username>" password="<password>" >

    <site contentUrl="" />

  </credentials>

</tsRequest>
```

### Create a Webhook

`curl "http://<server>/api/exp/sites/<site-id>/webhooks" -X POST -H "X-Tableau-Auth:<token>" -d @details.xml`

Replace token with the token from the sign in response body.

Content of details.xml:

```
<tsRequest>

  <webhook name="my_first_webhook">

    <webhook-source>

      <webhook-source-event-datasource-created />

    </webhook-source>

    <webhook-destination>

      <webhook-destination-http method="POST" url="<URL>" />

    </webhook-destination>

  </webhook>

</tsRequest>
```

Replace URL with the destination URL for the webhook. The webhook destination URL must be https and have a valid certificate.

### Test the Webhook

Testing the webhook sends an empty payload to the configured destination
URL of the webhook and returns the response from the server. This is
useful for testing, to ensure that things are being sent from Tableau
and received back as expected.

`curl "http://<server>/api/exp/sites/<site-id>/webhooks/<webhook-id>" -X GET -H "X-Tableau-Auth:<token>"`

Replace webhook-id with the webhook id from the create webhook response body.

### List Webhooks

`curl "http://<server>/api/exp/sites/<site-id>/webhooks" -X GET -H "X-Tableau-Auth:<token>"`

## Events

For the initial release of the developer preview of webhooks, these events are supported:  

| Friendly Event Name          | API Event Name                                    |
| ---------------------------- | ------------------------------------------------- |
| Datasource Refresh Started   | webhook-source-event-datasource-refresh-started   |
| Datasource Refresh Succeeded | webhook-source-event-datasource-refresh-succeeded |
| Datasource Refresh Failed    | webhook-source-event-datasource-refresh-failed    |
| Datasource Updated           | webhook-source-event-datasource-updated           |
| Datasource Created           | webhook-source-event-datasource-created           |
| Datasource Deleted           | webhook-source-event-datasource-deleted           |
| Workbook Updated             | webhook-source-event-workbook-updated             |
| Workbook Created             | webhook-source-event-workbook-created             |
| Workbook Deleted             | webhook-source-event-workbook-deleted             |

## Payloads  

When one of the subscribed events fires, a JSON payload is sent to the
URL that is configured. The payloads vary based on the type of event.  

### Datasource Events  

The payloads for the datasource events (refresh started, refresh
succeeded, refresh failed, created, deleted, updated) are the same:  

```
{  

  "resource":"DATASOURCE",  

  "event-type":"DatasourceCreated",  

  "resource-name":"My Datasource",  

  "site-id":"8b2a95d8-52b9-40a4-8712-cd6da771bd1b",  

  "resource-id":"99"  

}
```
| Field         | Description                                                                          |
| ------------------ | ----------------------------------------------------------------------------------------- |
| resource           | Will always be “DATASOURCE” for datasource events.                                        |
| event-type         | Type of event that occurred. Can be DatasourceRefreshStarted, DatasourceRefreshSucceeded, DatasourceRefreshFailed, DatasourceCreated, DatasourceDeleted, or DatasourceUpdated. |
| resource-name      | Name of the datasource in question.                                                       |
| site-id            | LUID for the site that contains the datasource.                                           |
| resource-id        | The datasource ID.                                                                        |

### Workbook Events  

The payloads for the workbook events (created, deleted, updated) are the
same:  

```
{  

  "resource":"WORKBOOK",  

  "event-type":"WorkbookCreated",  

  "resource-name":"My Workbook",  

  "site-id":"8b2a95d8-52b9-40a4-8712-cd6da771bd1b",  

  "resource-id":"99"  

}

```

| Field         | Description                                                                          |
| ------------------ | ----------------------------------------------------------------------------------------- |
| resource           | Will always be “WORKBOOK” for workbook events.                                            |
| event-type         | Type of event that occurred. Can be WorkbookCreated, WorkbookDeleted, or WorkbookUpdated. |
| resource-name      | Name of the workbook in question.                                                         |
| site-id            | LUID for the site that contains the workbook.                                             |
| resource-id        | The workbook ID.                                                                          |

## Tableau REST API Endpoints for Webhooks  

### API Version  

All REST API endpoints for webhooks are under the new experimental API version, “exp”. The base URL for the experimental API is: `https://10ax.online.tableau.com/api/exp/`.

### Authentication  

The Tableau Server REST API requires that you send an authentication token, in the **X-Tableau-Auth** header, with each request. The token lets the server verify your identity and makes sure that you signed in. For more information, see [Signing In and Signing Out (Authentication)](https://onlinehelp.tableau.com/current/api/rest_api/en-us/help.htm).  

### Create a Webhook  

Creates a new webhook for a site.  

#### URI

`POST /api/exp/sites/site-id/webhooks`

#### Parameter Values

`site-id` The ID of the site to create the webhook in.  
  
#### Request Body

```
<tsRequest>  
  <webhook name="webhook-name">  
    <webhook-source>  
      <webhook-source-event-name />  
    </webhook-source>  
    <webhook-destination>  
      <webhook-destination-http method="POST" url="url" />  
    </webhook-destination>  
  </webhook>  
</tsRequest>
```

#### Attribute Values

`webhook-name`   A name for the webhook.

`webhook-source-event-name`   The API event name for the source event. It must be one of the supported events, such as, \<webhook-source-event-datasource-refresh-started />  

`url`   The destination URL for the webhook. The webhook destination URL must be https and have a valid certificate.
  
#### Response Code

`201`  

#### Response Body

```
<tsResponse>  
    <webhook id="webhook-id" name="webhook-name">  
        <webhook-source>  
            <webhook-source-event-name />  
        </webhook-source>  
        <webhook-destination>  
            <webhook-destination-http method="POST" url="url"/>  
        </webhook-destination>  
    </webhook>  
</tsResponse>
```

#### Response Headers

`Location: /api/api-version/sites/site-id/webhooks/new-webhook-id`  

### Get a Webhook  

Returns information about the specified webhook.  

#### URI

`GET /api/exp/sites/site-id/webhooks/webhook-id`

#### Parameter Values

`site-id`   The ID of the site that contains the webhook.

`webhook-id`   The ID of the webhook to get information for.  
  
#### Request Body

None  

#### Response Code

`200`
  
#### Response Body

```
<tsResponse>  
    <webhook id="webhook-id" name="webhook-name">  
        <webhook-source>  
            <webhook-source-event-name />  
        </webhook-source>  
        <webhook-destination>  
            <webhook-destination-http method="POST" url="url"/>  
        </webhook-destination>  
    </webhook>  
</tsResponse>
```

### List Webhooks

Returns a list of all the webhooks on the specified site.  

#### URI

`GET /api/exp/sites/site-id/webhooks`
  
#### Parameter Values

`site-id`   The ID of the site that contains the webhooks.  
  
#### Request Body

None  

#### Response Code

`200`

#### Response Body

```
<tsResponse>  
   <webhooks>  
      <webhook id="webhook-id" name="webhook-name">  
        <webhook-source>  
            <webhook-source-event-name />  
        </webhook-source>  
        <webhook-destination>  
            <webhook-destination-http method="POST" url="url"/>  
        </webhook-destination>  
       </webhook>  
      ... additional webhooks ...  
   </webhooks>  
</tsResponse>  
```

### Test a Webhook

Tests the specified webhook. Sends an empty payload to the configured destination URL of the webhook and returns the response from the server. This is useful for testing, to ensure that things are being sent from Tableau and received back as expected.  

#### URI

`GET /api/exp/sites/site-id/webhooks/webhook-id/test`

#### Parameter Values

`site-id`   The ID of the site that contains the webhook.

`webhook-id`   The ID of the webhook to test.  

#### Request Body

None  

#### Response Code

`200`

#### Response Body

```
<tsResponse>  
    <webhookTestResult id="9f9bcaf8-8c4c-403c-b7e1-10dd85620f00" status="200">
       <body></body>  
    </webhookTestResult>  
</tsResponse>  
```

### Delete a Webhook  

Deletes the specified webhook.  

#### URI

`DELETE /api/exp/sites/site-id/webhooks/webhook-id`

#### Parameter Values

`site-id`   The ID of the site that contains the webhook.  

`webhook-id`   The ID of the webhook to delete.  
  
#### Request Body

None  

#### Response Code

`204`

#### Response Body

None  

### Update a Webhook  

To modify a webhook after it has been created, delete it and recreate it.
