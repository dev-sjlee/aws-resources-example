# Service SSE-KMS Encrypted Content From S3 Using CloudFront

[AWS Blog](https://aws.amazon.com/ko/blogs/networking-and-content-delivery/serving-sse-kms-encrypted-content-from-s3-using-cloudfront/)

!!! danger
    **DO NOT USE OAI at CloudFront!!**

## Lambda@Edge functions

=== "Node.js (16.x)"
    ``` javascript
    const crypto = require('crypto');
    const emptyHash = 'e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855';
    const signedHeadersGeneric = 'host;x-amz-content-sha256;x-amz-date;x-amz-security-token';
    const signedHeadersCustomOrigin = 'host;x-amz-cf-id;x-amz-content-sha256;x-amz-date;x-amz-security-token';
    const { AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY, AWS_SESSION_TOKEN } = process.env;

    exports.handler = async (event) => {

        const request = event.Records[0].cf.request;

        let originType = '';
        if (request.origin.hasOwnProperty('s3'))
            originType = 's3';
        else if (request.origin.hasOwnProperty('custom'))
            originType = 'custom';
        else
            throw("Unexpected origin type. Expected 's3' or 'custom'. Got: " + JSON.stringify(request.origin));

        const sigv4Options = {
            method: request.method,
            path: request.origin[originType].path + request.uri,
            credentials: {
                accessKeyId: AWS_ACCESS_KEY_ID,
                secretAccessKey: AWS_SECRET_ACCESS_KEY,
                sessionToken: AWS_SESSION_TOKEN
            },
            host: request.headers['host'][0].value,
            xAmzCfId: event.Records[0].cf.config.requestId,
            originType: originType
        };

        const signature = signV4(sigv4Options);

        for(var header in signature){
            request.headers[header.toLowerCase()] = [{
                key: header,
                value: signature[header].toString()
            }];
        }

        return request;
    };


    const signV4 = (options) => {
        const region = options.host.split('.')[2];
        const date = (new Date()).toISOString().replace(/[:-]|\.\d{3}/g, '');
        let canonicalHeaders = '';
        let signedHeaders = '';
        if (options.originType == 's3') {
            canonicalHeaders = ['host:'+options.host, 'x-amz-content-sha256:'+emptyHash, 'x-amz-date:'+date, 'x-amz-security-token:'+options.credentials.sessionToken].join('\n');
            signedHeaders = signedHeadersGeneric;
        } else {
            canonicalHeaders = ['host:'+options.host, 'x-amz-cf-id:'+options.xAmzCfId, 'x-amz-content-sha256:'+emptyHash, 'x-amz-date:'+date, 'x-amz-security-token:'+options.credentials.sessionToken].join('\n');
            signedHeaders = signedHeadersCustomOrigin;
        }
        const canonicalURI = encodeRfc3986(encodeURIComponent(decodeURIComponent(options.path).replace(/\+/g, ' ')).replace(/%2F/g, '/'));
        const canonicalRequest = [options.method, canonicalURI, '', canonicalHeaders + '\n', signedHeaders,emptyHash].join('\n');
        const credentialScope = [date.slice(0, 8), region, 's3/aws4_request'].join('/');
        const stringToSign = ['AWS4-HMAC-SHA256', date, credentialScope, hash(canonicalRequest, 'hex')].join('\n');
        const signature = hmac(hmac(hmac(hmac(hmac('AWS4' + options.credentials.secretAccessKey, date.slice(0, 8)), region), "s3"), 'aws4_request'), stringToSign, 'hex');
        const authorizationHeader = ['AWS4-HMAC-SHA256 Credential=' + options.credentials.accessKeyId + '/' + credentialScope,'SignedHeaders=' + signedHeaders,'Signature=' + signature].join(', ');

        return {
            'Authorization': authorizationHeader,
            'X-Amz-Content-Sha256' : emptyHash,
            'X-Amz-Date': date,
            'X-Amz-Security-Token': options.credentials.sessionToken
        };
    }

    const encodeRfc3986 = (urlEncodedStr) => {
        return urlEncodedStr.replace(/[!'()*]/g, c => '%' + c.charCodeAt(0).toString(16).toUpperCase())
    }

    const hash = (string, encoding) => {
        return crypto.createHash('sha256').update(string, 'utf8').digest(encoding)
    }

    const hmac = (key, string, encoding) => {
        return crypto.createHmac('sha256', key).update(string, 'utf8').digest(encoding)
    }
    ```

=== "Python (3.9)"
    ``` python
    import os
    import json

    emptyHash = 'e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855';
    signedHeadersGeneric = 'host;x-amz-content-sha256;x-amz-date;x-amz-security-token';
    signedHeadersCustomOrigin = 'host;x-amz-cf-id;x-amz-content-sha256;x-amz-date;x-amz-security-token';
    AWS_ACCESS_KEY_ID = os.getenv('AWS_ACCESS_KEY_ID')
    AWS_SECRET_ACCESS_KEY = os.getenv('AWS_SECRET_ACCESS_KEY')
    AWS_SESSION_TOKEN = os.getenv('AWS_SESSION_TOKEN')

    def lambda_handler(event, context):
        request = event['Records'][0]['cf']['request'];

        originType = ''
        if hasattr(request['origin'], 's3'):
            originType = 's3
        elif hasattr(request['origin'], 'custom'):
            originType = 'custom'
        else:
            raise Exception('Unexpected origin type. Expected \'s3\' or \'custom\'. Got: ' + json.dumps(request['origin']))
        
        sigv4Options = {
            method: 
        }

        return {
            'statusCode': 200,
            'body': json.dumps('Hello from Lambda!')
        }
    ```