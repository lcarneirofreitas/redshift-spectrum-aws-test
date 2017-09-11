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
s3://MYBUCKETREDSHIFT
```

- Creating external schema spectrum
```
create external schema if not exists stage
from data catalog
database 'stage'
region 'us-west-2'
iam_role 'arn:aws:iam::1111111111111111:role/IAMREDSHIFT'
create external database if not exists;
```
- Validate if the external schema was created
```
stage=> \x
Expanded display is on.

stage=> select * from svv_external_tables;
-[ RECORD 1 ]-----+-----------------------------------------------------------
schemaname        | stage
tablename         | category_stage_external
location          | s3://MYBUCKETREDSHIFT/stage
input_format      | org.apache.hadoop.mapred.TextInputFormat
output_format     | org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat
serialization_lib | org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe
serde_parameters  | {"field.delim":",","serialization.format":","}
compressed        | 0
parameters        | {"EXTERNAL":"TRUE","transient_lastDdlTime":"XXXXXXXXXXXX"}
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
```

```
psql -UUSERNAMEDB -p5439 -hdataproviders.XXXXXXXXXXXXXXX.us-east-2.redshift.amazonaws.com < category_stage_internal.sql
```

- Validate data in redshift table
```
select count(*) from public.category_stage_internal;
 count
-------
  1000
(1 row)
```

- Create external table spectrum
```
create external table stage.category_stage_external
(id smallint,
first_name varchar(50),
last_name varchar(50),
email varchar(50),
gender varchar(50),
ip_address varchar(20))
row format delimited fields terminated BY ','
stored as textfile
location 's3://MYBUCKETREDSHIFT/stage/';
```

- Loading data from redshift to the s3 spectrum
```
unload  ('select * from category_stage_internal order by id')
to 's3://MYBUCKETREDSHIFT/stage/'
iam_role 'arn:aws:iam::1111111111111111:role/IAMREDSHIFT'
manifest
allowoverwrite
maxfilesize 100 Mb
delimiter ',';
```

