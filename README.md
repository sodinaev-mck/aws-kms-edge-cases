# AWS KMS Edge cases

## Giving IAM principal access to secrets KMS

If there is a principal that needs to interact with a particular secret encrypted with a KMS key, ensure that user has necessary permissions OR is listed in KMS key's policy

```json
        {
            "Sid": "Allow use of the key",
            "Effect": "Allow",
            "Principal": {
                "AWS": [
                    "arn:aws:iam::012345678910:role/CoolChef",
                    "arn:aws:iam::012345678910:role/AnotherCoolChef",
                ]
            },
            "Action": [
                "kms:Encrypt",
                "kms:Decrypt",
                "kms:ReEncrypt*",
                "kms:GenerateDataKey*",
                "kms:DescribeKey"
            ],
            "Resource": "*"
        }
```

## S3 bucket cross account access to encrypted objects

Ref: https://repost.aws/knowledge-center/cross-account-access-denied-error-s3

- The bucket policy in Account A must grant access to Account B.
- The AWS KMS key policy in Account A must grant access to the user in Account B.
- The AWS Identity and Access Management (IAM) policy in Account B must grant user access to the bucket and the AWS KMS key in Account A.

KMS keys must grant access to Account B
```json
{
  "Sid": "Allow use of the key",
  "Effect": "Allow",
  "Principal": {
    "AWS": [
      "arn:aws:iam::111122223333:role/role_name"
    ]
  },
  "Action": [
    "kms:Decrypt"
  ],
  "Resource": "*"
}
```


Account B must grant access to user

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ExampleStmt1",
      "Action": [
        "s3:GetObject"
      ],
      "Effect": "Allow",
      "Resource": "arn:aws:s3:::DOC-EXAMPLE-BUCKET/*"
    },
    {
      "Sid": "ExampleStmt2",
      "Action": [
        "kms:Decrypt"
      ],
      "Effect": "Allow",
      "Resource": "arn:aws:kms:us-west-2:444455556666:key/1234abcd-12ab-34cd-56ef-1234567890ab"
    }
  ]
}
```

## Granting AutoScaling service linked role (SLR) access to EBS encryption key

Consider there is a KMS key used for encrypted EBS volumes, and key has a policy that allows creating grants for AWS service. Following Terraform code can be used to allow, let say AutoScaling service linked role, access to that KMS key

```hcl

data "aws_iam_role" "asg" {
    name = "AWSServiceRoleForAutoScaling"
}

resource "aws_kms_grant" "a" {
  name              = ""
  key_id            = aws_kms_key.a.key_id
  grantee_principal = data.aws_iam_role.asg.arn
  operations        = ["Encrypt", "Decrypt", "GenerateDataKey"]
}
```

## Encrypting S3 for ELB logging

Must use `SSE-S3` key