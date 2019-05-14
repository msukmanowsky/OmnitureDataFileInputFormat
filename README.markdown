# Omniture Data

## what is it
A collection tools for Hadoop eco system that makes it easier to parse Omniture Click Stream data. It consists of custom input format and a custom pig loader. Works identically to TextInputFormat except for the fact that it uses a EscapedLineReader which gets around Omniture's pesky escaped tabs and newlines. For more information about format, please refer to Omniture Documentation https://marketing.adobe.com/resources/help/en_US/sc/clickstream/analytics_clickstream.pdf.

It supports only the newer `org.apache.hadoop.mapreduce` API.

## License
https://github.com/msukmanowsky/OmnitureDataFileInputFormat/blob/master/LICENSE

## Maven
The artifacts are available in maven central repository.
```
<dependency>
  <groupId>org.rassee</groupId>
  <artifactId>omnituredata</artifactId>
  <version>1.0.1</version>
</dependency>
```
## Authors
### Satish Iyengar - [Gannett](http://gannett.com)
### Mike Sukmanowsky - (Parsely Inc)

## keys and values

* Keys are `LongWritable` and represent the byte offset of the record in the file.
* Values are `Text` with all instances of \\t, \\n or \\r\n replaced with a space (so that hive/pig can correctly split the columns)

## usage in Spark (Scala)

After adding the dependency to the project we can:

```
    import org.rassee.omniture.hadoop.mapred.OmnitureDataFileInputFormat
    import java.nio.charset.StandardCharsets
    import org.apache.hadoop.io.{LongWritable, Text}
    import org.apache.hadoop.mapred.InputFormat

    val rddLines: RDD[String] =
      sparkSession.sparkContext.hadoopFile(
        path = path,
        inputFormatClass = classOf[OmnitureDataFileInputFormat],
        keyClass = classOf[LongWritable],
        valueClass = classOf[Text]
      )
      .map(_._2.getBytes()).map(new String(_, StandardCharsets.UTF_8))
```

Then proceed as normal to split the TSV via a library or simply `rddLines.map(_.split("\t", -1).toList)`

## usage in hive
To use `OmnitureDataFileInputFormat` in Hive, simply specify it during your create statement.

    CREATE TABLE (...)
    ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t' 
    STORED AS INPUTFORMAT 'org.rassee.omniture.hadoop.mapred.OmnitureDataFileInputFormat' 
    OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.IgnoreKeyTextOutputFormat';

## usage in pig
```
    register 'hdfs://localhost:8020/tmp/omnituredata-1.0.0-SNAPSHOT-jar-with-dependencies.jar';
    set mapred.create.symlink yes;
    set mapreduce.job.cache.files hdfs://localhost:8020/tmp/gpapermobile-schema.json#gpapermobile-schema.json;
    a = load '/tmp/gpapermobile' using org.rassee.omniture.pig.OmnitureDataLoader('gpapermobile-schema.json');
    store a into '/tmp/gpapermobile_l1' using AvroStorage();
```

OmnitureDataLoader supports dynamic schema definition. Dynamic schema definition allows one to specific basic data types to the parsed data. Omniture provides a very wide table (100s of columns). This makes it very important to treat columns with specific data type as much as possible. The schema definition file must be in json format and must have following format. The schema file must be present in HDFS or s3 (s3n must be configured) location to be leveraged for distributed cache.
 
```
{
    "columnHeaders": "<RAW text from column_headers.tsv received from Omniture. This rarely changes>",
    "columnHeaderDelimiter":"\t",
    "longColumns": ["col1", "col2", col3"],
    "decimalColumns": ["col4"],
    "timestampColumns": ["col5"]
}
```
OR
```
{
    "columnHeaders": "<RAW text from column_headers.tsv received from Omniture. This rarely changes>",
    "columnHeaderDelimiter":"\t",
    "longColumns": ["col1", "col2", col3"],
    "decimalColumns": ["col4"],
    "timestampColumns": ["col5"]
}
```
OR
```
{
    "columnHeaders": "<RAW text from column_headers.tsv received from Omniture. This rarely changes>",
    "columnHeaderDelimiter":"\t"
}
```

If columns are not included in data types, then all columns are treated as "chararray". 

# TODO
* Add tests for pig using pig unit
* Add support for more data types
* Add support for custom schema definition in Hive through table properties
* Cleanup
