# redshift-spectrum-aws-test
Repository to describe how to test the redshift and spectrum aws

- First we create our IAM in aws to allow redshift access to bucket s3
```
[{
	"Version": "2012-10-17",
	"Statement": [{
		"Effect": "Allow",
		"Action": ["s3:Get*", "s3:List*"],
		"Resource": "arn:aws:s3:::MYBUCKETREDSHIFT/*"
	}]
}
```

- Now we must create our bucket s3
```
arn:aws:s3:::MYBUCKETREDSHIFT
```

- Creating external schema spectrum
```
create external schema if not exists TABLE_NAME
from data catalog
database 'TABLE_NAME'
region 'us-west-2'
iam_role 'arn:aws:iam::1111111111111111:role/IAMREDSHIFT'
create external database if not exists;
```
- Creating a table in redshift and inserting data

```
create table public.category_stage_internal (
        id smallint default 0,
        first_name varchar(50) default 'General',
        last_name varchar(50) default 'General',
        email varchar(50) default 'General',
        gender varchar(50) default 'General',
        ip_address varchar(20) default 'General');
	
psql -Ustage -p5439 -hdataproviders.XXXXXXXXXXXXXXX.us-east-2.redshift.amazonaws.com < category_stage_internal.sql
```

