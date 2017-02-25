# SQS (Simple Queue Service)

Simplified message producer and message consumer in Ruby using AWS [SQS][sqs].

Also See:
* [SQS FAQ][sqs-faq]
* [AWS::SQS][aws-sqs] Ruby client docs.

## Setup

In AWS:

* Access Key + Secret with SQS permissions.
* Setup credentials file in ` ~/.aws/credentials`.
  * Profile needs `region`, `aws_access_key_id`, `aws_secret_access_key`.
  * Alternatively specify `AWS_REGION`, `AWS_ACCESS_API_KEY`, `AWS_SECRET_ACCESS_KEY` environment variables.
* Create a "standard" SQS queue, note the name (e.g, `my-test-queue`).
  * `Visibility Timeout` should be > 10 seconds, which is 2x the simulated work delay.

Locally:

```bash
$ git clone git@github.com:mendable/aws-experiments.git
$ cd aws-experiments/sqs
$ docker build -t sqs-experiment .
```

## Basic Producer

Producer publishes single messages to the queue using [Aws::SQS::Queue][aws-sqs-queue].

```bash
$ docker run --rm -it \
   -v $(pwd):/app:ro \
   -v ~/.aws/credentials:/root/.aws/credentials:ro \
   -e AWS_PROFILE=default \
   -e QUEUE_NAME=my-test-queue \
   sqs-experiment \
   ./bin/producer
```

## Basic Consumer

Consumer takes messages from the queue and removes them once processed. Uses [AWS::SQS::QueuePoller][polling-messages] utility class with long-polling and individual/non-batch message fetching.

Running the standard consumer as simple as:

```bash
$ docker run --rm -it \
   -v $(pwd):/app:ro \
   -v ~/.aws/credentials:/root/.aws/credentials:ro \
   -e AWS_PROFILE=default \
   -e QUEUE_NAME=my-test-queue \
   sqs-experiment \
   ./bin/consumer
```

## Failure Scenarios

The basic consumer optionally accepts an additional argument `--simulate-failures`. When specified, some messages be selected for continual retry and eventual failure.

To see this in action:
* Create an additional queue in AWS of the same type (Standard/FIFO).
  * Default Visibility Timeout = 0 for a non-processed queue.
* Configure the Dead Letter Queue settings in AWS for the main work queue.

[sqs]: https://aws.amazon.com/sqs
[sqs-faq]: https://aws.amazon.com/sqs/faqs/
[polling-messages]: https://aws.amazon.com/blogs/developer/polling-messages-from-a-amazon-sqs-queue/
[aws-sqs-queue]: http://docs.aws.amazon.com/sdkforruby/api/Aws/SQS/Queue.html
[aws-sqs]: http://docs.aws.amazon.com/sdkforruby/api/Aws/SQS.html
