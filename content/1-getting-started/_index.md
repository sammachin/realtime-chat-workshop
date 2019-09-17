+++
title = "Getting Started"
chapter = false
weight = 1
+++

{{% notice note %}}
You will need an AWS account **If you don't already have an AWS account with Administrator access**: [create
one now](https://aws.amazon.com/getting-started/)
{{% /notice %}}
       
You can build bidirectional communication applications using WebSocket APIs in <a href="https://aws.amazon.com/api-gateway/" target="_blank" rel="noopener">Amazon API Gateway</a> without having to provision and manage any servers.

HTTP-based APIs use a request/response model with a client sending a request to a service and the service responding synchronously back to the client. WebSocket-based APIs are bidirectional in nature. This means that a client can send messages to a service and services can independently send messages to its clients.

This bidirectional behavior allows for richer types of client/service interactions because services can push data to clients without a client needing to make an explicit request. WebSocket APIs are often used in real-time applications such as chat applications, collaboration platforms, multiplayer games, and financial trading platforms.

This post explains how to build a serverless, real-time chat application using WebSocket API and API Gateway.

<h2>Overview</h2> 

Historically, building WebSocket APIs required setting up fleets of hosts that were responsible for managing the persistent connections that underlie the WebSocket protocol. Now, with API Gateway, this is no longer necessary. API Gateway handles the connections between the client and service. It lets you build your business logic using HTTP-based backends such as AWS Lambda, Amazon Kinesis, or any other HTTP endpoint.

Before you begin, here are a couple of the concepts of a WebSocket API in API Gateway. The first is a new resource type called a&nbsp;<em>route</em>. A route describes how API Gateway should handle a particular type of client request, and includes a&nbsp;<em>routeKey</em> parameter, a value that you provide to identify the route.

A WebSocket API is composed of one or more routes. To determine which route a particular inbound request should use, you provide a&nbsp;<em>route selection expression</em>. The expression is evaluated against an inbound request to produce a value that corresponds to one of your route’s <em>routeKey</em> values.

There are three special <em>routeKey</em> values that API Gateway allows you to use for a route:

<ul> 
        <li><strong>$default</strong>—Used when the route selection expression produces a value that does not match any of the other route keys in your API routes. This can be used, for example, to implement a generic error handling mechanism.</li> 
        <li><strong>$connect</strong>—The associated route is used when a client first connects to your WebSocket API.</li> 
        <li><strong>$disconnect</strong>—The associated route is used when a client disconnects from your API. This call is made on a best-effort basis.</li> 
</ul> 

You are not required to provide routes for any of these special routeKey values.

