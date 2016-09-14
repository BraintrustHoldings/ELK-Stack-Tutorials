# Logstash Tutorials

  * [Logstash Tutorials](#logstash-tutorials)
    * [Introduction](#introduction)
    * [Requirements](#requirements)
      * [Installing Logstash](#installing-logstash)
    * [Next Steps](#next-steps)

Created by [gh-md-toc](https://github.com/ekalinin/github-markdown-toc)

## Introduction

This repository contains a series of tutorials outlining the installation and configuration of
[Logstash](https://www.elastic.co/products/logstash).

## Requirements
- [Java 1.8](http://www.oracle.com/technetwork/java/javase/overview/index.html)
- [CentOS 7](https://www.centos.org/download/)
- [Logstash 2.x](https://www.elastic.co/downloads/logstash)

This tutorial assumes that you already have a CentOS 7 machine running with Java 1.8

### Installing Logstash

You can find the instructions for installing Logstash using yum for CentOS
[here](https://www.elastic.co/guide/en/logstash/2.3/installing-logstash.html)
on the official Elastic website. Once the RPM is installed, verify the
package is present by running

```
service logstash status
```

## Next Steps
Now that we have logstash installed, we can continue to
[Module 1 - Basic Commands](01-Basic_Usage.md)



