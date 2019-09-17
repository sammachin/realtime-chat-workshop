+++
title = "Test the chat API"
chapter = false
weight = 5
+++

To test the WebSocket API, you can use <a href="https://github.com/websockets/wscat" target="_blank" rel="noopener">wscat</a>, an open-source, command line tool.

<ol> 
<li><a href="https://www.npmjs.com/get-npm" target="_blank" rel="noopener">Install NPM</a>.</li> 
<li>Install wscat: <div class="prism-show-language"><div class="prism-show-language-label" data-language="Bash">Bash</div></div><pre class=" language-bash" data-language="Bash"><code class=" language-bash">$ npm <span class="token function">install</span> <span class="token operator">-</span>g wscat</code></pre> </li> 
<li>On the console, connect to your published API endpoint by executing the following command: <div class="prism-show-language"><div class="prism-show-language-label" data-language="Bash">Bash</div></div><pre class=" language-bash" data-language="Bash"><code class=" language-bash">$ wscat <span class="token operator">-</span>c wss<span class="token punctuation">:</span><span class="token operator">/</span><span class="token operator">/</span><span class="token punctuation">{</span>YOUR<span class="token operator">-</span>API<span class="token operator">-</span>ID<span class="token punctuation">}</span><span class="token punctuation">.</span>execute<span class="token operator">-</span>api<span class="token punctuation">.</span><span class="token punctuation">{</span>YOUR<span class="token operator">-</span>REGION<span class="token punctuation">}</span><span class="token punctuation">.</span>amazonaws<span class="token punctuation">.</span>com<span class="token operator">/</span><span class="token punctuation">{</span>STAGE<span class="token punctuation">}</span></code></pre> </li> 
<li>To test the sendMessage function, send a JSON message like the following example. The Lambda function sends it back using the callback URL: <div class="prism-show-language"><div class="prism-show-language-label" data-language="Bash">Bash</div></div><pre class=" language-bash" data-language="Bash"><code class=" language-bash">$ wscat <span class="token operator">-</span>c wss<span class="token punctuation">:</span><span class="token operator">/</span><span class="token operator">/</span><span class="token punctuation">{</span>YOUR<span class="token operator">-</span>API<span class="token operator">-</span>ID<span class="token punctuation">}</span><span class="token punctuation">.</span>execute<span class="token operator">-</span>api<span class="token punctuation">.</span><span class="token punctuation">{</span>YOUR<span class="token operator">-</span>REGION<span class="token punctuation">}</span><span class="token punctuation">.</span>amazonaws<span class="token punctuation">.</span>com<span class="token operator">/</span>dev
connected <span class="token punctuation">(</span>press CTRL<span class="token operator">+</span>C to quit<span class="token punctuation">)</span>
<span class="token operator">&gt;</span> <span class="token punctuation">{</span><span class="token string">"action"</span><span class="token punctuation">:</span><span class="token string">"sendmessage"</span><span class="token punctuation">,</span> <span class="token string">"data"</span><span class="token punctuation">:</span><span class="token string">"hello world"</span><span class="token punctuation">}</span>
<span class="token operator">&lt;</span> hello world</code></pre> </li> 
</ol> 

If you run wscat in multiple consoles, each will receive the “hello world”.

You’re now ready to send messages and get responses back and forth from your WebSocket API!
