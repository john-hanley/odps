# Open-source MapReduce {#concept_yc2_cyf_vdb .concept}

MaxCompute offers a set of native MapReduce programming models and interfaces. The inputs and outputs for these interfaces are MaxCompute tables, and data is organized for processing in record format.

However, MaxCompute APIs differ significantly from APIs for the Hadoop framework. Previously, if you wanted to migrate your Hadoop MapReduce jobs to MaxCompute, you needed to first rewrite the MapReduce code, compile, and debug the code using MaxCompute APIs, compress the final code into a JAR package, and finally upload the package to the MaxCompute platform.  This process is tedious and requires a lot of development and testing efforts. If you do not need to modify or modify the original Hadoop MapReduce code partially, running it in MaxCompute console is the best solution.

Now, the MaxCompute platform provides a plug-in that allows you to adapt Hadoop MapReduce code to MaxCompute MapReduce specifications. MaxCompute offers a degree of flexibility regarding binary-level compatibility for Hadoop MapReduce jobs. This means that, without modifying the code, you can specify configurations to directly run original Hadoop MapReduce Jar packages on MaxCompute.  You can download the development plug-in to get [started](http://repo.aliyun.com/download/hadoop2openmr-1.0.jar). This plug-in is currently in the testing stage and does not support custom comparators or key types. 

In the following example, a WordCount program is used to introduce the basic usage of the plug-in.

## Download the HadoopMR Plug-in {#section_cb2_l1g_vdb .section}

Click [here](http://repo.aliyun.com/download/hadoop2openmr-1.0.jar) to download the plug-in named hadoop2openmr-1.0.jar.

**Note:** This Jar package contains the dependencies with Hadoop 2.7.2. Do not include Hadoop dependencies in the Jar packages of your jobs to avoid version conflicts.

## Prepare Jar package {#section_byf_p1g_vdb .section}

Compile and export the WordCount JAR package named wordcount\_test.jar. The WordCount program source code is as follows:

```
package com.aliyun.odps.mapred.example.hadoop;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import java.io.IOException;
import java.util.StringTokenizer;
public class WordCount {
    public static class TokenizerMapper
        extends Mapper<Object, Text, Text, IntWritable>{
        private final static IntWritable one = new IntWritable(1);
        private Text word = new Text();
        public void map(Object key, Text value, Context context
        ) throws IOException, InterruptedException {
            StringTokenizer itr = new StringTokenizer(value.toString());
            while (itr.hasMoreTokens()) {
                word.set(itr.nextToken());
                context.write(word, one);
            }
        }
    }
    public static class IntSumReducer
        extends Reducer<Text,IntWritable,Text,IntWritable> {
        private IntWritable result = new IntWritable();
        public void reduce(Text key, Iterable<IntWritable> values,
            Context context
        ) throws IOException, InterruptedException {
            int sum = 0;
            for (IntWritable val : values) {
                sum += val.get();
            }
            result.set(sum);
            context.write(key, result);
        }
    }
    public static void main(String[] args) throws Exception {
        Configuration conf = new Configuration();
        Job job = Job.getInstance(conf, "word count");
        job.setJarByClass(WordCount.class);
        job.setMapperClass(TokenizerMapper.class);
        job.setCombinerClass(IntSumReducer.class);
        job.setReducerClass(IntSumReducer.class);
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);
        FileInputFormat.addInputPath(job, new Path(args[0]));
        FileOutputFormat.setOutputPath(job, new Path(args[1]));
        System.exit(job.waitForCompletion(true) ? 0 : 1);
    }
}

```

## Prepare Test Data {#section_y1l_s1g_vdb .section}

1.  Create input and output tables

    ```
    create table if not exists wc_in(line string);
    create table if not exists wc_out(key string, cnt bigint);
    ```

2.  Run Tunnel to import data to the input table.

    The data in the data.txt text file to be imported is as follows:

    ```
    hello maxcompute
    hello mapreduce
    ```

    you can use the `Tunnel command` at the MaxCompute console to import data from data.txt to wc\_in.

    ```
    tunnel upload data.txt wc_in;
    ```


## Configure the Mapping Between the Table and the HDFS File Path {#section_b22_1bg_vdb .section}

The configuration file is wordcount-table-res.conf.

```
{
  "file:/foo": {
    "resolver": {
      "resolver": "com.aliyun.odps.mapred.hadoop2openmr.resolver.TextFileResolver",
      "properties": {
          "text.resolver.columns.combine.enable": "true",
          "text.resolver.seperator": "\t"
      }
    },
    "tableInfos": [
      {
        "tblName": "wc_in",
        "partSpec": {},
        "label": "__default__"
      }
    ],
    "matchMode": "exact"
  },
  "file:/bar": {
    "resolver": {
      "resolver": "com.aliyun.odps.mapred.hadoop2openmr.resolver.BinaryFileResolver",
      "properties": {
          "binary.resolver.input.key.class" : "org.apache.hadoop.io.Text",
          "binary.resolver.input.value.class" : "org.apache.hadoop.io.LongWritable"
      }
    },
    "tableInfos": [
      {
        "tblName": "wc_out",
        "partSpec": {},
        "label": "__default__"
      }
    ],
    "matchMode": "fuzzy"
  }
}
```

Configuration item descriptions:

The configuration is a JSON file that describes the mapping relationships between HDFS files and MaxCompute tables. Generally, you must configure both the input and output. One HDFS path corresponds to one Resolver, tableInfos, and matchMode.

-   resolver: specifies the method of processing file data. Currently, you can choose from two built-in Resolvers: com.aliyun.odps.mapred.hadoop2openmr.resolver.TextFileResolver and com.aliyun.odps.mapred.hadoop2openmr.resolver.BinaryFileResolver.  In addition to specifying the Resolver name, you must also configure some properties about data parsing for the Resolver.
    -   TextFileResolver: regards an input or output as plain text if the data is of plain text type. When configuring an input Resolver, you must configure such properties as text.resolver.columns.combine.enable and text.resolver.seperator. When text.resolver.columns.combine.enable is set to true, all the columns in the input table are combined into a single string based on the delimiter specified by text.resolver.seperator. Otherwise, the first two columns in the input table are used as the key and value.
    -   BinaryFileResolver: converts binary data into a type that is supported by MaxCompute, for example, Bigint, Boolean, and Double. When configuring an output Resolver, you must configure the properties binary.resolver.input.key.class and binary.resolver.input.value.class, which define the key and value types of the intermediate result, respectively.
-   tableInfos: specifies the MaxCompute table that corresponds to HDFS. Currently, only the tblName parameter \(table name\) is configurable. The partSpec and label parameters must be the same as the values set for the parameters in this example.
-   matchMode: specifies the path matching mode. The exact mode indicates exact matching, and the fuzzy mode indicates fuzzy matching. You can use a regular expression in fuzzy mode to match the HDFS input path.

## Job Submission {#section_okw_gbg_vdb .section}

Use the MaxCompute command line tool odpscmd to submit jobs. For the installation and configuration of MaxCompute command line tool, see the [Console](../../../../intl.en-US/Tools and Downloads/Client.md#). In odpscmd, run the following command:

```
jar -DODPS_HADOOPMR_TABLE_RES_CONF=./wordcount-table-res.conf -classpath hadoop2openmr-1.0.jar,wordcount_test.jar com.aliyun.odps.mapred.example.hadoop.WordCount /foo/bar;
```

**Note:** 

-   wordcount-table-res.conf is a map with /foo/bar configured.
-   wordcount\_test.jar is your Jar package of Hadoop MapReduce.
-   com.aliyun.odps.mapred.example.hadoop.WordCount is the class name of job to be run.
-   /foo/bar refers to the path on HDFS, which is mapped to wc\_in and wc\_out in the JSON configuration file.
-   With the mapping relation configured, you must manually import the Hadoop HDFS input file to wc\_in for MR calculations by using data integration functions of DataX or DataWorks, and manually export the result wc\_out to your HDFS output directory\(/bar\).
-   In the preceding output, assume that hadoop2openmr-1.0.jar, wordcount\_test.jar, and wordcount-table-res.conf are stored in the current directory of odpscmd. If an error occurs, you must make the relevant changes when specifying the configuration and -classpath.

The running process is shown in the following figure.

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12015/1957_en-US.jpg)

After running the job, check the results table wc\_out to verify that the job has been completed. 

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12015/1959_en-US.jpg)

