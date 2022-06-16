# NAME

Amazon::S3 - A portable client library for working with and
managing Amazon S3 buckets and keys.

# SYNOPSIS

    #!/usr/bin/perl
    use warnings;
    use strict;

    use Amazon::S3;
    
    use vars qw/$OWNER_ID $OWNER_DISPLAYNAME/;
    
    my $aws_access_key_id     = "Fill me in!";
    my $aws_secret_access_key = "Fill me in too!";
    
    my $s3 = Amazon::S3->new(
        {   aws_access_key_id     => $aws_access_key_id,
            aws_secret_access_key => $aws_secret_access_key,
            retry                 => 1
        }
    );
    
    my $response = $s3->buckets;
    
    # create a bucket
    my $bucket_name = $aws_access_key_id . '-net-amazon-s3-test';
    my $bucket = $s3->add_bucket( { bucket => $bucket_name } )
        or die $s3->err . ": " . $s3->errstr;
    
    # store a key with a content-type and some optional metadata
    my $keyname = 'testing.txt';
    my $value   = 'T';
    $bucket->add_key(
        $keyname, $value,
        {   content_type        => 'text/plain',
            'x-amz-meta-colour' => 'orange',
        }
    );
    
    # list keys in the bucket
    $response = $bucket->list
        or die $s3->err . ": " . $s3->errstr;
    print $response->{bucket}."\n";
    for my $key (@{ $response->{keys} }) {
          print "\t".$key->{key}."\n";  
    }

    # delete key from bucket
    $bucket->delete_key($keyname);
    
    # delete bucket
    $bucket->delete_bucket;

# DESCRIPTION

`Amazon::S3` provides a portable client interface to Amazon Simple
Storage System (S3).

_This module is rather dated. For a much more robust and modern
implementation of an S3 interface try `Net::Amazon::S3`.
`Amazon::S3` ostensibly was intended to be a drop-in replacement for
`Net:Amazon::S3` that "traded some performance in return for
portability". That statement is no longer accurate as
`Net::Amazon::S3` implements much more of the S3 API and may have
changed the interface in ways that might break your
applications. However, `Net::Amazon::S3` is today dependent on
`Moose` which may in fact level the playing field in terms of
performance penalties that may have been introduced by
`Amazon::S3`. YMMV, however, this module may still appeal to some
that favor simplicity of the interface and a lower number of
dependencies. Below is the original description of the module._

