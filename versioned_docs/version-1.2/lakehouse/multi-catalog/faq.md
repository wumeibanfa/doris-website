---
{
    "title": "FAQ",
    "language": "en"
}
---

# FAQ

1. What to do with errors such as  `failed to get schema` and  `Storage schema reading not supported`  when accessing Icerberg tables via Hive Metastore?

   To fix this, please place the Jar file package of `iceberg` runtime in the `lib/` directory of Hive.

   And configure as follows in  `hive-site.xml` :

   ```
   metastore.storage.schema.reader.impl=org.apache.hadoop.hive.metastore.SerDeStorageSchemaReader
   ```

   After configuring, please restart Hive Metastore.

2. What to do with the `GSS initiate failed` error when connecting to Hive Metastore with Kerberos authentication?

   Usually it is caused by incorrect Kerberos authentication information, you can troubleshoot by the following steps:

   1. In versions before  1.2.1, the libhdfs3 library that Doris depends on does not enable gsasl. Please update to a version later than 1.2.2.
   2. Confirm that the correct keytab and principal are set for each component, and confirm that the keytab file exists on all FE and BE nodes.

       1. `hadoop.kerberos.keytab`/`hadoop.kerberos.principal`: for Hadoop HDFS
       2. `hive.metastore.kerberos.principal`: for hive metastore.

   3. Try to replace the IP in the principal with a domain name (do not use the default `_HOST` placeholder)
   4. Confirm that the `/etc/krb5.conf` file exists on all FE and BE nodes.

3. What to do with the`java.lang.VerifyError: xxx` error when accessing HDFS 3.x?

   Doris 1.2.1 and the older versions rely on Hadoop 2.8. Please update Hadoop to 2.10.2 or update Doris to 1.2.2 or newer.

4. An error is reported when using KMS to access HDFS: `java.security.InvalidKeyException: Illegal key size`

    Upgrade the JDK version to a version >= Java 8 u162. Or download and install the JCE Unlimited Strength Jurisdiction Policy Files corresponding to the JDK.

5. When querying a table in ORC format, FE reports an error `Could not obtain block` or `Caused by: java.lang.NoSuchFieldError: types`

    For ORC files, by default, FE will access HDFS to obtain file information and split files. In some cases, FE may not be able to access HDFS. It can be solved by adding the following parameters:

    `"hive.exec.orc.split.strategy" = "BI"`

    Other options: HYBRID (default), ETL.

6. An error is reported when connecting to SQLServer through JDBC Catalog: `unable to find valid certification path to requested target`

    Please add `trustServerCertificate=true` option in `jdbc_url`.

7. When connecting to the MySQL database through the JDBC Catalog, the Chinese characters are garbled, or the Chinese character condition query is incorrect

    Please add `useUnicode=true&characterEncoding=utf-8` in `jdbc_url`

    > Note: After version 1.2.3, these parameters will be automatically added when using JDBC Catalog to connect to the MySQL database.

8. An error is reported when connecting to the MySQL database through the JDBC Catalog: `Establishing SSL connection without server's identity verification is not recommended`

    Please add `useSSL=true` in `jdbc_url`

9. An error is reported when connecting Hive Catalog: `Caused by: java.lang.NullPointerException`

    If there is stack trace in fe.log:

    ```
    Caused by: java.lang.NullPointerException
        at org.apache.hadoop.hive.ql.security.authorization.plugin.AuthorizationMetaStoreFilterHook.getFilteredObjects(AuthorizationMetaStoreFilterHook.java:78) ~[hive-exec-3.1.3-core.jar:3.1.3]
        at org.apache.hadoop.hive.ql.security.authorization.plugin.AuthorizationMetaStoreFilterHook.filterDatabases(AuthorizationMetaStoreFilterHook.java:55) ~[hive-exec-3.1.3-core.jar:3.1.3]
        at org.apache.hadoop.hive.metastore.HiveMetaStoreClient.getAllDatabases(HiveMetaStoreClient.java:1548) ~[doris-fe.jar:3.1.3]
        at org.apache.hadoop.hive.metastore.HiveMetaStoreClient.getAllDatabases(HiveMetaStoreClient.java:1542) ~[doris-fe.jar:3.1.3]
        at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method) ~[?:1.8.0_181]
    ```

    Try adding `"metastore.filter.hook" = "org.apache.hadoop.hive.metastore.DefaultMetaStoreFilterHookImpl"` in `create catalog` statement.

10. An error is reported when connecting to the Hive database through the Hive Catalog: `RemoteException: SIMPLE authentication is not enabled. Available: [TOKEN, KERBEROS]`

    If both `show databases` and `show tables` are OK, and the above error occurs when querying, we need to perform the following two operations:
    - Core-site.xml and hdfs-site.xml need to be placed in the fe/conf and be/conf directories
    - The BE node executes the kinit of Kerberos, restarts the BE, and then executes the query.

