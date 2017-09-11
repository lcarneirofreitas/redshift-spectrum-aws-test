# redshift-spectrum-aws-test
Repository to describe how to test the redshift and spectrum aws

- First we create our IAM in aws to allow redshift access to bucket s3

[{
	"Version": "2012-10-17",
	"Statement": [{
		"Effect": "Allow",
		"Action": ["s3:Get*", "s3:List*"],
		"Resource": "arn:aws:s3:::MYBUCKETREDSHIFT/*"
	}]
}
