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

<div class="prism-show-language"><div class="prism-show-language-label" data-language="Json">Json</div></div><pre class=" language-json" data-language="Json"><code class=" language-json"><span class="token punctuation">{</span>
<span class="token property">"action"</span><span class="token operator">:</span> <span class="token string">"sendmessage"</span><span class="token punctuation">,</span>
“data”<span class="token operator">:</span> "Hello<span class="token punctuation">,</span> I am using WebSocket APIs in API Gateway.”
<span class="token punctuation">}</span></code></pre> 

To route messages based on the message content, the WebSocket API expects JSON-formatted messages with a property serving as a routing key.

<h3>Manage routes</h3> 

Now, configure your new API to respond to incoming requests. For this app, create three routes as described earlier. First, configure the <strong>sendmessage</strong> route with a Lambda integration.

<ol> 
<li>In the API Gateway console, under <strong>My Chat API</strong>, choose <strong>Routes</strong>.</li> 
<li>For <strong>New Route Key</strong>, enter <strong>sendmessage</strong> and confirm it.</li> 
</ol> 

Each route includes route-specific information such as its model schemas, as well as a reference to its target integration. With the <strong>sendmessage</strong> route created, <a href="https://docs.aws.amazon.com/apigateway/latest/developerguide/api-gateway-create-api-as-simple-proxy-for-lambda.html" target="_blank" rel="noopener">use a Lambda function as an integration</a> that is responsible for sending messages.

At this point, you’ve either deployed the AWS Serverless Application Repository backend or deployed it yourself using AWS SAM. In the Lambda console, look at the sendMessage function. This function receives the data sent by one of the clients, looks up all the currently connected clients, and sends the provided data to each of them. Here’s a snippet from the Lambda function:

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

When one of the clients sends a message with an action of “sendmessage,” this function scans the DynamoDB table to find all of the currently connected clients. For each of them, it makes a <em>postToConnection</em> call with the provided data.

To send messages properly, you must know the API to which your clients are connected. This means explicitly setting the SDK endpoint to point to your API.

What’s left is to properly track the clients that are connected to the chat room. For this, take advantage of two of the special <em>routeKey</em> values and implement routes for <strong>$connect</strong> and <strong>$disconnect</strong>.

The <strong>onConnect</strong> function inserts the <em>connectionId</em> value from <em>requestContext</em> to the DynamoDB table.

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

As I mentioned earlier, the <em>onDisconnect</em> function may not be called. To keep from retrying connections that are never going to be available again, add some additional logic in case the <em>postToConnection</em> call does not succeed:

<div class="prism-show-language"><div class="prism-show-language-label" data-language="Js">Js</div></div><pre class=" language-js" data-language="Js"><code class=" language-js"><span class="token keyword">if</span> <span class="token punctuation">(</span>err<span class="token punctuation">.</span>statusCode <span class="token operator">===</span> <span class="token number">410</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
console<span class="token punctuation">.</span><span class="token function">log</span><span class="token punctuation">(</span><span class="token string">"Found stale connection, deleting "</span> <span class="token operator">+</span> postParams<span class="token punctuation">.</span>connectionId<span class="token punctuation">)</span><span class="token punctuation">;</span>
DDB<span class="token punctuation">.</span><span class="token function">deleteItem</span><span class="token punctuation">(</span><span class="token punctuation">{</span> TableName<span class="token punctuation">:</span> process<span class="token punctuation">.</span>env<span class="token punctuation">.</span>TABLE_NAME<span class="token punctuation">,</span>
            Key<span class="token punctuation">:</span> <span class="token punctuation">{</span> connectionId<span class="token punctuation">:</span> <span class="token punctuation">{</span> S<span class="token punctuation">:</span> postParams<span class="token punctuation">.</span>connectionId <span class="token punctuation">}</span> <span class="token punctuation">}</span> <span class="token punctuation">}</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
<span class="token punctuation">}</span> <span class="token keyword">else</span> <span class="token punctuation">{</span>
console<span class="token punctuation">.</span><span class="token function">log</span><span class="token punctuation">(</span><span class="token string">"Failed to post. Error: "</span> <span class="token operator">+</span> JSON<span class="token punctuation">.</span><span class="token function">stringify</span><span class="token punctuation">(</span>err<span class="token punctuation">)</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
<span class="token punctuation">}</span></code></pre> 

API Gateway returns a status of 410 GONE when the connection is no longer available. If this happens, delete the identifier from the DynamoDB table.

To make calls to your connected clients, your application needs a new permission: “execute-api:ManageConnections”. This is handled for you in the AWS SAM template.