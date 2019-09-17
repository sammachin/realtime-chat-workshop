+++
title = "Build Our App"
chapter = false
weight = 2
+++

<p>To understand how you can use the new WebSocket API feature of API Gateway, I show you how to build a real-time chat app. To simplify things, assume that there is only a single chat room in this app. Here are the capabilities that the app includes:</p> 
       <ul> 
        <li>Clients join the chat room as they connect to the WebSocket API.</li> 
        <li>The backend can send messages to specific users via a callback URL that is provided after the user is connected to the WebSocket API.</li> 
        <li>Users can send messages to the room.</li> 
        <li>Disconnected clients are removed from the chat room.</li> 
       </ul> 
    
Here’s an overview of the real-time chat application:

       <div id="attachment_5799" style="width: 757px" class="wp-caption aligncenter"> 
        <img class="wp-image-5799 size-full" src="https://d2908q01vomqb2.cloudfront.net/1b6453892473a467d07372d45eb05abc2031647a/2018/12/18/websockets-arch.png" alt="" width="747" height="429"> 
        <p class="wp-caption-text">A serverless real-time chat application using WebSocket API on Amazon API Gateway</p> 
       </div> 

The application is composed of the WebSocket API in API Gateway that handles the connectivity between the client and servers (1). Two <a href="https://aws.amazon.com/lambda/" target="_blank" rel="noopener">AWS Lambda</a> functions react when clients connect (2) or disconnect (5) from the API. The sendMessage function (3) is invoked when the clients send messages to the server. The server sends the message to all connected clients (4) using the new API Gateway Management API. To track each of the connected clients, use a DynamoDB table to persist the connection identifiers.



