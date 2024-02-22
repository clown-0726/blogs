# AWS SQS, Boto3 and Python：带示例的完整指南

> 发布日期：2022-10-19 21:28:11

AWS Boto3 是 AWS 的 Python SDK。Boto3 可以直接在 Python 脚本中与 AWS 资源进行交互。在本教程中，我们将了解如何使用 Boto3 操作 AWS SQS。

## 先决条件
- Python3
- Boto3: Boto3 可以通过 pip 进行安装：`pip install boto3`
- AWS Credentials: 如果你没有 AWS credentials，这些[资源](https://docs.aws.amazon.com/sdk-for-java/v1/developer-guide/setup-credentials.html)可以帮助到你。

## 如何使用 Boto3 创建一个 SQS 队列？
我们将使用 Boto3 中的 `create_queue` 方法创建一个新的 SQS 队列。此方法中有一些重要参数：
- QueueName：将要创建的队列的名字
- Attributes：指定队列的属性值。一些常用属性包括：
  - DelaySeconds：消息在消费之前的延迟时间。
  - RedrivePolicy：指定死信队列功能。
  - VisibilityTimeout：队列的可见性时间（秒）。指特定消息对单个消费者的可见时间段。

在接下来的实例中，我们将要创建一个名为 my-new-queue 的队列，其 DelaySeconds 设置为 0， VisibilityTimeout 设置为 60 秒。

```python
def create_queue():
    sqs_client = boto3.client("sqs", region_name="us-west-2")
    response = sqs_client.create_queue(
        QueueName="my-new-queue",
        Attributes={
            "DelaySeconds": "0",
            "VisibilityTimeout": "60",  # 60 seconds
        }
    )
    print(response)
```

输出：
```
{'QueueUrl': 'https://us-west-2.queue.amazonaws.com/xxxx/my-new-queue', 'ResponseMetadata': {'RequestId': '91187b97-a624-5f94-988e-23c25057248f', 'HTTPStatusCode': 200, 'HTTPHeaders': {'x-amzn-requestid': '91187b97-a624-5f94-988e-23c25057248f', 'date': 'Fri, 18 Dec 2020 00:13:39 GMT', 'content-type': 'text/xml', 'content-length': '334'}, 'RetryAttempts': 0}}
```

## 如何得到 SQS 队列的 URL？
大多数 SQS API 都需要 QueueUrl，因此我们可以使用 `get_queue_url` 方法通过 `QueueName` 来检索队列的 url。

```python
def get_queue_url():
    sqs_client = boto3.client("sqs", region_name="us-west-2")
    response = sqs_client.get_queue_url(
        QueueName="my-new-queue",
    )
    return response["QueueUrl"]
```

输出：
```
https://us-west-2.queue.amazonaws.com/xxxx/my-new-queue
```

## 如何向 SQS 队列发送消息？
我们可以使用 Boto3 的 `send_message` 方法向 SQS 队列发送消息。使用此方法时需要记住的一些重要参数：
- QueueUrl：目标队列的 URL
- MessageBody：我们要发送的消息。消息需要序列化为 String。

```python
def send_message():
    sqs_client = boto3.client("sqs", region_name="us-west-2")

    message = {"key": "value"}
    response = sqs_client.send_message(
        QueueUrl="https://us-west-2.queue.amazonaws.com/xxx/my-new-queue",
        MessageBody=json.dumps(message)
    )
    print(response)
```

输出：
```
{'MD5OfMessageBody': '88bac95f31528d13a072c05f2a1cf371', 'MessageId': '2ce1541b-0472-4715-8375-f8a8587c16e9', 'ResponseMetadata': {'RequestId': '02a7b659-c044-5357-885e-ee0c398e24b0', 'HTTPStatusCode': 200, 'HTTPHeaders': {'x-amzn-requestid': '02a7b659-c044-5357-885e-ee0c398e24b0', 'date': 'Fri, 18 Dec 2020 00:27:54 GMT', 'content-type': 'text/xml', 'content-length': '378'}, 'RetryAttempts': 0}}
```

## 如何从 SQS 队列接收消息？
我们可以使用 Boto3 的 `receive_message` 方法丛 SQS 队列接收消息。使用此方法时需要记住的一些重要参数：
- QueueUrl：目标队列的 URL
- MaxNumberOfMessages：一次接收消息的最大数量。
- WaitTimeSeconds：等待消息到达队列的时间。用于消息的长轮询。

```python
def receive_message():
    sqs_client = boto3.client("sqs", region_name="us-west-2")
    response = sqs_client.receive_message(
        QueueUrl="https://us-west-2.queue.amazonaws.com/xxx/my-new-queue",
        MaxNumberOfMessages=1,
        WaitTimeSeconds=10,
    )

    print(f"Number of messages received: {len(response.get('Messages', []))}")

    for message in response.get("Messages", []):
        message_body = message["Body"]
        print(f"Message body: {json.loads(message_body)}")
        print(f"Receipt Handle: {message['ReceiptHandle']}")
```

输出：
```
Number of messages received: 1
Message body: {'key': 'value'}
```


## 如何从 SQS 队列删除消息？
重要的是，从 SQS 队列接收消息后，SQS 不会自动删除它。一旦 `VisibilityTimeout` 期限到期，任何其他消费者也可以检索相同的消息。为了确保没有其他使用者检索到相同的消息，需要在 `VisibilityTimeout` 时间段内将其删除。

我们可以使用 `delete_message` 从 SQS 队列中删除消息。
- 我们需要提供 `ReceiptHandle` 作为 `delete_message` 方法的参数。

```python
def delete_message(receipt_handle):
    sqs_client = boto3.client("sqs", region_name="us-west-2")
    response = sqs_client.delete_message(
        QueueUrl="https://us-west-2.queue.amazonaws.com/xxx/my-new-queue",
        ReceiptHandle=receipt_handle,
    )
    print(response)
```

输出：
```
{'ResponseMetadata': {'RequestId': 'd9a860cb-45ff-58ec-8232-389eb8d7c2c6', 'HTTPStatusCode': 200, 'HTTPHeaders': {'x-amzn-requestid': 'd9a860cb-45ff-58ec-8232-389eb8d7c2c6', 'date': 'Fri, 18 Dec 2020 00:42:16 GMT', 'content-type': 'text/xml', 'content-length': '215'}, 'RetryAttempts': 0}}
```

## 如何改变 SQS 队列的配置？
通常需要在创建队列后更改 SQS 属性，如 `VisiblityTimeout` 或 `DelaySeconds`。我们可以使用 `set_queue_attributes` 方法来更改属性。

```python
def change_queue_attributes():
    sqs_client = boto3.client("sqs", region_name="us-west-2")
    response = sqs_client.set_queue_attributes(
        QueueUrl="https://us-west-2.queue.amazonaws.com/734066626836/my-new-queue",
        Attributes={
            "DelaySeconds": "10",
            "VisibilityTimeout": "300",
        }
    )
    print(response)
```

输出：
```
{'ResponseMetadata': {'RequestId': '89461462-8566-5a06-b9e8-4c377b18016f', 'HTTPStatusCode': 200, 'HTTPHeaders': {'x-amzn-requestid': '89461462-8566-5a06-b9e8-4c377b18016f', 'date': 'Fri, 18 Dec 2020 00:47:59 GMT', 'content-type': 'text/xml', 'content-length': '225'}, 'RetryAttempts': 0}}
```

## 如何将 SQS 队列中的消息全部删除？
Boto3 提供了 `purge_queue` 方法从队列中删除所有的消息。

```python
def purge_queue():
    sqs_client = boto3.client("sqs", region_name="us-west-2")
    response = sqs_client.purge_queue(
        QueueUrl="https://us-west-2.queue.amazonaws.com/xxx/my-new-queue",
    )
    print(response)
```