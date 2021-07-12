<!-- note marker start -->
**NOTE**: This repo contains only the documentation for the private BoltsOps Pro repo code.
Original file: https://github.com/boltopspro/enable-aws-cloudtrail/blob/master/README.md
The docs are publish so they are available for interested customers.
For access to the source code, you must be a paying BoltOps Pro subscriber.
If are interested, you can contact us at contact@boltops.com or https://www.boltops.com

<!-- note marker end -->

# Enable AWS CloudTrail CloudFormation Blueprint

[![BoltOps Badge](https://img.boltops.com/boltops/badges/boltops-badge.png)](https://www.boltops.com)

![](https://img.boltops.com/boltopspro/blueprints/enable-aws-cloudtrail/cloudtrail-trail-3.png)

This blueprint can be used to enable CloudTrail in your AWS account.

* Enables MultiRegion trail by default, so you don't have to enable CloudTrail individually in each region.
* Creates a KMS key to encrypt the CloudTrail trail.
* Creates an S3 bucket with the proper IAM permissions to store the CloudTrail logs.
* Creates an S3 bucket to store the access logs.
* Creates a CloudWatch Log Group to store the CloudTrail logs for easier searching and compliance.
* Creates an SNS Topic you can subscribe to for notification. You enable this with `NotificationEmail` and `PublishToTopic` parameters.

Related Blueprints:

* [boltopspro/aws-config-aggregator](https://github.com/boltopspro-docs/aws-config-aggregator)
* [boltopspro/aws-config-bucket](https://github.com/boltopspro-docs/aws-config-bucket)
* [boltopspro/enable-aws-cloudtrail](https://github.com/boltopspro-docs/enable-aws-cloudtrail)
* [boltopspro/enable-aws-config](https://github.com/boltopspro-docs/enable-aws-config)
* [boltopspro/enable-aws-guardduty](https://github.com/boltopspro-docs/enable-guardduty)

## Usage

1. Add blueprint to Gemfile
2. Configure: configs/enable-aws-cloudtrail values
3. Deploy

## Add

Add the blueprint to your lono project's `Gemfile`.

```ruby
gem "enable-aws-cloudtrail", git: "git@github.com:boltopspro/enable-aws-cloudtrail.git"
```

## Configure

First you want to configure the `configs/enable-aws-cloudtrail` [config files](https://lono.cloud/docs/core/configs/).  You can use [lono seed](https://lono.cloud/reference/lono-seed/) to configure starter values quickly.

    LONO_ENV=development lono seed enable-aws-cloudtrail

For additional environments:

    LONO_ENV=production  lono seed enable-aws-cloudtrail

The generated files in `config/enable-aws-cloudtrail` folder look something like this:

    configs/enable-aws-cloudtrail/
    ├── params
    │   ├── development.txt
    │   └── production.txt
    └── variables
        ├── development.rb
        └── production.rb

Here's an example params file.

configs/enable-aws-cloudtrail/params/development.txt:

    # Parameter Group: Trail Configuration
    # EnableLogFileValidation=true
    # IncludeGlobalEvents=false
    # MultiRegion=true

    # Parameter Group: Delivery Notifications
    # NotificationEmail=
    # PublishToTopic=false

    # Parameter Group: AWS::KMS::Alias
    # AliasName=cloudtrail

    # Parameter Group: AWS::Logs::LogGroup
    # LogGroupName= # cloudtrail
    # RetentionInDays= # 7

## Deploy

Use the [lono cfn deploy](http://lono.cloud/reference/lono-cfn-deploy/) command to deploy. Example:

    LONO_ENV=development lono cfn deploy enable-aws-config
    LONO_ENV=production  lono cfn deploy enable-aws-config

## MultiRegion

By default, a MultiRegion region trail is created. This spares you from having to create a CloudTrail trail in each region.

### S3 Bucket DeletionPolicy

By default, both the S3 Buckets used for Storage and AccessLogs has a DeletionPolicy of `Retain`. This means that if you delete the stacks, the S3 Buckets will remain.  If you would like to have the buckets be deleted, use the `@deletion_policy` variable.  Example:

configs/enable-aws-cloudtrail/variables/development.rb:

```ruby
@deletion_policy = "Delete"
```

Note, with a DeletionPolicy of `Delete`, the stack will rollback when you try to delete it because the S3 Buckets are not empty. You must first:

1. Turn off Logging on the CloudTrail Configuration. You can do this on the CloudTrail Console.  There's a *Logging* ON Toggle switch on the upper-right hand corner.
2. Empty the S3 Buckets. You can this on the S3 Console. There's an *Empty* button on the upper-right hand corner.
3. Then you can delete CloudFormation stack cleanly.
