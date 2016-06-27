# AWSGetElastiCacheEndpointLambda
AWS Lambda function as Cloudformation Custom Resource to get ElastiCache Endpoint

# GetECEndpoint Lambda Function code

```
import botocore.session
import json

try:
    from urllib2 import HTTPError, build_opener, HTTPHandler, Request
except ImportError:
    from urllib.error import HTTPError
    from urllib.request import build_opener, HTTPHandler, Request

SUCCESS = 'SUCCESS'
FAILED = 'FAILED'

def send(event, context, response_status, reason=None, response_data=None, physical_resource_id=None):
    response_data = response_data or {}
    response_body = json.dumps(
        {
            'Status': response_status,
            'Reason': reason or 'See the details in CloudWatch Log Stream: ' + context.log_stream_name,
            'PhysicalResourceId': physical_resource_id or context.log_stream_name,
            'StackId': event['StackId'],
            'RequestId': event['RequestId'],
            'LogicalResourceId': event['LogicalResourceId'],
           'Data': response_data
        }
    )

    opener = build_opener(HTTPHandler)
    request = Request(event['ResponseURL'], data=response_body)
    request.add_header('Content-Type', '')
    request.add_header('Content-Length', len(response_body))
    request.get_method = lambda: 'PUT'
    try:
        response = opener.open(request)
        print('Status code: {}'.format(response.getcode()))
        print('Status message: {}'.format(response.msg))
        return True
    except HTTPError as exc:
        print('Failed executing HTTP request: {}'.format(exc.code))
        return False

session = botocore.session.get_session()

def lambda_handler(event,context):
    print 'EVENT: ', event
    client = session.create_client('elasticache', region_name=event['ResourceProperties']['Region'])
    endpoint = client.describe_cache_clusters(
        CacheClusterId=event['ResourceProperties']['RedisCluster'],
        ShowCacheNodeInfo=True
    )['CacheClusters'][0]['CacheNodes'][0]['Endpoint']['Address']
    print 'ENDPOINT: ', endpoint
    send(event, context, SUCCESS, None, {'Endpoint': endpoint})
    ```
