---
description: Enable CloudWatch Contributer Insights in API Gateway using log full data.
---

# Enable Contributer Insights in API Gateway

![API Gateway Image](01-enable-contributor-insights-in-api-gateway-01.png)

1. Enter the API Gateway service, choose API to enable Contributer Insights and enter the stage menu.
2. Choose stage.
3. Enter `Logs/Tracing` menu.
4. Enable `Log full requests/responses data`.
5. Save changes.

![CloudWatch Insights Image](01-enable-contributor-insights-in-api-gateway-02.png)

1. Enter the CloudWatch service, and enter the `Contributer Insights` menu.
2. Start to create rule.

![Insights Rule Definition 1 Image](01-enable-contributor-insights-in-api-gateway-03.png)

1. Choose API Gateway log group.
2. Choose rule type. There are sample rules for API Gateway like HTTP method, IP, etc.

![Insights Rule Definition 2 Image](01-enable-contributor-insights-in-api-gateway-04.png)

1. Choose log format. Recommands `JSON`.
2. Add contribution key like `pettype`.
3. Choose aggregation type.
4. Click `Next`.

![Insights Rule Definition 3 Image](01-enable-contributor-insights-in-api-gateway-05.png)

1. Enter the rule name.
2. Click `Next`.

![Insights Rule Definition 4 Image](01-enable-contributor-insights-in-api-gateway-06.png)

Create rule.

![Finished to Create Insights Rule Image](01-enable-contributor-insights-in-api-gateway-07.png)

You should be able to see a graph in a few minutes time.

[Observability Workshop](https://catalog.workshops.aws/observability/en-US/contributorinsights/customquery)