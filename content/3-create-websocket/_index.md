+++
title = "Create a new WebSocket API"
chapter = false
weight = 3
+++

<ol> 
<li>In the API Gateway console, choose <strong>Create API, New API</strong>.</li> 
<li>Under <strong>Choose the protocol</strong>, choose <strong>WebSocket</strong>.</li> 
<li>For <strong>API name</strong>, enter <strong>My Chat API</strong>.</li> 
<li>For <strong>Route Selection Expression</strong>, enter <strong>$request.body.action</strong>.</li> 
<li>Enter a description if you’d like and click <strong>Create API</strong>.</li> 
</ol> 

The attribute described by the <strong>Route Selection Expression</strong> value should be present in every message that a client sends to the API. Here’s one example of a request message:

```
{
    "action": "sendmessage",
    “data”: "Hello, I am using WebSocket APIs in API Gateway.”
}
```

To route messages based on the message content, the WebSocket API expects JSON-formatted messages with a property serving as a routing key.

### Manage routes

Now, configure your new API to respond to incoming requests. For this app, create three routes as described earlier. First, configure the <strong>sendmessage</strong> route with a Lambda integration.

<ol> 
<li>In the API Gateway console, under <strong>My Chat API</strong>, choose <strong>Routes</strong>.</li> 
<li>For <strong>New Route Key</strong>, enter <strong>sendmessage</strong> and confirm it.</li> 
</ol> 

Each route includes route-specific information such as its model schemas, as well as a reference to its target integration. With the <strong>sendmessage</strong> route created, <a href="https://docs.aws.amazon.com/apigateway/latest/developerguide/api-gateway-create-api-as-simple-proxy-for-lambda.html" target="_blank" rel="noopener">use a Lambda function as an integration</a> that is responsible for sending messages.

At this point, you’ve either deployed the AWS Serverless Application Repository backend or deployed it yourself using AWS SAM. In the Lambda console, look at the sendMessage function. This function receives the data sent by one of the clients, looks up all the currently connected clients, and sends the provided data to each of them. Here’s a snippet from the Lambda function:

```
DDB.scan(scanParams, function (err, data) {
    // some code omitted for brevity
   var apigwManagementApi = new AWS.ApiGatewayManagementApi({
      apiVersion: "2018-11-29",
      endpoint: event.requestContext.domainName " /" + event.requestContext.stage
   });
    var postParams = {
        data: JSON.parse(event.body).data
    };
    let callbackArray = data.Items.map( async(el) => { ... });
    Promise.all(callbackArray);
    // some code omitted for brevity
});
```

When one of the clients sends a message with an action of “sendmessage,” this function scans the DynamoDB table to find all of the currently connected clients. For each of them, it makes a <em>postToConnection</em> call with the provided data.

To send messages properly, you must know the API to which your clients are connected. This means explicitly setting the SDK endpoint to point to your API.

What’s left is to properly track the clients that are connected to the chat room. For this, take advantage of two of the special <em>routeKey</em> values and implement routes for <strong>$connect</strong> and <strong>$disconnect</strong>.

The <strong>onConnect</strong> function inserts the <em>connectionId</em> value from <em>requestContext</em> to the DynamoDB table.

```
exports.handler = function(event, context, callback) {
  var putParams = {
    TableName: process.env.TABLE_NAME,
    Item: {
      connectionId: { S: event.requestContext.connectionId }
    }
  };

  DDB.putItem(putParams, function(err, data) {
    callback(null, {
      statusCode: err ? 500 : 200,
      body: err ? "Failed to connect: " + JSON.stringify(err) : "Connected"
    });
  });
};
```

The <em>onDisconnect</em> function removes the record corresponding with the specified connectionId value.

```
exports.handler = function(event, context, callback) {
  var deleteParams = {
    TableName: process.env.TABLE_NAME,
    Key: {
      connectionId: { S: event.requestContext.connectionId }
    }
  };

  DDB.deleteItem(deleteParams, function(err) {
    callback(null, {
      statusCode: err ? 500 : 200,
      body: err ? "Failed to disconnect: " + JSON.stringify(err) : "Disconnected."
    });
  });
};
```

As I mentioned earlier, the <em>onDisconnect</em> function may not be called. To keep from retrying connections that are never going to be available again, add some additional logic in case the <em>postToConnection</em> call does not succeed:

```
if (err.statusCode === 410) {
  console.log("Found stale connection, deleting " + postParams.connectionId);
  DDB.deleteItem({ TableName: process.env.TABLE_NAME,
                   Key: { connectionId: { S: postParams.connectionId } } });
} else {
  console.log("Failed to post. Error: " + JSON.stringify(err));
}
```

API Gateway returns a status of 410 GONE when the connection is no longer available. If this happens, delete the identifier from the DynamoDB table.

To make calls to your connected clients, your application needs a new permission: “execute-api:ManageConnections”. This is handled for you in the AWS SAM template.