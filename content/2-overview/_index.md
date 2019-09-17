+++
title = "Overview"
chapter = false
weight = 2
+++

<p>To understand how you can use the new WebSocket API feature of API Gateway, I will show you how to build a real-time chat app. To simplify things, assume that there is only a single chat room in this app. Here are the capabilities that the app includes:</p> 
       <ul> 
        <li>Clients join the chat room as they connect to the WebSocket API.</li> 
        <li>The backend can send messages to specific users via a callback URL that is provided after the user is connected to the WebSocket API.</li> 
        <li>Users can send messages to the room.</li> 
        <li>Disconnected clients are removed from the chat room.</li> 
       </ul> 
    
Here’s an overview of the real-time chat application:

<div id="attachment_5799" style="width: 757px" class="wp-caption aligncenter"> 
<img class="wp-image-5799 size-full" src="https://d2908q01vomqb2.cloudfront.net/1b6453892473a467d07372d45eb05abc2031647a/2018/12/18/websockets-arch.png" alt="" width="747" height="429"> 

The application is composed of the WebSocket API in API Gateway that handles the connectivity between the client and servers (1). Two <a href="https://aws.amazon.com/lambda/" target="_blank" rel="noopener">AWS Lambda</a> functions react when clients connect (2) or disconnect (5) from the API. The sendMessage function (3) is invoked when the clients send messages to the server. The server sends the message to all connected clients (4) using the new API Gateway Management API. To track each of the connected clients, use a DynamoDB table to persist the connection identifiers.