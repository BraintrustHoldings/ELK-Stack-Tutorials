# Logstash Basic Usage

  * [Logstash Basic Usage](#logstash-basic-usage)
    * [Introduction](#introduction)
    * [Basic Commands](#basic-commands)
      * [Starting / Stopping / Status / Checkconfig](#starting--stopping--status--checkconfig)
    * [Configuring Logstash](#configuring-logstash)
      * [Basic Configuration](#basic-configuration)
        * [Input Plugins](#input-plugins)
          * [Generator Plugin](#generator-plugin)
        * [Output Plugins](#output-plugins)
        * [stdout plugin](#stdout-plugin)
      * [First Logstash Startup](#first-logstash-startup)
        * [Validate Configuration](#validate-configuration)
        * [Start Logstash](#start-logstash)
      * [Understanding the Output](#understanding-the-output)
    * [Gotcha's / Troubleshooting](#gotchas--troubleshooting)
    * [Next Steps](#next-steps)

Created by [gh-md-toc](https://github.com/ekalinin/github-markdown-toc)

## Introduction
This module will outling the basic usage of Logstash including starting, stopping,
and checking the status of the logstash service. In addition, an overview of the
configuration files and basic options will be given.

Logstash is a way to consume application logging output, parse it quickly into a JSON document,
and ship the parsed output to a given destination. This will be the main workhorse for transforming
log4j/logback files into a JSON format ready for consumption by ElasticSearch.

## Basic Commands
Since we used the RPM to install logstash, we have the following commands provided:

### Starting / Stopping / Status / Checkconfig
Starting Logstash:
```
service logstash start
```
Stopping Logstash:
```
service logstash start
```
Status of Logstash:
```
service logstash start
```
This will tell you if the logstash service is running or is stopped.

```
service logstash checkconfig
```
Since we have put any configuration files in ```/etc/conf.d/logstash```, Logstash will not
have anything to check against and will let us know. Let us take a look at adding a basic
configuration.

## Configuring Logstash
Logstash will read all the files contained in ```/etc/conf.d/logstash``` for its configuration.
It is important to note that Logstash will concatonate ***ALL*** files present in that directory
together to build its configuration. What this means is that if you are setting up multiple
configuration files to handle multiple input and output destinations, you will need to guard
each section carefully to not have multiple copies of each input sent to each destination output.
We will go over this in more detail in the future.

### Basic Configuration
To start, let's place the following basic config in the Logstash configuration directory ```/etc/conf.d/logstash```:

(Note: This file can also be found [here](src/main/resources/com/braintrust/logstash/01-Basic_Usage/logstash-basic.conf))
```
input {
  generator {
    lines => [
      "line 1",
      "line 2",
      "line 3"
    ]
    # Emit all lines 3 times.
    count => 3
  }
}

output {
  stdout { codec => rubydebug }
}
```
#### Input Plugins
This configuration file is a JSON formatted document broken in two important parts. First, we have the ```input plugins``` section.
The ```input plugin``` section is where you define the inputs that you will feed into Logstash for it to parse.
All of the available [input plugins]((https://www.elastic.co/guide/en/logstash/current/input-plugins.html)) can be found on the Elastic
website. We will start by using the [Generator](https://www.elastic.co/guide/en/logstash/current/plugins-inputs-generator.html) plugin.

##### Generator Plugin
This plugin is defined by the generator {} JSON document. Inside we define the lines that we wish to have printed and sent into
the logstash process. The count paramater says to print these lines three times.

### Output Plugins
The ```output plugin``` section is where you define the different destinations Logstash should send its parsed output.
There are many different [output plugins](https://www.elastic.co/guide/en/logstash/current/output-plugins.html) supported
and you can find the full listing on the Elastic website.

#### stdout plugin
The stdout output plugin simply tells Logstash to dump the parsed output of each line to stdout.

### First Logstash Startup
Now that we have our basic configuration file in place, let us run logstash for the first time.

#### Validate Configuration
Before we attempt to start Logstash, we should first validate our configuration:
```
service logstash configtest
Configuration OK
```
#### Start Logstash
With our configuration valid, we can now start Logstash:
```
service logstash start
```
Here is a sample output that should be printed on the screen once the process comes up:
```
{:timestamp=>"2016-09-13T20:09:49.987000-0400", :message=>"Pipeline main started"}
{
       "message" => "line 1",
      "@version" => "1",
    "@timestamp" => "2016-09-14T00:09:49.928Z",
          "host" => "BrainTrust.host.local",
      "sequence" => 0
}
{
       "message" => "line 2",
      "@version" => "1",
    "@timestamp" => "2016-09-14T00:09:49.985Z",
          "host" => "BrainTrust.host.local",
      "sequence" => 0
}
{
       "message" => "line 3",
      "@version" => "1",
    "@timestamp" => "2016-09-14T00:09:49.986Z",
          "host" => "BrainTrust.host.local",
      "sequence" => 0
}
{
       "message" => "line 1",
      "@version" => "1",
    "@timestamp" => "2016-09-14T00:09:49.987Z",
          "host" => "BrainTrust.host.local",
      "sequence" => 1
}
{
       "message" => "line 2",
      "@version" => "1",
    "@timestamp" => "2016-09-14T00:09:49.987Z",
          "host" => "BrainTrust.host.local",
      "sequence" => 1
}
{
       "message" => "line 3",
      "@version" => "1",
    "@timestamp" => "2016-09-14T00:09:49.988Z",
          "host" => "BrainTrust.host.local",
      "sequence" => 1
}
{
       "message" => "line 1",
      "@version" => "1",
    "@timestamp" => "2016-09-14T00:09:49.988Z",
          "host" => "BrainTrust.host.local",
      "sequence" => 2
}
{
       "message" => "line 2",
      "@version" => "1",
    "@timestamp" => "2016-09-14T00:09:49.988Z",
          "host" => "BrainTrust.host.local",
      "sequence" => 2
}
{
       "message" => "line 3",
      "@version" => "1",
    "@timestamp" => "2016-09-14T00:09:49.988Z",
          "host" => "BrainTrust.host.local",
      "sequence" => 2
}
{:timestamp=>"2016-09-13T20:09:50.120000-0400", :message=>"Pipeline main has been shutdown"}
{:timestamp=>"2016-09-13T20:09:53.002000-0400", :message=>"stopping pipeline", :id=>"main"}

```
### Understanding the Output
As you can see, Logstash took our three input lines that we defined in the Generator input plugin, parsed each line
into a JSON document, and printed it to stdout. The generator plugin also specified to print those three line three times.
We can see that we had 9 total lines of output. Take a look at the following table to understand the fields and their
meaning in the output:

| Field      	| Description                                  	|
|------------	|----------------------------------------------	|
| message    	| The content of the line that Logstash parsed 	|
| @version   	| A versioning field automatically generated   	|
| @timestamp 	| The timestamp showing when the line was read 	|
| host       	| The host that generated the line             	|
| sequence   	| The iteration of the generator plugin        	|

The ``@timestamp`` field is very import. This is the field that ElasticSearch will use when plotting and storing your
events. It is important to ensure that each of your message being parsed by Logstash has an accurate timestamp. Otherwise,
your events could be stored in a wrong index or in an unexpected location.

## Next Steps
Now that we have successfully configured and started Logstash for the first time, we can being to look at more advanced
configuration options and perform our first Grok operations in [Module 2 - Configuration and Grok](02-Configuration_And_Grok.md).

## Gotcha's / Troubleshooting
If the Logstash process is failing to start, be sure that all the Logstash configuration files
are readable by the logstash user (this user should have been added by the RPM install).
