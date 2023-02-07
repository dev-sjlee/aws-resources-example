# ServiceAccount

``` yaml title="service-account"
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-service-account
  namespace: default
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::$account_id:role/my-role
```