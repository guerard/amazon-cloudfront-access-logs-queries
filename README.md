# Instructions

You should have two buckets in the same region you want your Athena table:
- somewhere to put the artifacts during sam package i.e. ARTIFACTS-BUCKET
- the bucket for the access logs i.e. ACCESS-LOGS-BUCKET

```shell script
sam package \
--template-file template.yaml \
--output-template-file packaged.yaml \
--s3-bucket <ARTIFACTS-BUCKET>
```

```shell script
aws cloudformation deploy \
--template-file packaged.yaml \
--stack-name athena-amplify-stack \
--capabilities CAPABILITY_IAM \
--parameter-overrides BucketName=<ACCESS-LOGS-BUCKET>
```

Once the stack has spun up, open up the Athena query editor and specify the results folder as
`<ACCESS-LOGS-BUCKET>/query-results`.

Next you'll run the command:
```
MSCK REPAIR TABLE partitioned_gz;
```
This will update the table's metadata to include your partitions in s3.
You only have to run this the first time, as the created lambda (CreatePartFn)
will incrementally create hourly partitions for you.

Here's a sample SQL query to get you going:
```
SELECT * FROM partitioned_gz LIMIT 10;
```

Since the table is partitioned by hour, make sure to filter by date so they run
efficiently:
```
SELECT SUM(bytes) AS total_bytes
FROM partitioned_gz
WHERE year = '2020'
AND month = '03'
AND day = '19'
AND hour BETWEEN '12' AND '23';
```
