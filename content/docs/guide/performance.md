---
title: "Performance"
date: 2020-02-21T18:43:01Z
lastmod: 2020-07-11
weight: 110
draft: false
---

## Reuse Connections with Keep-Alive

By default, the default Node.js HTTP/HTTPS agent creates a new TCP connection for every new request. To avoid the cost of establishing a new connection, you can reuse an existing connection.

The easiest way to configure SDK for JavaScript to reuse TCP connections is to set the `AWS_NODEJS_CONNECTION_REUSE_ENABLED` environment variable to `1`.

Using Serverless Framework, this can be set as an environment variable in the `serverless.yml` file.

{{< codeblock "language-yaml" >}}
  environment:
    AWS_NODEJS_CONNECTION_REUSE_ENABLED	: "1"
{{< /codeblock  >}}

Running a test for 5 mins using `artillery`, CloudWatch Insights produced the following with no keep-alive set:

| Max Duration  | Average Duration | Min Duration  | 95th Percentile |
| ------------- | ---------------- | ------------- | --------------- |
| 439.89ms      | 78.2102ms        | 52.35ms       | 115.182ms       |

When keep-alive is turned on, this was reduced to:

| Max Duration  | Average Duration | Min Duration  | 95th Percentile |
| ------------- | ---------------- | ------------- | --------------- |
| 334.15ms      | 34.3112ms	       | 24.06ms       | 49.0142ms       |





{{< spacer >}}