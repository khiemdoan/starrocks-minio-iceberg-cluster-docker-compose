# StarRocks Minio cluster use Iceberg format inside Docker Compose

A simple StarRocks and MinIO cluster uses Iceberg as the table format inside Docker Compose.

This example is based on the [StarRocks documentation](https://docs.starrocks.io/docs/quick_start/iceberg/), and [StarRocks repository](https://github.com/StarRocks/demo/tree/master/documentation-samples)

## Disclaimer

This repository is created by [Khiem Doan](https://github.com/khiemdoan) for educational purposes only. It is not an official StarRocks or MinIO project.

This is a simple setup for development and testing purposes. It is not intended for production use.

I do not take any responsibility for any issues that may arise from using this setup in production.

## Running

1. Run `mkdirs.sh` to create necessary directories.
1. Create a `.env` file in the follow `.env.example` file.
1. Start compose

    ```bash
    docker compose up -d
    ```

1. After the containers are up, you should change password for the `root` user in StarRocks.

    ```sh
    docker exec -it starrocks-leader bash
    mysql -P9030 -h 127.0.0.1 -u root --prompt="StarRocks > " -p
    # without password
    ```

1. Then run the following SQL command to change the password, for example:

    ```sql
    SET PASSWORD = PASSWORD('$w0rdf1sh');
    ```

1. Add compute nodes

    ```sql
    ALTER SYSTEM ADD COMPUTE NODE "starrocks-cn-1:9050";
    ALTER SYSTEM ADD COMPUTE NODE "starrocks-cn-2:9050";
    ALTER SYSTEM ADD COMPUTE NODE "starrocks-cn-3:9050";
    ALTER SYSTEM ADD COMPUTE NODE "starrocks-cn-4:9050";
    ALTER SYSTEM ADD COMPUTE NODE "starrocks-cn-5:9050";
    ALTER SYSTEM ADD COMPUTE NODE "starrocks-cn-6:9050";
    ```

## Accessing services

- MySQL: `localhost:9030`
- StarRocks: [http://localhost:8030](http://localhost:8030)
- MinIO: [http://localhost:9000](http://localhost:9000)

## Try a sample Iceberg table

1. Create a catalog

    ```sql
    CREATE EXTERNAL CATALOG 'demo'
    COMMENT "External catalog to Apache Iceberg on MinIO"
    PROPERTIES
    (
        "type"="iceberg",
        "iceberg.catalog.type"="rest",
        "iceberg.catalog.uri"="http://iceberg-rest:8181",
        "iceberg.catalog.warehouse"="warehouse",
        "aws.s3.access_key"="q49jLoEI9affE1M5CqsM",
        "aws.s3.secret_key"="oFYrxDaD2dXp758MLmx1tBBKmmAejWFWCOSXHjEq",
        "aws.s3.endpoint"="http://minio:9000",
        "aws.s3.enable_path_style_access"="true",
        "client.factory"="com.starrocks.connector.iceberg.IcebergAwsClientFactory"
    );

    SET CATALOG demo;
    ```

1. Creating and using a database

    ```sql
    CREATE DATABASE nyc;

    USE nyc;
    ```

1. Creating a table

    ```sql
    CREATE TABLE demo.nyc.taxis
    (
        trip_id            bigint,
        trip_distance      float,
        fare_amount double,
        store_and_fwd_flag string,
        vendor_id          bigint
    ) PARTITION BY (vendor_id);
    ```

1. Writing Data to a Table

    ```sql
    INSERT INTO demo.nyc.taxis
    VALUES (1000371, 1.8, 15.32, 'N', 1),
           (1000372, 2.5, 22.15, 'N', 2),
           (1000373, 0.9, 9.01, 'N', 2),
           (1000374, 8.4, 42.13, 'Y', 1);
    ```

1. Reading Data from a Table

    ```sql
    SELECT *
    FROM demo.nyc.taxis;
    ```
