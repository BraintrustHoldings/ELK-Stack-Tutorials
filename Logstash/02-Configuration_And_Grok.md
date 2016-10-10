# Additional Configuration and Grok Introduction

  * [Additional Configuration and Grok Introduction](#additional-configuration-and-grok-introduction)
    * [Additional Configuration](#additional-configuration)
      * [File Input Plugin](#file-input-plugin)
        * [Configuring the File Input Plugin](#configuring-the-file-input-plugin)
        * [Sample Input File](#sample-input-file)
        * [Run Logstash](#run-logstash)
        * [File Codec Property](#file-codec-property)
    * [Filter Plugins](#filter-plugins)
      * [Grok Filter Plugin](#grok-filter-plugin)
      * [Date Plugin](#date-plugin)
    * [Next Steps](#next-steps)
    * [Gothca's / Troubleshooting](#gothcas--troubleshooting)
      * [Continuous Output](#continuous-output)
      * [No Output When Starting](#no-output-when-starting)
      * [_dateparsefailure tag](#_dateparsefailure-tag)

Created by [gh-md-toc](https://github.com/ekalinin/github-markdown-toc)

## Additional Configuration
Now that we have installed, configured, started and ran Logstash, let's take a look at some additional configuration options.

### File Input Plugin
The [File](https://www.elastic.co/guide/en/logstash/current/plugins-inputs-file.html) plugin is a very flexible input
plugin that will handle following the contents of any file on disk. There are few mechanisms that Logstash uses to
achieve this but we will not look at this in depth. The details on those mechanisms are found on the file input plugin
page on Elastic's website.

#### Configuring the File Input Plugin
For this next step, we will be removing the generator plugin and using the file input plugin. We will need a couple of
files for this but lets start by looking at our initial configuration:
```
input {
  file {
    path => ["/tmp/sample-log.txt"]
    discover_interval => 10
    sincedb_path => "/tmp/logstash_sincedb_file"
    start_position => "beginning"
  }
}
```
There are many more configuration options available but these are a good core set to begin understanding. Here is
a brief explanation of what each field does:

| Field | Description |
|-------------------|-------------------------------------------------------------------------------------------------------------|
| path | The file path or parse, can handle wildcards in the entries |
| discover_interval | How often Logstash should check for new files |
| sincedb_path | The location Logstash should write its sincedb file (keeps track of position of last read in an input file) |
| start_position | Where to begin reading a new file (beginning and end are valid values) |

#### Sample Input File
For this exercise, copy the file [sample-log.txt](src/main/resources/com/braintrust/logstash/02-Configuration_And_Grok/sample-log.txt)
to /tmp. We will also set the ```sincedb_path``` to /tmp/logstash_sincedb_file so we can examine the contents.
Sample Log File:
```
2016-09-14 19:23:21,442 BrainTrust.host-8001  DEBUG com.braintrust.http.controller.Controller -   - Controller: init
2016-09-14 19:23:21,443 BrainTrust.host-8001  INFO com.braintrust.http.controller.Controller -   - NavigationFile=/WEB-INF/mvc/Navigation.xml
2016-09-14 19:23:21,514 BrainTrust.host-8001  ERROR com.braintrust.WorkerFactory -   - Could not find the target file
    java.io.IOException: Could not find the target file
    at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
    at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:57)
    at com.intellij.rt.execution.application.AppMain.main(AppMain.java:144)
2016-09-14 19:23:21,515 BrainTrust.host-8001  WARN com.braintrust.WorkerFactory -   - Some Warning Here with coolFieldToGrab: cool value, more logging
2016-09-14 19:23:21,516 BrainTrust.host-8001  INFO com.braintrust.Navigation -   - Navigation update completed, switching to live rule set
```
As you can see, this looks like a typical Java log snippet. There are messages at different levels, an exception,
a multi-line message, and other typical features. This will be our primary working input for the remainder of the tutorial.

#### Run Logstash
Now that we have staged our sample log file and configured the input plugin, start Logstash. You should see
a lot of parsed messages fly by the screen (since we are still using the stdout output plugin) as shown below:
```
{
       "message" => "2016-09-14 19:23:21,442 BrainTrust.host-8001  DEBUG com.braintrust.http.controller.Controller -   - Controller: init",
      "@version" => "1",
    "@timestamp" => "2016-09-14T21:29:32.516Z",
          "path" => "/tmp/sample-log.txt",
          "host" => "host.machine"
}
{
       "message" => "2016-09-14 19:23:21,443 BrainTrust.host-8001  INFO com.braintrust.http.controller.Controller -   - NavigationFile=/WEB-INF/mvc/Navigation.xml",
      "@version" => "1",
    "@timestamp" => "2016-09-14T21:29:32.567Z",
          "path" => "/tmp/sample-log.txt",
          "host" => "host.machine"
}
{
       "message" => "2016-09-14 19:23:21,514 BrainTrust.host-8001  ERROR com.braintrust.WorkerFactory -   - Could not find the target file",
      "@version" => "1",
    "@timestamp" => "2016-09-14T21:29:32.568Z",
          "path" => "/tmp/sample-log.txt",
          "host" => "host.machine"
}
{
       "message" => "    java.io.IOException: Could not find the target file",
      "@version" => "1",
    "@timestamp" => "2016-09-14T21:29:32.568Z",
          "path" => "/tmp/sample-log.txt",
          "host" => "host.machine"
}
{
       "message" => "    at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)",
      "@version" => "1",
    "@timestamp" => "2016-09-14T21:29:32.569Z",
          "path" => "/tmp/sample-log.txt",
          "host" => "host.machine"
}
....
```
#### File Codec Property
As you can see, there are several things not quite right with the output. The exception should probably be contained in one message.
The first thing we will need to do is set the codec property on the file input plugin. Lets change our input plugin to the following:
```
input {
  file {
    path => ["/tmp/sample-log.txt"]
    discover_interval => 10
    sincedb_path => "/tmp/logstash_sincedb_file"
    start_position => "beginning"
    codec => multiline {
      pattern => "^%{TIMESTAMP_ISO8601} "
      negate => true
      what => "previous"
    }
  }
}
```
What this will do is cause all lines that do not start with an ISO8601 timestamp to be rolled into the previous
line's message. More information on handling multiple line messages can be found on the [multiline messages](https://www.elastic.co/guide/en/logstash/current/multiline.html)
page on Elastic. Running this again we see that the exception is now contained in one message instead of being
split across many messages:
```
{
    "@timestamp" => "2016-09-14T22:05:04.890Z",
       "message" => "2016-09-14 19:23:21,514 BrainTrust.host-8001  ERROR com.braintrust.WorkerFactory -   - Could not find the target file\n    java.io.IOException: Could not find the target file\n    at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)\n    at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:57)\n    at com.intellij.rt.execution.application.AppMain.main(AppMain.java:144)",
      "@version" => "1",
          "tags" => [
        [0] "multiline"
    ],
          "path" => "/tmp/sample-log.txt",
          "host" => "host.machine"
}
```
See how Logstash added the "multiline" tag to the event? You can add tags to different events to make them
easier to search for and also perform additional parsing on them. We will look more at this later.

You may have noticed that the timestamp on the event itself and the Logstash output do not match. Maybe you want to
pull out the class name or log level into their own field.
We will begin to address these and other issues this by introducing the Filter Plugins, specifically the Grok plugin.

## Filter Plugins
The [Filter Plugins](https://www.elastic.co/guide/en/logstash/current/filter-plugins.html) are what Logstash uses to
perform intermediate processing on an event. Filter plugins are added as another JSON document in a Logstash configuration file.
The one we will be focusing on is the Grok plugin. There are many different plugins available so make sure to give them a look over.

### Grok Filter Plugin
[Grok](https://www.elastic.co/guide/en/logstash/current/plugins-filters-grok.html) is a flexible parsing plugin which
uses a regex like syntax to pull apart message events and field data from them. Let us add a basic filter block with a
simple grok statement to make the timestamps from our log messages be set on each output record. Replace the contents
of ```/etc/logstash/conf.d/logstash-basic.conf``` with the following:

You can find the full file [here](src/main/resources/com/braintrust/logstash/02-Configuration_And_Grok/logstash-intermediate.conf)
```
input {
  file {
    path => ["/tmp/sample-log.txt"]
    discover_interval => 10
    sincedb_path => "/tmp/logstash_sincedb_file"
    start_position => "beginning"
    codec => multiline {
      pattern => "^%{TIMESTAMP_ISO8601} "
      negate => true
      what => "previous"
    }
  }
}

filter {
  grok {
    match => { "message" => "%{TIMESTAMP_ISO8601:tempTime}" }
  }
  date {
    match => [ "tempTime", "YYYY-MM-dd HH:mm:ss,SSS"]
    remove_field => [ "tempTime" ]
  }
}

output {
  stdout { codec => rubydebug }
}
```
The grok plugin here is saying match against the ```message``` field (Logstash default which contains the entire input event)
and use the following pattern ```%{TIMESTAMP_ISO8601}``` and add it to a field called ```tempTime```.  The timestamp pattern
we are using to match is built in, the %{PATTERN} syntax is used to activate a built in pattern. More information on those
can be found on the grok filter plugin page.

### Date Plugin
The [Date](https://www.elastic.co/guide/en/logstash/current/plugins-filters-date.html#plugins-filters-date-target) plugin
is used to parse dates out of fields and put them into a destination field. By default, the date filter will place
any sucessfully parsed date into the ```@timestamp``` field, which is our desired behavior. Notice, we are also removing
our ```tempTime``` field using the remove_field configuarion option. Now all of our dates will have the correct timestamp
in GMT format. You can change the timezone and several other options, please consult the documentation for additional options.
Here is some sample output with these filter plugins enabled and configured:
```
{
    "@timestamp" => "2016-09-14T23:23:21.442Z",
       "message" => "2016-09-14 19:23:21,442 BrainTrust.host-8001  DEBUG com.braintrust.http.controller.Controller -   - Controller: init",
      "@version" => "1",
          "path" => "/tmp/sample-log.txt",
          "host" => "host.machine"
}
```

## Next Steps
Now that we have taken a look at configuring Logstash to tail real files on the file system, gained an understanding
of the Logstash configuration, and setup some filter plugin's we are ready to move on to advanced grok techniques in
[Module 03 - Advanced Grok Concepts](03-Advanced_Grok_Concepts.md)



## Gothca's / Troubleshooting
### Continuous Output
If your Logstash keeps spitting out the log messages over and over, check to ensure that the directory configured
for the ```sincedb_path``` is readable and writable by Logstash. Otherwise, it will not be able to record its position
in the file and will continually reprocess it.

### No Output When Starting
Did you remember to delete Logstash's sincedb file (/tmp/logstash_sincedb_file in our example)? If Logstash has ran once
and already parsed the file, when it comes up it will think it has already parsed the sample file. You will need to
delete the sincedb file in order for Logstash to "discover" the file again.

### _dateparsefailure tag
If you are seeing _dateparsefailure in your output tags and do not see the correct timestamp, your date filter is
not successfully parsing the date out of the tempTime field. Consult the documentation on the Joda time date
parsing syntax to troubleshoot this issue further.
