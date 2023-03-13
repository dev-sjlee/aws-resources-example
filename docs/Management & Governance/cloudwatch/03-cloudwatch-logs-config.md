---
title: CloudWatch Logs Config
description: Config CloudWatch Logs log group automatically.
---

# CloudWatch Logs Config

``` py linenums="1" hl_lines="4 5 6 7"
import boto3
import json

REGION = ''
KMS_KEY_ARN = ''
RETENTION = 90
PROJECT_NAME = ''

client = boto3.session.Session(region_name=REGION).client('logs')
response = client.describe_log_groups()
log_groups = [item for item in response['logGroups'] if not item.get('kmsKeyId', None) or not item.get('retentionInDays', None)]

kms_policy = {
    'Effect': 'Allow',
    'Principal': {
        'Service': f'logs.{REGION}.amazonaws.com'
    },
    'Action': [
        'kms:Encrypt*',
        'kms:Decrypt*',
        'kms:ReEncrypt*',
        'kms:GenerateDataKey*',
        'kms:Describe*'
    ],
    'Resource': '*',
    'Condition': {
        'ForAnyValue:ArnEquals': {
            'kms:EncryptionContext:aws:logs:arn': [item['arn'][:-2] for item in log_groups]
        }
    }
}

print('\nKMS Key Policy\n')

print(json.dumps(kms_policy, indent=4))

start = input('Config Log Groups? (Y/n) ')

if start == 'Y' or start == 'y' or start == '':
    for log in log_groups:
        try:
            try:
                client.delete_log_group(logGroupName=log['logGroupName'])
        
            except client.exceptions.ResourceNotFoundException:
                pass
            
            client.create_log_group(
                logGroupName=log['logGroupName'],
                kmsKeyId=KMS_KEY_ARN,
                tags={
                    'project': PROJECT_NAME
                }
            )
            client.put_retention_policy(
                logGroupName=log['logGroupName'],
                retentionInDays=RETENTION
            )

            print(f'SUCCESS! {log["logGroupName"]}')
        
        except client.exceptions.ResourceAlreadyExistsException:
            print(f'ResourceAlreadyExistsException {log["logGroupName"]}')


else:
    exit
```