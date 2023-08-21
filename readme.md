**Basic guide on how to use the SUPER data type in AWS Redshift**

In 2022 Redshift announced a new data type for querying semi-structured JSON data within a Redshift database table.  This datta type is SUPER.  The ability to query JSON formatted data in Redshift reduces the need to flatten JSON objects prior to ingesting them into Redshift.  For more information on the SUPER data types see [here](/https://docs.aws.amazon.com/redshift/latest/dg/r_SUPER_type.html)

In this demo we will complete the following;
*	Create a Redshift Serverless instances, database and table with SUPER data type column
*	Ingest data from S3 into the table
*	Query the hierarchical data using PartiSQL dot notation

To get started with Redshift Serverless and how to provision a , please follow the steps in this [video](/https://www.youtube.com/watch?v=XcRJjXudIf8&t=7s).

Once your Redshift Serverless endpoint is ready, select the Query Data V2 option.

The following code creates a table with a SUPER data type column; 

<code> CREATE TABLE customer_orders_lineitem
(c_custkey bigint
,c_name varchar
,c_address varchar
,c_nationkey smallint
,c_phone varchar
,c_acctbal decimal(12,2)
,c_mktsegment varchar
,c_comment varchar
,c_orders super
);
</code>

The following statement will ingest data from a publiclly available bucket into the table that has just been created.  You need to replace the role ARN with a role ARN that has permissions to S3; 

<code> COPY customer_orders_lineitem FROM 's3://redshift-downloads/semistructured/tpch-nested/data/json/customer_orders_lineitem'
REGION 'us-east-1' IAM_ROLE 'arn:aws:iam::XXXXXXXXXXX:role/service-role/AmazonRedshift-CommandsAccessRole-20221123T180048'
FORMAT JSON 'auto';
</code>

Lets check that data has been loaded into the table – there should be 5 rows.  

<code> select * from customer_orders_lineitem </code>

This statement returns the SUPER data type column

<code> select c_orders from customer_orders_lineitem </code>

Redshift uses PartiQL to enable navigation into arrays and structures using the [] bracket and dot notation respectively.  

Lets use dot notation to filter records

<code>SELECT count(*) FROM customer_orders_lineitem WHERE c_orders[0]. o_orderkey IS NOT NULL </code>

The following example uses the bracket and dot navigation in both GROUP BY and ORDER BY clauses.

<code> SELECT c_orders[0].o_orderdate,
       c_orders[0].o_orderstatus,
       count(*)
FROM customer_orders_lineitem
WHERE c_orders[0].o_orderkey IS NOT NULL
GROUP BY c_orders[0].o_orderstatus,
         c_orders[0].o_orderdate
ORDER BY c_orders[0].o_orderdate;

</code>

There is no limit to the depth of dot notation.

For more exammples of how to query semi-structured data using the SUPER data type please see [here](/https://docs.aws.amazon.com/redshift/latest/dg/r_SUPER_sample_dataset.html) 


