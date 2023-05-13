---
title: Delivery to OpenSearch Serverless
description: Delivery streaming data to Amazon OpenSearch Serverless.
---

# Delivery to OpenSearch Serverless

## Create OpenSearch Serverless Collection

### Create OpenSeach Serverless VPC Endpoint

![Create OpenSearch Serverless VPC Endpoint in AWS Console](../../assets/images/firehose/delivery-to-opensearch-serverless/1.png)

Choose private or protected subnet and collection's security group.

### Create OpenSearch Serverless Collection

![Create OpenSearch Serverless Collection in AWS Console - Collection details](../../assets/images/firehose/delivery-to-opensearch-serverless/2.png)

Type the correct collection name and type.

![Create OpenSearch Serverless Collection in AWS Console - Encryption](../../assets/images/firehose/delivery-to-opensearch-serverless/3.png)

Choose KMS key type.

![Create OpenSearch Serverless Collection in AWS Console - Network access settings](../../assets/images/firehose/delivery-to-opensearch-serverless/4.png)

Choose the VPC endpoint which you created in [here](#create-openseach-serverless-vpc-endpoint).

