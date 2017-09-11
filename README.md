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
psql -UUSERNAMEDB -hdataproviders.XXXXXXXXXXXXXXX.us-east-2.redshift.amazonaws.com < category_stage_internal.sql
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

- Validate if the external table was created
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

- Data saved in s3 spectrum
```
aws s3 ls s3://MYBUCKETREDSHIFT/stage
2017-09-05 19:43:22        169 0000_part_00
2017-09-05 19:43:22        173 0001_part_00
2017-09-05 19:43:22        114 0002_part_00
2017-09-05 19:43:22        165 0003_part_00
2017-09-05 19:43:22        169 0004_part_00
2017-09-05 19:43:22        169 0005_part_00
2017-09-05 19:43:23        340 manifest
```

- Querying data in the two tables
```
stage=> select * from stage.category_stage_external where id=987;
 id  | first_name | last_name |          email           | gender |   ip_address
-----+------------+-----------+--------------------------+--------+----------------
 987 | Rasia      | Gertray   | rgertrayre@angelfire.com | Female | 234.121.81.127
(1 row)

stage=> select * from public.category_stage_internal where id=987;
 id  | first_name | last_name |          email           | gender |   ip_address
-----+------------+-----------+--------------------------+--------+----------------
 987 | Rasia      | Gertray   | rgertrayre@angelfire.com | Female | 234.121.81.127
(1 row)

stage=> select count(*) from public.category_stage_internal;
 count
-------
  1000
(1 row)

stage=> select count(*) from stage.category_stage_external;
 count
-------
  1010
(1 row)
```

- Deleting table data in redshift
```
stage=> delete from category_stage_internal;
DELETE 1000
stage=> select count(*) from public.category_stage_internal;
 count
-------
     0
(1 row)
```

- Now let's retrieve all data from the redshift table with the data from the s3 spectrum table
```
stage=> copy category_stage_internal
stage-> from 's3://MYBUCKETREDSHIFT/stage/manifest'
stage-> iam_role 'arn:aws:iam::1111111111111111:role/IAMREDSHIFT'
stage-> manifest
stage-> delimiter ',';
INFO:  Load into table 'category_stage_internal' completed, 1000 record(s) loaded successfully.
COPY

stage=> select count(*) from public.category_stage_internal;
 count
-------
  1000
(1 row)
```
