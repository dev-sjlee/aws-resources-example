---
title: CloudFront Caching Sequence
description: Aboutn CloudFront caching sequence with diagram.
---

# CloudFront Caching Sequence

## Using Amazon S3 to Origin

``` mermaid
sequenceDiagram
    autonumber
    Client->>Edge: Get object
    alt is already existed in Edge
        Edge->>Client: Serve object (2XX/3XX, Hit from cloudfront)
    else is not already existed in Edge
        Edge->>Origin: Get object
        alt is found in S3 bucket
            Origin->>Edge: Save object
            Edge->>Client: Serve object (2XX, Miss from cloudfront)
        else is not found in S3 bucket
            Origin->>Edge: Serve 4XX/5XX
            Edge->>Client: Serve Error (4XX/5XX, Error from cloudfront)
        end
    end
```