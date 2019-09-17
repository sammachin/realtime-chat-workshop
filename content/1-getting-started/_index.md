+++
title = "Concepts"
chapter = false
weight = 1
+++

To complete this tutorial you will need an AWS account **If you don't already have an AWS account with Administrator access**: [create
one now](https://aws.amazon.com/getting-started/)

Before you begin, here are a couple of the concepts of a WebSocket API in API Gateway. The first is a new resource type called a&nbsp;<em>route</em>. A route describes how API Gateway should handle a particular type of client request, and includes a&nbsp;<em>routeKey</em> parameter, a value that you provide to identify the route.

A WebSocket API is composed of one or more routes. To determine which route a particular inbound request should use, you provide a&nbsp;<em>route selection expression</em>. The expression is evaluated against an inbound request to produce a value that corresponds to one of your route’s <em>routeKey</em> values.

There are three special <em>routeKey</em> values that API Gateway allows you to use for a route:

<ul> 
        <li><strong>$default</strong>—Used when the route selection expression produces a value that does not match any of the other route keys in your API routes. This can be used, for example, to implement a generic error handling mechanism.</li> 
        <li><strong>$connect</strong>—The associated route is used when a client first connects to your WebSocket API.</li> 
        <li><strong>$disconnect</strong>—The associated route is used when a client disconnects from your API. This call is made on a best-effort basis.</li> 
</ul> 

You are not required to provide routes for any of these special routeKey values.

