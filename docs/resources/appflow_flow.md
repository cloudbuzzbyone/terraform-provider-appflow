---
subcategory: "AppFlow"
layout: "aws"
page_title: "AWS: aws_appflow_flow"
description: |-
  Provides an AppFlow Flow resource.
---

# 🇮🇱 Resource: aws_appflow_flow 🇮🇱

Provides an AppFlow flow resource.

### [Cloudbuzz by One: Contact Us](mailto://info@cloudbuzz.co.il)

[![](https://github.com/cloudbuzzbyone/terraform-provider-appflow/assets/110726427/99c0e37e-82fb-4df2-9099-d796cc313ebf)](mailto://info@cloudbuzz.co.il)

### 🇮🇱 [We Stand with Israel!](https://cloudbuzz.co.il) 🇮🇱

[![](https://github.com/cloudbuzzbyone/terraform-provider-appflow/assets/110726427/e7cd93a1-105e-4c43-b6ab-fe5029c92b2e)](https://cloudbuzz.co.il)

## Example Usage

```terraform
resource "aws_s3_bucket" "example_source" {
  bucket = "example-source"
}

data "aws_iam_policy_document" "example_source" {
  statement {
    sid    = "AllowAppFlowSourceActions"
    effect = "Allow"

    principals {
      type        = "Service"
      identifiers = ["appflow.amazonaws.com"]
    }

    actions = [
      "s3:ListBucket",
      "s3:GetObject",
    ]

    resources = [
      "arn:aws:s3:::example_source",
      "arn:aws:s3:::example_source/*",
    ]
  }
}


resource "aws_s3_bucket_policy" "example_source" {
  bucket = aws_s3_bucket.example_source.id
  policy = data.aws_iam_policy_document.example_source.json
}


resource "aws_s3_object" "example" {
  bucket = aws_s3_bucket.example_source.id
  key    = "example_source.csv"
  source = "example_source.csv"
}


resource "aws_s3_bucket" "example_destination" {
  bucket = "example-destination"
}


data "aws_iam_policy_document" "example_destination" {
  statement {
    sid    = "AllowAppFlowDestinationActions"
    effect = "Allow"

    principals {
      type        = "Service"
      identifiers = ["appflow.amazonaws.com"]
    }

    actions = [
      "s3:PutObject",
      "s3:AbortMultipartUpload",
      "s3:ListMultipartUploadParts",
      "s3:ListBucketMultipartUploads",
      "s3:GetBucketAcl",
      "s3:PutObjectAcl",
    ]

    resources = [
      "arn:aws:s3:::example_destination",
      "arn:aws:s3:::example_destination/*",
    ]
  }
}


resource "aws_s3_bucket_policy" "example_destination" {
  bucket = aws_s3_bucket.example_destination.id
  policy = data.aws_iam_policy_document.example_destination.json
}


resource "aws_appflow_flow" "example" {
  name = "example"

  flow_status = "active"

  source_flow_config {
    connector_type = "SAPOData"

    connector_profile_name = "ExampleConnectorName"

    source_connector_properties {
      sapo_data {
        object_path = "https://sap.foo.bar/sap/odata/EXAMPLE"

        pagination_config {
          max_page_size = 10000
        }
      }
    }
  }

  destination_flow_config {
    connector_type = "S3"

    destination_connector_properties {
      s3 {
        bucket_name   = aws_s3_bucket_policy.example_destination.bucket
        bucket_prefix = "dest"

        s3_output_format_config {
          prefix_config {
            file_type                   = "PARQUET"
            preserve_source_data_typing = true

            aggregation_config {
              aggregation_type = "None"
            }

            prefix_config {
              prefix_format = "DAY"
              prefix_type   = "PATH_AND_FILENAME"
            }
          }
        }
      }
    }
  }

  task {
    source_fields = [""]
    task_type     = "Map_all"

    connector_operator {
      s3 = "NO_OP"
    }
  }

  trigger_config {
    trigger_type = "Scheduled"
    trigger_properties {
      scheduled {
        schedule_expression = "rate(1days)"
        data_pull_mode      = "incremental"

        first_execution_from = null

        schedule_start_time = formatdate(
          "YYYY-MM-DD'T'hh:mm:ssZ",
          timeadd(timestamp(), "5m")
        )

        timezone = "Asia/Jerusalem"
      }
    }
  }

  lifecycle {
    ignore_changes = [
      name,
      kms_arn,
      task,
      trigger_config[0].trigger_properties[0].scheduled[0].schedule_start_time,
      source_flow_config
    ]
  }
}

```

## Argument Reference

This resource supports the following arguments:

- `name` - (Required) Name of the flow.
- `destination_flow_config` - (Required) A [Destination Flow Config](#destination-flow-config) that controls how Amazon AppFlow places data in the destination connector.
- `source_flow_config` - (Required) The [Source Flow Config](#source-flow-config) that controls how Amazon AppFlow retrieves data from the source connector.
- `task` - (Required) A [Task](#task) that Amazon AppFlow performs while transferring the data in the flow run.
- `trigger_config` - (Required) A [Trigger](#trigger-config) that determine how and when the flow runs.
- `description` - (Optional) Description of the flow you want to create.
- `kms_arn` - (Optional) ARN (Amazon Resource Name) of the Key Management Service (KMS) key you provide for encryption. This is required if you do not want to use the Amazon AppFlow-managed KMS key. If you don't provide anything here, Amazon AppFlow uses the Amazon AppFlow-managed KMS key.
- `tags` - (Optional) Key-value mapping of resource tags. If configured with a provider [`default_tags` configuration block](https://registry.terraform.io/providers/hashicorp/aws/latest/docs#default_tags-configuration-block) present, tags with matching keys will overwrite those defined at the provider-level.
- `tags_all` - Map of tags assigned to the resource, including those inherited from the provider [`default_tags` configuration block](https://registry.terraform.io/providers/hashicorp/aws/latest/docs#default_tags-configuration-block).
- `flow_status` - (Required) Specifies the operational state of the flow. Valid values:
  - `Active` - The flow operates based on its configuration. Scheduled flows run as per schedule, while event-triggered flows run upon detecting specified events. On-demand flows run only when manually triggered.
  - `Suspended` - Deactivates an active flow. Scheduled and event-triggered flows cease to run until reactivated, with no effect on on-demand flows.

### Destination Flow Config

- `connector_type` - (Required) Type of connector, such as Salesforce, Amplitude, and so on. Valid values are`S3` and `SAPOData`.
- `destination_connector_properties` - (Required) This stores the information that is required to query a particular connector. See [Destination Connector Properties](#destination-connector-properties) for more information.
- `api_version` - (Optional) API version that the destination connector uses.
- `connector_profile_name` - (Optional) Name of the connector profile. This name must be unique for each connector profile in the AWS account.

#### Destination Connector Properties

- `s3` - (Optional) Properties that are required to query Amazon S3. See [S3 Destination Properties](#s3-destination-properties) for more details.
- `sapo_data` - (Optional) Properties that are required to query SAPOData. See [SAPOData Destination Properties](#sapodata-destination-properties) for more details.

##### S3 Destination Properties

- `bucket_name` - (Required) Amazon S3 bucket name in which Amazon AppFlow places the transferred data.
- `bucket_prefix` - (Optional) Object key for the bucket in which Amazon AppFlow places the destination files.
- `s3_output_format_config` - (Optional) Configuration that determines how Amazon AppFlow should format the flow output data when Amazon S3 is used as the destination. See [S3 Output Format Config](#s3-output-format-config) for more details.

###### S3 Output Format Config

- `aggregation_config` - (Optional) Aggregation settings that you can use to customize the output format of your flow data. See [Aggregation Config](#aggregation-config) for more details.
- `file_type` - (Optional) File type that Amazon AppFlow places in the Amazon S3 bucket. Valid values are `CSV`, `JSON`, and `PARQUET`.
- `prefix_config` - (Optional) Determines the prefix that Amazon AppFlow applies to the folder name in the Amazon S3 bucket. You can name folders according to the flow frequency and date. See [Prefix Config](#prefix-config) for more details.
- `preserve_source_data_typing` - (Optional, Boolean) Whether the data types from the source system need to be preserved (Only valid for `Parquet` file type)

##### SAPOData Destination Properties

- `object_path` - (Required) Object path specified in the SAPOData flow destination.
- `error_handling_config` - (Optional) Settings that determine how Amazon AppFlow handles an error when placing data in the destination. See [Error Handling Config](#error-handling-config) for more details.
- `id_field_names` - (Optional) Name of the field that Amazon AppFlow uses as an ID when performing a write operation such as update or delete.
- `success_response_handling_config` - (Optional) Determines how Amazon AppFlow handles the success response that it gets from the connector after placing data. See [Success Response Handling Config](#success-response-handling-config) for more details.
- `write_operation` - (Optional) Possible write operations in the destination connector. When this value is not provided, this defaults to the `INSERT` operation. Valid values are `INSERT`, `UPSERT`, `UPDATE`, and `DELETE`.

###### Success Response Handling Config

- `bucket_name` - (Optional) Name of the Amazon S3 bucket.
- `bucket_prefix` - (Optional) Amazon S3 bucket prefix.

###### Aggregation Config

- `aggregation_type` - (Optional) Whether Amazon AppFlow aggregates the flow records into a single file, or leave them unaggregated. Valid values are `None` and `SingleFile`.

###### Prefix Config

- `prefix_format` - (Optional) Determines the level of granularity that's included in the prefix. Valid values are `YEAR`, `MONTH`, `DAY`, `HOUR`, and `MINUTE`.
- `prefix_type` - (Optional) Determines the format of the prefix, and whether it applies to the file name, file path, or both. Valid values are `FILENAME`, `PATH`, and `PATH_AND_FILENAME`.

###### Error Handling Config

- `bucket_name` - (Optional) Name of the Amazon S3 bucket.
- `bucket_prefix` - (Optional) Amazon S3 bucket prefix.
- `fail_on_first_destination_error` - (Optional, boolean) If the flow should fail after the first instance of a failure when attempting to place data in the destination.

### Source Flow Config

- `connector_type` - (Required) Type of connector, such as Salesforce, Amplitude, and so on. Valid values are `S3` and `SAPOData`.
- `source_connector_properties` - (Required) Information that is required to query a particular source connector. See [Source Connector Properties](#source-connector-properties) for details.
- `api_version` - (Optional) API version that the destination connector uses.
- `connector_profile_name` - (Optional) Name of the connector profile. This name must be unique for each connector profile in the AWS account.
- `incremental_pull_config` - (Optional) Defines the configuration for a scheduled incremental data pull. If a valid configuration is provided, the fields specified in the configuration are used when querying for the incremental data pull. See [Incremental Pull Config](#incremental-pull-config) for more details.

#### Source Connector Properties

- `s3` - (Optional) Information that is required for querying Amazon S3. See [S3 Source Properties](#s3-source-properties) for more details.
- `sapo_data` - (Optional) Information that is required for querying SAPOData as a flow source. See [SAPO Source Properties](#sapodata-source-properties) for more details.

##### S3 Source Properties

- `bucket_name` - (Required) Amazon S3 bucket name where the source files are stored.
- `bucket_prefix` - (Optional) Object key for the Amazon S3 bucket in which the source files are stored.
- `s3_input_format_config` - (Optional) When you use Amazon S3 as the source, the configuration format that you provide the flow input data. See [S3 Input Format Config](#s3-input-format-config) for details.

###### S3 Input Format Config

- `s3_input_file_type` - (Optional) File type that Amazon AppFlow gets from your Amazon S3 bucket. Valid values are `CSV` and `JSON`.

##### SAPOData Source Properties

- `object_path` - (Required) Object path specified in the SAPOData flow source.
- `pagination_config` - (Optional) The maximum number of records that Amazon AppFlow receives in each page of the response from your SAP application. see [SAPOData Pagination Config](#sapodata-pagination-config)

##### SAPOData Pagination Config

- `max_page_size` - (Optional) The maximum number of records that Amazon AppFlow receives in each page of
  the response from your SAP application. For transfers of OData records, the
  maximum page size is 3,000. For transfers of data that comes from an ODP
  provider, the maximum page size is 10,000.
  Defaults to 1,000.

#### Incremental Pull Config

- `datetime_type_field_name` - (Optional) Field that specifies the date time or timestamp field as the criteria to use when importing incremental records from the source.

### Task

- `source_fields` - (Required) Source fields to which a particular task is applied.
- `task_type` - (Required) Particular task implementation that Amazon AppFlow performs. Valid values are `Arithmetic`, `Filter`, `Map`, `Map_all`, `Mask`, `Merge`, `Passthrough`, `Truncate`, and `Validate`.
- `connector_operator` - (Optional) Operation to be performed on the provided source fields. See [Connector Operator](#connector-operator) for details.
- `destination_field` - (Optional) Field in a destination connector, or a field value against which Amazon AppFlow validates a source field.
- `task_properties` - (Optional) Map used to store task-related information. The execution service looks for particular information based on the `TaskType`. Valid keys are `VALUE`, `VALUES`, `DATA_TYPE`, `UPPER_BOUND`, `LOWER_BOUND`, `SOURCE_DATA_TYPE`, `DESTINATION_DATA_TYPE`, `VALIDATION_ACTION`, `MASK_VALUE`, `MASK_LENGTH`, `TRUNCATE_LENGTH`, `MATH_OPERATION_FIELDS_ORDER`, `CONCAT_FORMAT`, `SUBFIELD_CATEGORY_MAP`, and `EXCLUDE_SOURCE_FIELDS_LIST`.

#### Connector Operator

- `s3` - (Optional) Operation to be performed on the provided Amazon S3 source fields. Valid values are `PROJECTION`, `LESS_THAN`, `GREATER_THAN`, `BETWEEN`, `LESS_THAN_OR_EQUAL_TO`, `GREATER_THAN_OR_EQUAL_TO`, `EQUAL_TO`, `NOT_EQUAL_TO`, `ADDITION`, `MULTIPLICATION`, `DIVISION`, `SUBTRACTION`, `MASK_ALL`, `MASK_FIRST_N`, `MASK_LAST_N`, `VALIDATE_NON_NULL`, `VALIDATE_NON_ZERO`, `VALIDATE_NON_NEGATIVE`, `VALIDATE_NUMERIC`, and `NO_OP`.
- `sapo_data` - (Optional) Operation to be performed on the provided SAPOData source fields. Valid values are `PROJECTION`, `LESS_THAN`, `GREATER_THAN`, `CONTAINS`, `BETWEEN`, `LESS_THAN_OR_EQUAL_TO`, `GREATER_THAN_OR_EQUAL_TO`, `EQUAL_TO`, `NOT_EQUAL_TO`, `ADDITION`, `MULTIPLICATION`, `DIVISION`, `SUBTRACTION`, `MASK_ALL`, `MASK_FIRST_N`, `MASK_LAST_N`, `VALIDATE_NON_NULL`, `VALIDATE_NON_ZERO`, `VALIDATE_NON_NEGATIVE`, `VALIDATE_NUMERIC`, and `NO_OP`.

### Trigger Config

- `trigger_type` - (Required) Type of flow trigger. Valid values are `Scheduled`, `Event`, and `OnDemand`.
- `trigger_properties` - (Optional) Configuration details of a schedule-triggered flow as defined by the user. Currently, these settings only apply to the `Scheduled` trigger type. See [Scheduled Trigger Properties](#scheduled-trigger-properties) for details.

#### Scheduled Trigger Properties

The `trigger_properties` block only supports one attribute: `scheduled`, a block which in turn supports the following:

- `schedule_expression` - (Required) Scheduling expression that determines the rate at which the schedule will run, for example `rate(5minutes)`.
- `data_pull_mode` - (Optional) Whether a scheduled flow has an incremental data transfer or a complete data transfer for each flow run. Valid values are `Incremental` and `Complete`.
- `first_execution_from` - (Optional) Date range for the records to import from the connector in the first flow run. Must be a valid RFC3339 timestamp.
- `schedule_end_time` - (Optional) Scheduled end time for a schedule-triggered flow. Must be a valid RFC3339 timestamp.
- `schedule_offset` - (Optional) Optional offset that is added to the time interval for a schedule-triggered flow. Maximum value of 36000.
- `schedule_start_time` - (Optional) Scheduled start time for a schedule-triggered flow. Must be a valid RFC3339 timestamp.
- `timezone` - (Optional) Time zone used when referring to the date and time of a scheduled-triggered flow, such as `America/New_York`.

```terraform
resource "aws_appflow_flow" "example" {
  # ... other configuration ...

  trigger_config {
    scheduled {
      schedule_expression = "rate(1minutes)"
    }
  }
}
```

## Attribute Reference

This resource exports the following attributes in addition to the arguments above:

- `arn` - Flow's ARN.

## Import

In Terraform v1.5.0 and later, use an [`import` block](https://developer.hashicorp.com/terraform/language/import) to import AppFlow flows using the `arn`. For example:

```terraform
import {
  to = aws_appflow_flow.example
  id = "arn:aws:appflow:us-west-2:123456789012:flow/example-flow"
}
```

Using `terraform import`, import AppFlow flows using the `arn`. For example:

```console
% terraform import aws_appflow_flow.example arn:aws:appflow:us-west-2:123456789012:flow/example-flow
```