<h3>Create a new WebSocket API</h3> 
       <ol> 
        <li>In the API Gateway console, choose <strong>Create API, New API</strong>.</li> 
        <li>Under <strong>Choose the protocol</strong>, choose <strong>WebSocket</strong>.</li> 
        <li>For <strong>API name</strong>, enter <strong>My Chat API</strong>.</li> 
        <li>For <strong>Route Selection Expression</strong>, enter <strong>$request.body.action</strong>.</li> 
        <li>Enter a description if you’d like and click <strong>Create API</strong>.</li> 
       </ol> 
       <p>The attribute described by the <strong>Route Selection Expression</strong> value should be present in every message that a client sends to the API. Here’s one example of a request message:</p> 
       <div class="prism-show-language"><div class="prism-show-language-label" data-language="Json">Json</div></div><pre class=" language-json" data-language="Json"><code class=" language-json"><span class="token punctuation">{</span>
    <span class="token property">"action"</span><span class="token operator">:</span> <span class="token string">"sendmessage"</span><span class="token punctuation">,</span>
    “data”<span class="token operator">:</span> "Hello<span class="token punctuation">,</span> I am using WebSocket APIs in API Gateway.”
<span class="token punctuation">}</span></code></pre> 
       <p>To route messages based on the message content, the WebSocket API expects JSON-formatted messages with a property serving as a routing key.</p> 
       <h3>Manage routes</h3> 
       <p>Now, configure your new API to respond to incoming requests. For this app, create three routes as described earlier. First, configure the <strong>sendmessage</strong> route with a Lambda integration.</p> 
       <ol> 
        <li>In the API Gateway console, under <strong>My Chat API</strong>, choose <strong>Routes</strong>.</li> 
        <li>For <strong>New Route Key</strong>, enter <strong>sendmessage</strong> and confirm it.</li> 
       </ol> 
       <p>Each route includes route-specific information such as its model schemas, as well as a reference to its target integration. With the <strong>sendmessage</strong> route created, <a href="https://docs.aws.amazon.com/apigateway/latest/developerguide/api-gateway-create-api-as-simple-proxy-for-lambda.html" target="_blank" rel="noopener">use a Lambda function as an integration</a> that is responsible for sending messages.</p> 
       <p>At this point, you’ve either deployed the AWS Serverless Application Repository backend or deployed it yourself using AWS SAM. In the Lambda console, look at the sendMessage function. This function receives the data sent by one of the clients, looks up all the currently connected clients, and sends the provided data to each of them. Here’s a snippet from the Lambda function:</p> 
       <div class="prism-show-language"><div class="prism-show-language-label" data-language="Js">Js</div></div><pre class=" language-js" data-language="Js"><code class=" language-js">DDB<span class="token punctuation">.</span><span class="token function">scan</span><span class="token punctuation">(</span>scanParams<span class="token punctuation">,</span> <span class="token keyword">function</span> <span class="token punctuation">(</span>err<span class="token punctuation">,</span> data<span class="token punctuation">)</span> <span class="token punctuation">{</span>
    <span class="token comment" spellcheck="true">// some code omitted for brevity</span>
   <span class="token keyword">var</span> apigwManagementApi <span class="token operator">=</span> <span class="token keyword">new</span> <span class="token class-name">AWS<span class="token punctuation">.</span>ApiGatewayManagementApi</span><span class="token punctuation">(</span><span class="token punctuation">{</span>
      apiVersion<span class="token punctuation">:</span> <span class="token string">"2018-11-29"</span><span class="token punctuation">,</span>
      endpoint<span class="token punctuation">:</span> event<span class="token punctuation">.</span>requestContext<span class="token punctuation">.</span>domainName <span class="token string">" /"</span> <span class="token operator">+</span> event<span class="token punctuation">.</span>requestContext<span class="token punctuation">.</span>stage
   <span class="token punctuation">}</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
    <span class="token keyword">var</span> postParams <span class="token operator">=</span> <span class="token punctuation">{</span>
        data<span class="token punctuation">:</span> JSON<span class="token punctuation">.</span><span class="token function">parse</span><span class="token punctuation">(</span>event<span class="token punctuation">.</span>body<span class="token punctuation">)</span><span class="token punctuation">.</span>data
    <span class="token punctuation">}</span><span class="token punctuation">;</span>
    <span class="token keyword">let</span> callbackArray <span class="token operator">=</span> data<span class="token punctuation">.</span>Items<span class="token punctuation">.</span><span class="token function">map</span><span class="token punctuation">(</span> <span class="token keyword">async</span><span class="token punctuation">(</span>el<span class="token punctuation">)</span> <span class="token operator">=</span><span class="token operator">&gt;</span> <span class="token punctuation">{</span> <span class="token punctuation">.</span><span class="token punctuation">.</span><span class="token punctuation">.</span> <span class="token punctuation">}</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
    Promise<span class="token punctuation">.</span><span class="token function">all</span><span class="token punctuation">(</span>callbackArray<span class="token punctuation">)</span><span class="token punctuation">;</span>
    <span class="token comment" spellcheck="true">// some code omitted for brevity</span>
<span class="token punctuation">}</span><span class="token punctuation">)</span><span class="token punctuation">;</span></code></pre> 
       <p><em>Note- The code snippet above was updated on December 20, 2018 to change the casing in API names and sync the code with the GitHub repository.</em></p> 
       <p>When one of the clients sends a message with an action of “sendmessage,” this function scans the DynamoDB table to find all of the currently connected clients. For each of them, it makes a <em>postToConnection</em> call with the provided data.</p> 
       <p>To send messages properly, you must know the API to which your clients are connected. This means explicitly setting the SDK endpoint to point to your API.</p> 
       <p>What’s left is to properly track the clients that are connected to the chat room. For this, take advantage of two of the special <em>routeKey</em> values and implement routes for <strong>$connect</strong> and <strong>$disconnect</strong>.</p> 
       <p>The <strong>onConnect</strong> function inserts the <em>connectionId</em> value from <em>requestContext</em> to the DynamoDB table.</p> 
       <div class="prism-show-language"><div class="prism-show-language-label" data-language="Js">Js</div></div><pre class=" language-js" data-language="Js"><code class=" language-js">exports<span class="token punctuation">.</span>handler <span class="token operator">=</span> <span class="token keyword">function</span><span class="token punctuation">(</span>event<span class="token punctuation">,</span> context<span class="token punctuation">,</span> callback<span class="token punctuation">)</span> <span class="token punctuation">{</span>
  <span class="token keyword">var</span> putParams <span class="token operator">=</span> <span class="token punctuation">{</span>
    TableName<span class="token punctuation">:</span> process<span class="token punctuation">.</span>env<span class="token punctuation">.</span>TABLE_NAME<span class="token punctuation">,</span>
    Item<span class="token punctuation">:</span> <span class="token punctuation">{</span>
      connectionId<span class="token punctuation">:</span> <span class="token punctuation">{</span> S<span class="token punctuation">:</span> event<span class="token punctuation">.</span>requestContext<span class="token punctuation">.</span>connectionId <span class="token punctuation">}</span>
    <span class="token punctuation">}</span>
  <span class="token punctuation">}</span><span class="token punctuation">;</span>

  DDB<span class="token punctuation">.</span><span class="token function">putItem</span><span class="token punctuation">(</span>putParams<span class="token punctuation">,</span> <span class="token keyword">function</span><span class="token punctuation">(</span>err<span class="token punctuation">,</span> data<span class="token punctuation">)</span> <span class="token punctuation">{</span>
    <span class="token function">callback</span><span class="token punctuation">(</span><span class="token keyword">null</span><span class="token punctuation">,</span> <span class="token punctuation">{</span>
      statusCode<span class="token punctuation">:</span> err <span class="token operator">?</span> <span class="token number">500</span> <span class="token punctuation">:</span> <span class="token number">200</span><span class="token punctuation">,</span>
      body<span class="token punctuation">:</span> err <span class="token operator">?</span> <span class="token string">"Failed to connect: "</span> <span class="token operator">+</span> JSON<span class="token punctuation">.</span><span class="token function">stringify</span><span class="token punctuation">(</span>err<span class="token punctuation">)</span> <span class="token punctuation">:</span> <span class="token string">"Connected"</span>
    <span class="token punctuation">}</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
  <span class="token punctuation">}</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
<span class="token punctuation">}</span><span class="token punctuation">;</span></code></pre> 
       <p>The <strong>onDisconnect</strong> function removes the record corresponding with the specified <em>connectionId</em> value.</p> 
       <div class="prism-show-language"><div class="prism-show-language-label" data-language="Js">Js</div></div><pre class=" language-js" data-language="Js"><code class=" language-js">exports<span class="token punctuation">.</span>handler <span class="token operator">=</span> <span class="token keyword">function</span><span class="token punctuation">(</span>event<span class="token punctuation">,</span> context<span class="token punctuation">,</span> callback<span class="token punctuation">)</span> <span class="token punctuation">{</span>
  <span class="token keyword">var</span> deleteParams <span class="token operator">=</span> <span class="token punctuation">{</span>
    TableName<span class="token punctuation">:</span> process<span class="token punctuation">.</span>env<span class="token punctuation">.</span>TABLE_NAME<span class="token punctuation">,</span>
    Key<span class="token punctuation">:</span> <span class="token punctuation">{</span>
      connectionId<span class="token punctuation">:</span> <span class="token punctuation">{</span> S<span class="token punctuation">:</span> event<span class="token punctuation">.</span>requestContext<span class="token punctuation">.</span>connectionId <span class="token punctuation">}</span>
    <span class="token punctuation">}</span>
  <span class="token punctuation">}</span><span class="token punctuation">;</span>

  DDB<span class="token punctuation">.</span><span class="token function">deleteItem</span><span class="token punctuation">(</span>deleteParams<span class="token punctuation">,</span> <span class="token keyword">function</span><span class="token punctuation">(</span>err<span class="token punctuation">)</span> <span class="token punctuation">{</span>
    <span class="token function">callback</span><span class="token punctuation">(</span><span class="token keyword">null</span><span class="token punctuation">,</span> <span class="token punctuation">{</span>
      statusCode<span class="token punctuation">:</span> err <span class="token operator">?</span> <span class="token number">500</span> <span class="token punctuation">:</span> <span class="token number">200</span><span class="token punctuation">,</span>
      body<span class="token punctuation">:</span> err <span class="token operator">?</span> <span class="token string">"Failed to disconnect: "</span> <span class="token operator">+</span> JSON<span class="token punctuation">.</span><span class="token function">stringify</span><span class="token punctuation">(</span>err<span class="token punctuation">)</span> <span class="token punctuation">:</span> <span class="token string">"Disconnected."</span>
    <span class="token punctuation">}</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
  <span class="token punctuation">}</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
<span class="token punctuation">}</span><span class="token punctuation">;</span></code></pre> 
       <p>As I mentioned earlier, the <em>onDisconnect</em> function may not be called. To keep from retrying connections that are never going to be available again, add some additional logic in case the <em>postToConnection</em> call does not succeed:</p> 
       <div class="prism-show-language"><div class="prism-show-language-label" data-language="Js">Js</div></div><pre class=" language-js" data-language="Js"><code class=" language-js"><span class="token keyword">if</span> <span class="token punctuation">(</span>err<span class="token punctuation">.</span>statusCode <span class="token operator">===</span> <span class="token number">410</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
  console<span class="token punctuation">.</span><span class="token function">log</span><span class="token punctuation">(</span><span class="token string">"Found stale connection, deleting "</span> <span class="token operator">+</span> postParams<span class="token punctuation">.</span>connectionId<span class="token punctuation">)</span><span class="token punctuation">;</span>
  DDB<span class="token punctuation">.</span><span class="token function">deleteItem</span><span class="token punctuation">(</span><span class="token punctuation">{</span> TableName<span class="token punctuation">:</span> process<span class="token punctuation">.</span>env<span class="token punctuation">.</span>TABLE_NAME<span class="token punctuation">,</span>
                   Key<span class="token punctuation">:</span> <span class="token punctuation">{</span> connectionId<span class="token punctuation">:</span> <span class="token punctuation">{</span> S<span class="token punctuation">:</span> postParams<span class="token punctuation">.</span>connectionId <span class="token punctuation">}</span> <span class="token punctuation">}</span> <span class="token punctuation">}</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
<span class="token punctuation">}</span> <span class="token keyword">else</span> <span class="token punctuation">{</span>
  console<span class="token punctuation">.</span><span class="token function">log</span><span class="token punctuation">(</span><span class="token string">"Failed to post. Error: "</span> <span class="token operator">+</span> JSON<span class="token punctuation">.</span><span class="token function">stringify</span><span class="token punctuation">(</span>err<span class="token punctuation">)</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
<span class="token punctuation">}</span></code></pre> 
       <p>API Gateway returns a status of 410 GONE when the connection is no longer available. If this happens, delete the identifier from the DynamoDB table.</p> 
       <p>To make calls to your connected clients, your application needs a new permission: “execute-api:ManageConnections”. This is handled for you in the AWS SAM template.</p> 
       <h3>Deploy the WebSocket API</h3> 
       <p>The next step is to <a href="https://docs.aws.amazon.com/apigateway/latest/developerguide/how-to-deploy-api-with-console.html" target="_blank" rel="noopener">deploy the WebSocket API</a>. Because it’s the first time that you’re deploying the API, create a stage, such as “dev,” and give a sample description.</p> 
       <p>The <strong>Stage editor</strong> screen displays all the information for managing your WebSocket API, such as <strong>Settings</strong>, <strong>Logs/Tracing</strong>, <strong>Stage Variables</strong>, and <strong>Deployment History</strong>. Also, make sure that you check your limits and <a href="https://docs.aws.amazon.com/apigateway/latest/developerguide/api-gateway-request-throttling.html?icmpid=docs_apigateway_console" target="_blank" rel="noopener">read more about API Gateway throttling</a>.</p> 
       <p>Make sure that you monitor your tests using the dashboard available for your newly created WebSocket API.</p> 
       <p><img class="aligncenter wp-image-5800" src="https://d2908q01vomqb2.cloudfront.net/1b6453892473a467d07372d45eb05abc2031647a/2018/12/18/websockets-api-dashboard.png" alt="" width="750" height="334"></p> 
       <h3>Test the chat API</h3> 
       <p>To test the WebSocket API, you can use <a href="https://github.com/websockets/wscat" target="_blank" rel="noopener">wscat</a>, an open-source, command line tool.</p> 
       <ol> 
        <li><a href="https://www.npmjs.com/get-npm" target="_blank" rel="noopener">Install NPM</a>.</li> 
        <li>Install wscat: <div class="prism-show-language"><div class="prism-show-language-label" data-language="Bash">Bash</div></div><pre class=" language-bash" data-language="Bash"><code class=" language-bash">$ npm <span class="token function">install</span> <span class="token operator">-</span>g wscat</code></pre> </li> 
        <li>On the console, connect to your published API endpoint by executing the following command: <div class="prism-show-language"><div class="prism-show-language-label" data-language="Bash">Bash</div></div><pre class=" language-bash" data-language="Bash"><code class=" language-bash">$ wscat <span class="token operator">-</span>c wss<span class="token punctuation">:</span><span class="token operator">/</span><span class="token operator">/</span><span class="token punctuation">{</span>YOUR<span class="token operator">-</span>API<span class="token operator">-</span>ID<span class="token punctuation">}</span><span class="token punctuation">.</span>execute<span class="token operator">-</span>api<span class="token punctuation">.</span><span class="token punctuation">{</span>YOUR<span class="token operator">-</span>REGION<span class="token punctuation">}</span><span class="token punctuation">.</span>amazonaws<span class="token punctuation">.</span>com<span class="token operator">/</span><span class="token punctuation">{</span>STAGE<span class="token punctuation">}</span></code></pre> </li> 
        <li>To test the sendMessage function, send a JSON message like the following example. The Lambda function sends it back using the callback URL: <div class="prism-show-language"><div class="prism-show-language-label" data-language="Bash">Bash</div></div><pre class=" language-bash" data-language="Bash"><code class=" language-bash">$ wscat <span class="token operator">-</span>c wss<span class="token punctuation">:</span><span class="token operator">/</span><span class="token operator">/</span><span class="token punctuation">{</span>YOUR<span class="token operator">-</span>API<span class="token operator">-</span>ID<span class="token punctuation">}</span><span class="token punctuation">.</span>execute<span class="token operator">-</span>api<span class="token punctuation">.</span><span class="token punctuation">{</span>YOUR<span class="token operator">-</span>REGION<span class="token punctuation">}</span><span class="token punctuation">.</span>amazonaws<span class="token punctuation">.</span>com<span class="token operator">/</span>dev
connected <span class="token punctuation">(</span>press CTRL<span class="token operator">+</span>C to quit<span class="token punctuation">)</span>
<span class="token operator">&gt;</span> <span class="token punctuation">{</span><span class="token string">"action"</span><span class="token punctuation">:</span><span class="token string">"sendmessage"</span><span class="token punctuation">,</span> <span class="token string">"data"</span><span class="token punctuation">:</span><span class="token string">"hello world"</span><span class="token punctuation">}</span>
<span class="token operator">&lt;</span> hello world</code></pre> </li> 
       </ol> 
       <p>If you run wscat in multiple consoles, each will receive the “hello world”.</p> 
       <p>You’re now ready to send messages and get responses back and forth from your WebSocket API!</p> 
       