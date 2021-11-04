## `kinesis-client-buildpack` for Heroku

This is a [Heroku buildpack][0] that pulls in JAR files used to make use the Amazon [Kinesis Client Library][1].  Since Java 8 is required to make use of those JAR files, this buildpack is dependent on the official [Heroku JVM buildpack][2].

[0]: http://devcenter.heroku.com/articles/buildpacks
[1]: http://docs.aws.amazon.com/kinesis/latest/dev/developing-consumers-with-kcl.html
[2]: https://github.com/heroku/heroku-buildpack-jvm-common

## Compatability

This buildpack is compatible with [the `aws-kclrb` gem][https://rubygems.org/gems/aws-kclrb] and uses the [aws-kcl-runner gem provided by Grayscale](https://github.com/Grayscale-Labs/amazon-kinesis-client-ruby/tree/grayscale-patches) to install the appropriate JAR files.

## Usage

Since the JAR files in the buildpack have to correspond with the gem version, you probably want to pin buildpack versions you want to use when adding the buildpack:

    $ heroku buildpacks:add --index=1 heroku/jvm
    $ heroku buildpacks:add --index=2 https://github.com/Grayscale-Labs/kinesis-client-buildpack.git

    $ heroku buildpacks -a my-ruby-app
    === my-ruby-app Buildpack URLs
    1. heroku/jvm
    2. https://github.com/Grayscale-Labs/kinesis-client-buildpack.git
    3. heroku/ruby

## Details

The jar list provided by the aws-kcl-runner gem is the most interesting part of this buildpack.

You can find the latest version of the `amazon-kinesis-client` in its [Maven repository listing][4].

[4]: https://mvnrepository.com/artifact/com.amazonaws/amazon-kinesis-client

Once a new project is created with this file, the dependencies can be
downloaded to a directory.  From the project root, one can simply:

```
mkdir jars
mvn dependency:copy-dependencies -DoutputDirectory=jars
```

The group ID, artifact ID and versions for each can be extracted using `mvn`:

```
mvn dependency:list
```

The result shows all dependencies and their versions, e.g.:

```
com.amazonaws:aws-java-sdk-kinesis:jar:1.11.171:compile
com.fasterxml.jackson.core:jackson-databind:jar:2.6.7.1:compile
com.amazonaws:aws-java-sdk-kms:jar:1.11.171:compile
com.fasterxml.jackson.core:jackson-core:jar:2.6.7:compile
com.fasterxml.jackson.dataformat:jackson-dataformat-cbor:jar:2.6.7:compile
commons-codec:commons-codec:jar:1.9:compile
com.amazonaws:aws-java-sdk-cloudwatch:jar:1.11.171:compile
com.amazonaws:amazon-kinesis-client:jar:1.8.1:compile
com.amazonaws:aws-java-sdk-dynamodb:jar:1.11.171:compile
com.google.guava:guava:jar:18.0:compile
joda-time:joda-time:jar:2.8.1:compile
commons-lang:commons-lang:jar:2.6:compile
commons-logging:commons-logging:jar:1.1.3:compile
software.amazon.ion:ion-java:jar:1.0.2:compile
com.amazonaws:aws-java-sdk-s3:jar:1.11.171:compile
com.amazonaws:aws-java-sdk-core:jar:1.11.171:compile
org.apache.httpcomponents:httpclient:jar:4.5.2:compile
org.apache.httpcomponents:httpcore:jar:4.4.4:compile
com.fasterxml.jackson.core:jackson-annotations:jar:2.6.0:compile
com.amazonaws:jmespath-java:jar:1.11.171:compile
com.google.protobuf:protobuf-java:jar:2.6.1:compile
```

This listing can then be split out to update the list of jars and versions that should be updated in the aws-kcl-runner rake task.

## Testing

If you want to modify the buildpack (probably the `compile` file) and test it, you can run it on a development machine:

```
mkdir tmp
STACK=heroku-16 ./bin/compile tmp tmp
```
