  * [Advanced Grok Concepts](#advanced-grok-concepts)
    * [Capturing Logging Levels](#capturing-logging-levels)
    * [Fielding Information](#fielding-information)
    * [Cleaning Up Fields](#cleaning-up-fields)
    * [Applying Tags](#applying-tags)
    * [Additional Exercises](#additional-exercises)
    
Created by [gh-md-toc](https://github.com/ekalinin/github-markdown-toc)

# Advanced Grok Concepts

Let's look at some additional operations we can do with the grok filter and how we can produce valuable
metadata tags from our application output.

## Capturing Logging Levels
Pulling out the log level of each message will help us later with filtering and searching through our application's
logging. To accomplish this, we will augment our existing grok filter with additional behaviors. Take a look at a
line from our sample log file:
```
2016-09-14 19:23:21,443 BrainTrust.host-8001  INFO com.braintrust.http.controller.Controller
-   - NavigationFile=/WEB-INF/mvc/Navigation.xml
```

We are already pulling out the date information, so we just need to build up our grok example to account for the rest
of the line. Try changing the grok filter to use the following pattern:
```
  grok {
    match => { "message" => "%{TIMESTAMP_ISO8601:tempTime}\s%{NOTSPACE:machine}\s+%{NOTSPACE:logLevel}" }
  }
```

Here we are captureing the time stamp, then the host, and the log level of the message. As mentioned previously, the grok
filter supports a regular expressions for pulling out information. It also comes with around 150 predefined patterns that
you use while parsing your log messages. Here we are using the ```NOTSPACE``` pattern, which captures any character that
isn't a space. The full list of predefined grok patterns can be found [here](https://github.com/logstash-plugins/logstash-patterns-core/blob/master/patterns/grok-patterns).

Let's run logstash and see what our output looks like now:
```
{
    "@timestamp" => "2016-09-14T23:23:21.515Z",
       "message" => "2016-09-14 19:23:21,515 BrainTrust.host-8001  WARN com.braintrust.WorkerFactory -   - Some Warning Here with coolFieldToGrab: cool value, more logging",
      "@version" => "1",
          "path" => "/tmp/sample-log.txt",
          "host" => "host.machine",
       "machine" => "BrainTrust.host-8001",
      "logLevel" => "WARN"
}
{
    "@timestamp" => "2016-09-14T23:23:21.516Z",
       "message" => "2016-09-14 19:23:21,516 BrainTrust.host-8001  INFO com.braintrust.Navigation -   - Navigation update completed, switching to live rule set",
      "@version" => "1",
          "path" => "/tmp/sample-log.txt",
          "host" => "host.machine",
       "machine" => "BrainTrust.host-8001",
      "logLevel" => "INFO"
}

```
Now we have the log level for each message broken out into its own field, which will be great when we searching and
filtering out messages with ElasticSearch.

## Fielding Information
What if you wanted to pull out certain pieces of information from the log messages themselves and have them fielded?
Take a look at the following log line and we will show how that is possible:
```
2016-09-14 19:23:21,515 BrainTrust.host-8001  WARN com.braintrust.WorkerFactory -   -
Some Warning Here with coolFieldToGrab: cool value, more logging

```
We will grab the value ```cool value``` and stick it in a field called ```coolFieldToGrab```. Take a look at the
our filter block to see how this is done
```
filter {
  grok {
    match => { "message" => "%{TIMESTAMP_ISO8601:tempTime}\s%{NOTSPACE:machine}\s+%{NOTSPACE:logLevel}%{GREEDYDATA:body}" }
  }
  date {
    match => [ "tempTime", "YYYY-MM-dd HH:mm:ss,SSS"]
    remove_field => [ "tempTime" ]
  }

  if [body] =~ /.*coolFieldToGrab.*/ {
    grok {
      match => { "body" => ".*coolFieldToGrab:\s%{DATA:coolFieldToGrab},.*"}
    }
  }
}
```
First, we are creating a field called ```body``` to hold the rest of the message once we do our initial timestamp,
host, and log level parsing. We have added an if block in the parser to look for our field ```coolFieldToGrab``` in
our ```body``` field which is done using a regex. Once it is found, we parse the message and save the value. Running
logstash with this configuration gives the following output:
```
{
         "@timestamp" => "2016-09-14T23:23:21.515Z",
            "message" => "2016-09-14 19:23:21,515 BrainTrust.host-8001  WARN com.braintrust.WorkerFactory -   - Some Warning Here with coolFieldToGrab: cool value, more logging",
           "@version" => "1",
               "path" => "/tmp/sample-log.txt",
               "host" => "host.machine",
            "machine" => "BrainTrust.host-8001",


           "logLevel" => "WARN",
               "body" => " com.braintrust.WorkerFactory -   - Some Warning Here with coolFieldToGrab: cool value, more logging",
    "coolFieldToGrab" => " cool value"
}
```
Now, whenever a message contains ```coolFieldToGrab```, the value will be captured in a field.

## Cleaning Up Fields
We probably don't need to keep both the ```message``` and ```body``` field. Since those three fields are already being
captured, we can drop the message field and replace it with the body field. We will accomplish this using the
[mutate](https://www.elastic.co/guide/en/logstash/current/plugins-filters-mutate.html#plugins-filters-mutate-remove_field) filter
```
  mutate {
    replace => [ "message", "%{body}" ]
    remove_field => [ "body" ]
  }
```
By adding this to our filter{} block, we are replacing the value of the ```message``` field with the
value of our ```body``` field, then dropping the ```body``` field from our JSON document.
Here is what the output will look with this addition:
```
{


         "@timestamp" => "2016-09-14T23:23:21.515Z",
            "message" => " com.braintrust.WorkerFactory -   - Some Warning Here with coolFieldToGrab: cool value, more logging",
           "@version" => "1",
               "path" => "/tmp/sample-log.txt",
               "host" => "host.machine",
            "machine" => "BrainTrust.host-8001",
           "logLevel" => "WARN",
    "coolFieldToGrab" => "cool value"
}
{
    "@timestamp" => "2016-09-14T23:23:21.516Z",
       "message" => " com.braintrust.Navigation -   - Navigation update completed, switching to live rule set",
      "@version" => "1",
          "path" => "/tmp/sample-log.txt",
          "host" => "host.machine",
       "machine" => "BrainTrust.host-8001",
      "logLevel" => "INFO"
}
```
This is a much cleaner output and will minimize the space needed to store our records.

## Applying Tags
You can also add custom tags to your messages. This can help you later when making visualizations or when looking for
specific messages later. We will use the following log messages as our example for this exercise:
```
2016-09-14 19:21:40,574 BrainTrust.host-8001  INFO com.braintrust.ClassOne - InterestingParameter  -
   filename: input file
   creationTimestamp: Wed Sep 14 19:21:32 GMT 2016
   filetype: ZIP
```
We will look for the creationTimestamp: text in all log messages coming from ```com.braintrust.ClassOne``` and apply
a ```CREATION``` tag. Add the following snippet to your filter{} block:
```
  if [body] =~ /.*com.braintrust.ClassOne.*/ and [body] =~ /.*creationTimestamp:.*/ {
    mutate {
      add_tag => [ "CREATION" ]
    }
  }
```
Again, we are using two regex matchers to scan for the desired class and the desired text. Using the mutate filter
allows us to add our specific tag to our document. Running Logstash with these additions will produce the following output:
```
{
    "@timestamp" => "2016-09-14T23:21:40.574Z",
       "message" => " com.braintrust.ClassOne - InterestingParameter  -\n   filename: input file\n   creationTimestamp: Wed Sep 14 19:21:32 GMT 2016\n   filetype: ZIP",
      "@version" => "1",
          "tags" => [
        [0] "multiline",
        [1] "CREATION"
    ],
          "path" => "/tmp/sample-log.txt",
          "host" => "host.machine",
       "machine" => "BrainTrust.host-8001",
      "logLevel" => "INFO"
}
```
## Additional Exercises
Hopefully, you will feel somewhat comfortable using Logstash and have a good understanding of how to use the different
filters to parse your log messages. From here, I would recommend trying the following exercises to strengthen your
knowledge.
1. Pull the class from each log line into a field called ```class```
2. Change the ```CREATION``` tag if block to use the ```class``` field instead of scanning the body
3. Pull the ```Interesting Parameter``` value out of the log message and field it

You can find the full Logstash config file used in this tutorial [here](src/main/resources/com/braintrust/logstash/03-Advanced_Grok_Concepts/logstash-advanced.conf).