11. If the `show tables` is normal after creating the Hive Catalog, but the query report `java.net.UnknownHostException: xxxxx`

    Add a property in CATALOG:
    ```
    'fs.defaultFS' = 'hdfs://<your_nameservice_or_actually_HDFS_IP_and_port>'
    ```
12. The values of the partition fields in the hudi table can be found on hive, but they cannot be found on doris.

    Doris and hive currently query hudi differently. Doris needs to add partition fields to the avsc file of the hudi table structure. If not added, it will cause Doris to query partition_ Val is empty (even if home. datasource. live_sync. partition_fields=partition_val is set)

    ```
    {
        "type": "record",
        "name": "record",
        "fields": [{
            "name": "partition_val",
            "type": [
                "null",
                "string"
                ],
            "doc": "Preset partition field, empty string when not partitioned",
            "default": null
            },
            {
            "name": "name",
            "type": "string",
            "doc": "名称"
            },
            {
            "name": "create_time",
            "type": "string",
            "doc": "创建时间"
            }
        ]
    }
    ```

13. The table in orc format of Hive 1.x may encounter system column names such as `_col0`, `_col1`, `_col2`... in the underlying orc file schema, which need to be specified in the catalog configuration. Add `hive.version` to 1.x.x so that it will use the column names in the hive table for mapping.

    ```sql
    CREATE CATALOG hive PROPERTIES (
        'hive.version' = '1.x.x'
    );
    ```

14. When using JDBC Catalog to synchronize MySQL data to Doris, the date data synchronization error occurs. It is necessary to check whether the MySQL version corresponds to the MySQL driver package. For example, the driver com.mysql.cj.jdbc.Driver is required for MySQL8 and above.

15. If an error is reported while configuring Kerberos in the catalog: `SIMPLE authentication is not enabled. Available:[TOKEN, KERBEROS]`.

    Need to put `core-site.xml` to the `"${DORIS_HOME}/be/conf"` directory.

    If an error is reported while accessing HDFS: `No common protection layer between client and server`, check the `hadoop.rpc.protection` on the client and server to make them consistent.

    ```
        <?xml version="1.0" encoding="UTF-8"?>
        <?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
        
        <configuration>
        
            <property>
                <name>hadoop.security.authentication</name>
                <value>kerberos</value>
            </property>
        
        </configuration>
    ```

16. The solutions when configuring Kerberos in the catalog and encounter an error: `Unable to obtain password from user`.
    - The principal used must exist in the klist, use `klist -kt your.keytab` to check.
    - Ensure the catalog configuration correct, such as missing the `yarn.resourcemanager.principal`.
    - If the preceding checks are correct, the JDK version installed by yum or other package-management utility in the current system maybe have an unsupported encryption algorithm. It is recommended to install JDK by yourself and set `JAVA_HOME` environment variable.

17. If an error is reported while querying the catalog with Kerberos: `GSSException: No valid credentials provided (Mechanism level: Failed to find any Kerberos Ticket)`.
    - Restarting FE and BE can solve the problem in most cases. 
    - Before the restart all the nodes, can put `-Djavax.security.auth.useSubjectCredsOnly=false` to the `JAVA_OPTS` in `"${DORIS_HOME}/be/conf/be.conf"`, which can obtain credentials through the underlying mechanism, rather than through the application.
    - Get more solutions to common JAAS errors from the [JAAS Troubleshooting](https://docs.oracle.com/javase/8/docs/technotes/guides/security/jgss/tutorials/Troubleshooting.html).

18. If an error related to the Hive Metastore is reported while querying the catalog: `Invalid method name`.

    Configure the `hive.version`.

    ```sql
    CREATE CATALOG hive PROPERTIES (
        'hive.version' = '2.x.x'
    );
    ```

19. `BlockMissingExcetpion: Could not obtain block: BP-XXXXXXXXX No live nodes contain current block`

    Possible solutions include:
    - Use `hdfs fsck file -files -blocks -locations` to check if the file is healthy.
    - Use telnet to check connectivity with the DataNode.
    - Check the DataNode logs.

    If encountering the following error:
    `org.apache.hadoop.hdfs.server.datanode.DataNode: Failed to read expected SASL data transfer protection handshake from client at /XXX.XXX.XXX.XXX:XXXXX. Perhaps the client is running an older version of Hadoop which does not support SASL data transfer protection`.
    It indicates that HDFS is configured for encrypted transmission while the client is not, causing the error.

    You can use any of the following solutions:
    - Copying hdfs-site.xml and core-site.xml to the be/conf and fe/conf directories. (Recommended)
    - In hdfs-site.xml, find the corresponding configuration `dfs.data.transfer.protection`, and set this parameter in the catalog.
