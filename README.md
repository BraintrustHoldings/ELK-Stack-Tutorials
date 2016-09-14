# ELK Stack Tutorials

  * [ELK Stack Tutorials](#elk-stack-tutorials)
    * [Introduction](#introduction)
    * [Requirements](#requirements)
    * [ELK Component Overview](#elk-component-overview)
      * [ElasticSearch](#elasticsearch)
      * [Logstash](#logstash)
      * [Kibana](#kibana)
      * [Redis](#redis)
    * [Getting Started](#getting-started)

Created by [gh-md-toc](https://github.com/ekalinin/github-markdown-toc)

## Introduction

This repository contains a series of tutorials outlining the installation and configuration of the ELK
([ElasticSearch](https://www.elastic.co/products/elasticsearch), [Logstash](https://www.elastic.co/products/logstash), and
[Kibana](https://www.elastic.co/products/kibana)) stack. These three products together can offer a powerful, scalable way
to parse, store, retrieve, and visualize application logs and metrics. The outline of the tutorials for each product
will start with basic setup and installation, configuration, and several real world examples. These tutorias will have
various requirements that are listed below.

Each module will contain its own README.md with that module's specific requirements. Here we have listed all requirements
and assumptions needed across all different tutorial modules.

## Requirements
- [Java 1.8](http://www.oracle.com/technetwork/java/javase/overview/index.html)
- [CentOS 7](https://www.centos.org/download/)
- [Logstash 2.x](https://www.elastic.co/downloads/logstash)
- [ElasticSearch 2.x](https://www.elastic.co/downloads/elasticsearch)
- [Kibana 4.x](https://www.elastic.co/downloads/kibana)
- [Redis 3.2.3](http://redis.io/download)

Below you will find Elastic's brief summary of each component in the ELK stack. This is not meant to be comprehensive
and if you wish to learn more, please visit the [Elastic](https://www.elastic.co/) website for more information.

While Redis is not included with the ELK stack or an Elastic product, we will be using it as a messaging queue
between logstash and ElasticSearch. Please visit [Redis](http://redis.io/download) for a more detailed description
of this product.

## ELK Component Overview

### ElasticSearch

Elasticsearch is a distributed, open source search and analytics engine, designed for horizontal scalability,
reliability, and easy management. It combines the speed of search with the power of analytics via a sophisticated,
developer-friendly query language covering structured, unstructured, and time-series data.

### Logstash

Logstash is a flexible, open source data collection, enrichment, and transportation pipeline.
With connectors to common infrastructure for easy integration,
Logstash is designed to efficiently process a growing list of log, event, and unstructured data sources
for distribution into a variety of outputs, including Elasticsearch.

### Kibana

Kibana is an open source data visualization platform that allows you to interact with your data through
stunning, powerful graphics. From histograms to geomaps, Kibana brings your data to life with visuals that
can be combined into custom dashboards that help you share insights from your data far and wide.

### Redis

Redis is an open source (BSD licensed), in-memory data structure store, used as database, cache and
message broker. It supports data structures such as strings, hashes, lists, sets, sorted sets with
range queries, bitmaps, hyperloglogs and geospatial indexes with radius queries. Redis has built-in
replication, Lua scripting, LRU eviction, transactions and different levels of on-disk persistence,
and provides high availability via Redis Sentinel and automatic partitioning with Redis Cluster.


## Getting Started

Now that we have gone through the overview and requirements, head over to the [Logstash](Logstash/README.md)
module to get started.

