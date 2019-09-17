+++
title = "Test the chat API"
chapter = false
weight = 5
+++

To test the WebSocket API, you can use <a href="https://github.com/websockets/wscat" target="_blank" rel="noopener">wscat</a>, an open-source, command line tool.

<a href="https://www.npmjs.com/get-npm" target="_blank" rel="noopener">Install NPM</a>.

Install wscat: 

```
npm install -g wscat
```

On the console, connect to your published API endpoint by executing the following command:

```
wscat -c wss://{YOUR-API-ID}.execute-api.{YOUR-REGION}.amazonaws.com/{STAGE}
```

To test the sendMessage function, send a JSON message like the following example. The Lambda function sends it back using the callback URL:

```
wscat -c wss://{YOUR-API-ID}.execute-api.{YOUR-REGION}.amazonaws.com/dev
connected (press CTRL+C to quit)
> {"action":"sendmessage", "data":"hello world"}
< hello world
```

If you run wscat in multiple consoles, each will receive the “hello world”.

You’re now ready to send messages and get responses back and forth from your WebSocket API!
