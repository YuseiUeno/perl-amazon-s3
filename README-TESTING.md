# Testing This Module

From the original documentation for `Net::Amazon::S3`...

>Testing S3 is a tricky thing. Amazon wants to charge you a bit of
money each time you use their service. And yes, testing counts as
using. Because of this, the application's test suite by default, skips anything
approaching a real test.

I'm not so sure exactly how expensive creating a bucket and then
reading and writing a few bytes from S3 really is nowadays. In any
event, by default, the tests that actually create buckets and objects
will not be executed unless you set the environment variable
`AMAZON_S3_EXPENSIVE_TESTS` to some value. Testing can be controlled
with additional environment variables described below.

| Variable | Description |
| -------- | ----------- |
| `AMAZON_S3_EXPENSIVE_TESTS` | Doesn't matter what you set it to. Just has to be set |
| `AWS_ACCESS_KEY_ID` | Your AWS access key |
| `AWS_ACCESS_KEY_SECRET` | Your AWS sekkr1t passkey. Be forewarned that setting this environment variable on a shared system might leak that information to another user. Be careful. |
| `AWS_SESSION_TOKEN` |  Optional session token. |
| `S3_HOST` | Defaults to s3.amazonaws.com.  Set this for example if you want to test module against an API compatible service like minio. |
| `AMAZON_S3_SKIP_ACL_TESTS` |  Doesn't matter what you set it to. Just has to be set if you want to skip ACLs tests. |
| `AMAZON_S3_SKIP_REGION_CONSTRAINT_TEST` |  Doesn't matter what you set it to. Just has to be set if you want to skip region constraint test. |
| `AMAZON_S3_MINIO` | Doesn't matter what you set it to. Just has to be set if you want to skip tests that would fail on minio. |
| `AMAZON_S3_LOCALSTACK` | Doesn't matter what you set it to. Just has to be set if you want to skip tests that would fail on minio. |
| `AMAZON_S3_REGIONS` | Comma delimited list of regions to test |

__CAUTION__

__In order to test ACLs, the test will create a public bucket and then
make the bucket private. The test will perform the same kind of tests
on objects. The test will also delete the bucket and the objects as
well, however, stuff happens and you may be left with a public bucket
or object should these tests fail.__

__Check your account to make sure the buckets and objects have been
deleted. The bucket name will be have a prefix of
`net-amazon-s3-test-` and a suffix of your `AWS_ACCESS_KEY_ID`.__

# Regional Constraints

One of the original unit tests for this module attempted to create a
bucket in the EU region to ostensibly test regional constraints and
DNS based bucket names.

The test would create a bucket in the default region, delete the
bucket, then attempt to create a bucket with the same name in a
different region.  Today, this will fail consistently with a 409 error
(Operation Aborted).  This is due to the fact that you cannot
immediately reclaim a bucket name after deletion as it may takes some
time to free that bucket name in all regions.

[Bucket restrictions and limitations](https://docs.aws.amazon.com/AmazonS3/latest/userguide/BucketRestrictions.html)

To test regional constraints then, the current test will change the
name of the bucket if it encounters a 409 error while creating the
bucket.  The test will then proceed to read the ACLs to determine if
the constraint was successful. By default, the tests will only create
bucket in the default region (but it will check that the constraint is
in place). If you want to test creation of buckets in alternate
regions in addition to testing in the default region, set the
environment variable `AMAZON_S3_REGIONS` to one or more comma
separated regions.

```
cd src/main/perl
make test AMAZON_S3_EXPENSIVE_TESTS=1 AMAZON_S3_REGIONS='eu-west-1'
```

# Using S3 Mocking Services

If you want to test *some* parts of this module but don't want to
spend a few pennies (or don't have access to AWS credentials) you can
try one of the S3 mocking services.  The two most popular services
seem to be:

* [LocalStack](https://localstack.io)
* [minio](https://min.io)

Both of these implement a subset of the S3 API. __Note that Some tests will fail
on both services (as of the writing of this document).__ To make it
through the tests, try setting one or more of the environment
variables above which will selectively skip some test.
