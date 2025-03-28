## Creating a WordCount JAR file 

**~$** ```gedit WordCount.java```

## WordCount.java
    import java.io.IOException;
    import java.util.StringTokenizer;
    import org.apache.hadoop.conf.Configuration;
    import org.apache.hadoop.fs.Path;
    import org.apache.hadoop.io.IntWritable;
    import org.apache.hadoop.io.LongWritable;
    import org.apache.hadoop.io.Text;
    import org.apache.hadoop.mapreduce.Job;
    import org.apache.hadoop.mapreduce.Mapper;
    import org.apache.hadoop.mapreduce.Reducer;
    import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
    import org.apache.hadoop.mapreduce.lib.input.TextInputFormat;
    import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
    import org.apache.hadoop.mapreduce.lib.output.TextOutputFormat;
    public class WordCount {
        public static class Map extends Mapper<LongWritable, Text, Text, IntWritable> {
            private final static IntWritable one = new IntWritable(1);
            private Text word = new Text();
            @Override
            protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
                String line = value.toString();
                StringTokenizer tokenizer = new StringTokenizer(line);
                while (tokenizer.hasMoreTokens()) {
                    word.set(tokenizer.nextToken());
                    context.write(word, one);
                }
            }
        }
        
        public static class Reduce extends Reducer<Text, IntWritable, Text, IntWritable> {
            private IntWritable value = new IntWritable(0);
            @Override
            protected void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException {
                int sum = 0;
                for (IntWritable value : values)
                    sum += value.get();
                value.set(sum);
                context.write(key, value);
            }
        }
    
        public static void main(String[] args) throws Exception {
            Configuration conf = new Configuration();
            Job job = new Job(conf, "wordcount");
            job.setJarByClass(WordCount.class);
            job.setOutputKeyClass(Text.class);
            job.setOutputValueClass(IntWritable.class);
            job.setMapperClass(Map.class);
            job.setReducerClass(Reduce.class);
            job.setInputFormatClass(TextInputFormat.class);
            job.setOutputFormatClass(TextOutputFormat.class);
            job.setNumReduceTasks(1);
    
            FileInputFormat .setInputPaths(job, new Path(args[0]));
            FileOutputFormat.setOutputPath(job, new Path(args[1]));
    
            boolean success = job.waitForCompletion(true);
            System.out.println(success);
        }
    }

**~$** ```javac -cp hadoopjar/*: WordCount.java```

**~$** ```jar -cvf WordCount.jar WordCount*.class```

## Start  Hadoop

**~$** ```hdfs namenode -format```

**~$** ```start-all.sh```

**~$** ```hdfs dfs -ls -R /```

**~$** ```hdfs dfs -mkdir /wordcount```

**~$** ```hdfs dfs -ls -R /```

**~$** ```gedit words.txt```
 
 

## words.txt

    The majestic mountains stood tall against the backdrop of the clear blue sky, 
    casting long shadows across the lush green valleys below. A gentle breeze whispered through the pine 
    trees, carrying with it the sweet scent of wildflowers. The air was crisp and invigorating, filling 
    my lungs with every breath. As I stood there, surrounded by the beauty of nature,I felt a sense of 
    peace wash over me. It was as if time stood still, and all my worries melted away.In that moment, 
    I was grateful for the simple joys that life had to offer
    
**~$** ```hdfs dfs -put ./words.txt /wordcount```

**~$** ```hdfs dfs -ls -R /```

## Run the JAR file in HDFS

**~$** ```hadoop jar ./WordCount.jar WordCount /wordcount/words.txt /output```

**~$** ```hdfs dfs -cat /output/part-r-00000```