> Amazon S3 is storage for the Internet. It is designed to
> make web-scale computing easier for developers. Amazon S3
> provides a simple web services interface that can be used to
> store and retrieve any amount of data, at any time, from
> anywhere on the web. It gives any developer access to the
> same highly scalable, reliable, fast, inexpensive data
> storage infrastructure that Amazon uses to run its own
> global network of web sites. The service aims to maximize
> benefits of scale and to pass those benefits on to
> developers.
>
> To sign up for an Amazon Web Services account, required to
> use this library and the S3 service, please visit the Amazon
> Web Services web site at http://www.amazonaws.com/.
>
> You will be billed accordingly by Amazon when you use this
> module and must be responsible for these costs.
>
> To learn more about Amazon's S3 service, please visit:
> http://s3.amazonaws.com/.
>
> The need for this module arose from some work that needed
> to work with S3 and would be distributed, installed and used
> on many various environments where compiled dependencies may
> not be an option. [Net::Amazon::S3](https://metacpan.org/pod/Net::Amazon::S3) used [XML::LibXML](https://metacpan.org/pod/XML::LibXML)
> tying it to that specific and often difficult to install
> option. In order to remove this potential barrier to entry,
> this module is forked and then modified to use [XML::SAX](https://metacpan.org/pod/XML::SAX)
> via [XML::Simple](https://metacpan.org/pod/XML::Simple).

# LIMITATIONS

As noted this module is no longer a _drop-in_ replacement for
`Net::Amazon::S3` and has limitations that may make the use of this
module in your applications questionable. The list of limitations
below may not be complete.

- API Signing

    Making calls to AWS APIs requires that the calls be signed.  Amazon
    has added a new signing method (Signature Version 4) to increase
    security around their APIs.  This module continues to use the original
    signing method (Signature Version 2).

    **New regions after January 30, 2014 will only support Signature Version 4.**

    There has been some effort to add support of Signature Version 4
    however several method in this package may need significant
    refactoring and testing in order to support the new sigining method.

    - Signature Version 2

        [https://docs.aws.amazon.com/AmazonS3/latest/userguide/RESTAuthentication.html](https://docs.aws.amazon.com/AmazonS3/latest/userguide/RESTAuthentication.html)

    - Signature Version 4

        [https://docs.aws.amazon.com/AmazonS3/latest/API/sigv4-query-string-auth.html](https://docs.aws.amazon.com/AmazonS3/latest/API/sigv4-query-string-auth.html)

- New APIs

    This module does not support the myriad of new API method calls
    available for S3 since its original creation.

- Multipart Upload Support

    While there are undocumented methods for multipart uploads (used for
    files >5Gb), those methods have not been tested and may not in fact
    work today.

    For more information regarding multipart uploads visit the link below.

    [https://docs.aws.amazon.com/AmazonS3/latest/API/API\_CreateMultipartUpload.html](https://docs.aws.amazon.com/AmazonS3/latest/API/API_CreateMultipartUpload.html)

# METHODS AND SUBROUTINES

## new 

Create a new S3 client object. Takes some arguments:

- credentials (optional)

    Reference to a class (like `Amazon::Credentials`) that can provide
    credentials via the methods:

        get_aws_access_key_id()
        get_aws_secret_access_key()
        get_token()

    If you do not provide a credential class you must provide the keys
    when you instantiate the object. See below.

    _You are strongly encourage to use a class that provides getters. If
    you choose to provide your credentials to this class then they will be
    stored in this object. If you dump the class you will likely expose
    those credentials._

- aws\_access\_key\_id

    Use your Access Key ID as the value of the AWSAccessKeyId parameter
    in requests you send to Amazon Web Services (when required). Your
    Access Key ID identifies you as the party responsible for the
    request.

- aws\_secret\_access\_key 

    Since your Access Key ID is not encrypted in requests to AWS, it
    could be discovered and used by anyone. Services that are not free
    require you to provide additional information, a request signature,
    to verify that a request containing your unique Access Key ID could
    only have come from you.

    **DO NOT INCLUDE THIS IN SCRIPTS OR APPLICATIONS YOU
    DISTRIBUTE. YOU'LL BE SORRY.**

    _Consider using a credential class as described above to provide
    credentials, otherwise this class will store your credentials for
    signing the requests. If you dump this object to logs your credentials
    could be discovered._

- token

    An optional temporary token that will be inserted in the request along
    with your access and secret key.  A token is used in conjunction with
    temporary credentials when your EC2 instance has
    assumed a role and you've scraped the temporary credentials from
    _http://169.254.169.254/latest/meta-data/iam/security-credentials_

- secure

    Set this to a true value if you want to use SSL-encrypted connections
    when connecting to S3. Starting in version 0.49, the default is true.

    default: true

- timeout

    Defines the time, in seconds, your script should wait or a
    response before bailing.

    default: 30s

- retry

    Enables or disables the library to retry upon errors. This
    uses exponential backoff with retries after 1, 2, 4, 8, 16,
    32 seconds, as recommended by Amazon.

    default: off

- host

    Defines the S3 host endpoint to use.

    default: s3.amazonaws.com

    Note that requests are made to domain buckets when possible.  You can
    prevent that behavior if either the bucket name does conform to DNS
    bucket naming conventions or you preface the bucket name with '/'.

    If you set a region then the host name will be modified accordingly if
    it is an Amazon endpoint.

- region

    The AWS region you where your bucket is located.

    default: no region

- buffer\_size

    The default buffer size when reading or writing files.

    default: 4096

## buckets

Returns `undef` on error, else HASHREF of results:

- owner\_id

    The owner's ID of the buckets owner.

- owner\_display\_name

    The name of the owner account. 

- buckets

    Any ARRAYREF of [Amazon::SimpleDB::Bucket](https://metacpan.org/pod/Amazon::SimpleDB::Bucket) objects for the 
    account.

## add\_bucket 

Takes a HASHREF:

- bucket

    The name of the bucket you want to add

- acl\_short (optional)

    See the set\_acl subroutine for documenation on the acl\_short options

Returns 0 on failure or a [Amazon::S3::Bucket](https://metacpan.org/pod/Amazon::S3::Bucket) object on success

## bucket BUCKET

Takes a scalar argument, the name of the bucket you're creating

Returns an (unverified) bucket object from an account. This method does not access the network.

## delete\_bucket

Takes either a [Amazon::S3::Bucket](https://metacpan.org/pod/Amazon::S3::Bucket) object or a HASHREF containing 

- bucket

    The name of the bucket to remove

Returns false (and fails) if the bucket isn't empty.

Returns true if the bucket is successfully deleted.

## list\_bucket, list\_bucket\_v2

List all keys in this bucket.

Takes a HASHREF of arguments:

- bucket

    REQUIRED. The name of the bucket you want to list keys on.

- prefix

    Restricts the response to only contain results that begin with the
    specified prefix. If you omit this optional argument, the value of
    prefix for your query will be the empty string. In other words, the
    results will be not be restricted by prefix.

- delimiter

    If this optional, Unicode string parameter is included with your
    request, then keys that contain the same string between the prefix
    and the first occurrence of the delimiter will be rolled up into a
    single result element in the CommonPrefixes collection. These
    rolled-up keys are not returned elsewhere in the response.  For
    example, with prefix="USA/" and delimiter="/", the matching keys
    "USA/Oregon/Salem" and "USA/Oregon/Portland" would be summarized
    in the response as a single "USA/Oregon" element in the CommonPrefixes
    collection. If an otherwise matching key does not contain the
    delimiter after the prefix, it appears in the Contents collection.

    Each element in the CommonPrefixes collection counts as one against
    the MaxKeys limit. The rolled-up keys represented by each CommonPrefixes
    element do not.  If the Delimiter parameter is not present in your
    request, keys in the result set will not be rolled-up and neither
    the CommonPrefixes collection nor the NextMarker element will be
    present in the response.

    NOTE: CommonPrefixes isn't currently supported by Amazon::S3. 

- max-keys 

    This optional argument limits the number of results returned in
    response to your query. Amazon S3 will return no more than this
    number of results, but possibly less. Even if max-keys is not
    specified, Amazon S3 will limit the number of results in the response.
    Check the IsTruncated flag to see if your results are incomplete.
    If so, use the Marker parameter to request the next page of results.
    For the purpose of counting max-keys, a 'result' is either a key
    in the 'Contents' collection, or a delimited prefix in the
    'CommonPrefixes' collection. So for delimiter requests, max-keys
    limits the total number of list results, not just the number of
    keys.

- marker

    This optional parameter enables pagination of large result sets.
    `marker` specifies where in the result set to resume listing. It
    restricts the response to only contain results that occur alphabetically
    after the value of marker. To retrieve the next page of results,
    use the last key from the current page of results as the marker in
    your next request.

    See also `next_marker`, below. 

    If `marker` is omitted,the first page of results is returned. 

Returns `undef` on error and a HASHREF of data on success:

The HASHREF looks like this:

    {
          bucket       => $bucket_name,
          prefix       => $bucket_prefix, 
          marker       => $bucket_marker, 
          next_marker  => $bucket_next_available_marker,
          max_keys     => $bucket_max_keys,
          is_truncated => $bucket_is_truncated_boolean
          keys          => [$key1,$key2,...]
     }

Explanation of bits of that:

- is\_truncated

    B flag that indicates whether or not all results of your query were
    returned in this response. If your results were truncated, you can
    make a follow-up paginated request using the Marker parameter to
    retrieve the rest of the results.

- next\_marker 

    A convenience element, useful when paginating with delimiters. The
    value of `next_marker`, if present, is the largest (alphabetically)
    of all key names and all CommonPrefixes prefixes in the response.
    If the `is_truncated` flag is set, request the next page of results
    by setting `marker` to the value of `next_marker`. This element
    is only present in the response if the `delimiter` parameter was
    sent with the request.

Each key is a HASHREF that looks like this:

     {
        key           => $key,
        last_modified => $last_mod_date,
        etag          => $etag, # An MD5 sum of the stored content.
        size          => $size, # Bytes
        storage_class => $storage_class # Doc?
        owner_id      => $owner_id,
        owner_displayname => $owner_name
    }

## get\_logger

Returns the logger object. If you did not set a logger when you
created the object then the an instance of `Amazon::S3::Logger` is
returned. You can log to STDERR using this logger. For example:

    $s3->get_logger->debug('this is a debug message');

    $s3->get_logger->trace(sub { return Dumper([$response]) });

## list\_bucket\_all, list\_bucket\_all\_v2

List all keys in this bucket without having to worry about
'marker'. This is a convenience method, but may make multiple requests
to S3 under the hood.

Takes the same arguments as list\_bucket.

_You are encouraged to use the newer `list_bucket_all_v2` method._

## last\_response

Returns the last [HTTP::Response](https://metacpan.org/pod/HTTP::Response) object.

## last\_request

Returns the last [HTTP::Request](https://metacpan.org/pod/HTTP::Request) object.

## level

Set the logging level.

default: error

# ABOUT

This module contains code modified from Amazon that contains the
following notice:

    #  This software code is made available "AS IS" without warranties of any
    #  kind.  You may copy, display, modify and redistribute the software
    #  code either by itself or as incorporated into your code; provided that
    #  you do not remove any proprietary notices.  Your use of this software
    #  code is at your own risk and you waive any claim against Amazon
    #  Digital Services, Inc. or its affiliates with respect to your use of
    #  this software code. (c) 2006 Amazon Digital Services, Inc. or its
    #  affiliates.

# TESTING

Testing S3 is a tricky thing. Amazon wants to charge you a bit of 
money each time you use their service. And yes, testing counts as using.
Because of this, the application's test suite skips anything approaching 
a real test unless you set these environment variables:

- AMAZON\_S3\_EXPENSIVE\_TESTS

    Doesn't matter what you set it to. Just has to be set

- AWS\_ACCESS\_KEY\_ID 

    Your AWS access key

- AWS\_ACCESS\_KEY\_SECRET

    Your AWS sekkr1t passkey. Be forewarned that setting this environment variable
    on a shared system might leak that information to another user. Be careful.

- AMAZON\_S3\_SKIP\_ACL\_TESTS

    Doesn't matter what you set it to. Just has to be set if you want
    to skip ACLs tests.

- AMAZON\_S3\_SKIP\_REGION\_CONSTRAINT\_TEST

    Doesn't matter what you set it to. Just has to be set if you want
    to skip region constraint test.

- AMAZON\_S3\_MINIO

    Doesn't matter what you set it to. Just has to be set if you want
    to skip tests that would fail on minio.

- AMAZON\_S3\_LOCALSTACK

    Doesn't matter what you set it to. Just has to be set if you want
    to skip tests that would fail on LocalStack.

- AMAZON\_S3\_REGIONS

    A comma delimited list of regions to use for testing. The default will
    only test creating a bucket in the local region.

_Consider using an S3 mocking service like `minio` or `LocalStack`
if you want to create real tests for your applications or this module._

# ADDITIONAL INFORMATION

## LOGGING AND DEBUGGING

Additional debugging information can be output to STDERR by setting
the `level` option when you instantiate the `Amazon::S3`
object. Levels are represented as a string.  The valid levels are:

    fatal
    error
    warn
    info
    debug
    trace

You can set an optionally pass in a logger that implements a subset of
the `Log::Log4perl` interface.  Your logger should support at least
these method calls. If you do not supply a logger the default logger
(`Amazon::S3::Logger`) will be used.

    get_logger()
    fatal()
    error()
    warn()
    info()
    debug()
    trace()
    level()

At the `trace` level, every HTTP request and response will be output
to STDERR.  At the `debug` level information regarding the higher
level methods will be output to STDERR.  There currently is no
additional information logged at lower levels.

## S3 LINKS OF INTEREST

- [Bucket restrictions and limitations](https://docs.aws.amazon.com/AmazonS3/latest/userguide/BucketRestrictions.html)
- [Bucket naming rules](https://docs.aws.amazon.com/AmazonS3/latest/userguide/bucketnamingrules.html)
- [Amazon S3 REST API](https://docs.aws.amazon.com/AmazonS3/latest/API/Welcome.html)
- [Authenticating Requests (AWS Signature Version 4)](https://docs.aws.amazon.com/AmazonS3/latest/API/sig-v4-authenticating-requests.html)
- [Authenticating Requests (AWS Signature Version 2)](https://docs.aws.amazon.com/AmazonS3/latest/userguide/RESTAuthentication.html)

# SUPPORT

Bugs should be reported via the CPAN bug tracker at

&lt;http://rt.cpan.org/NoAuth/ReportBug.html?Queue=Amazon-S3>

For other issues, contact the author.

# AUTHOR

Original author: Timothy Appnel <tima@cpan.org>

Current maintainer: Rob Lauer <bigfoot@cpan.org>

# SEE ALSO

[Amazon::S3::Bucket](https://metacpan.org/pod/Amazon::S3::Bucket), [Net::Amazon::S3](https://metacpan.org/pod/Net::Amazon::S3)

# COPYRIGHT AND LICENCE

This module was initially based on [Net::Amazon::S3](https://metacpan.org/pod/Net::Amazon::S3) 0.41, by
Leon Brocard. Net::Amazon::S3 was based on example code from
Amazon with this notice:

_This software code is made available "AS IS" without warranties of any
kind.  You may copy, display, modify and redistribute the software
code either by itself or as incorporated into your code; provided that
you do not remove any proprietary notices.  Your use of this software
code is at your own risk and you waive any claim against Amazon
Digital Services, Inc. or its affiliates with respect to your use of
this software code. (c) 2006 Amazon Digital Services, Inc. or its
affiliates._

The software is released under the Artistic License. The
terms of the Artistic License are described at
http://www.perl.com/language/misc/Artistic.html. Except
where otherwise noted, `Amazon::S3` is Copyright 2008, Timothy
Appnel, tima@cpan.org. All rights reserved.
