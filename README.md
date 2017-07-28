# Laika BOSS: Object Scanning System

Laika BOSS is a versatile object scanner and intrusion detection system. 

## Documentation

See the [Wiki](https://github.com/lmco/laikaboss/wiki) for documentation, examples, and other useful information.

Read the ***[whitepaper](http://lockheedmartin.com/content/dam/lockheed/data/isgs/documents/LaikaBOSS%20Whitepaper.pdf)***

## Overview

Laika is an object scanner and intrusion detection system that strives to achieve the following goals:

+ **Scalable**
	+ Work across multiple systems
	+ High volume of input from many sources

+ **Flexible**
	+ Modular architecture
	+ Highly configurable dispatching and dispositioning logic
	+ Tactical code insertion (without needing restart)

+ **Verbose**
	+ Generate more metadata than you know what to do with

Each scan does three main actions on each object:

+ **Extract child objects** Some objects are archives, some are wrappers, and others are obfuscators. Whatever the case may be, find children objects that should be scanned recursively by extracting them out.

+ **Mark flags** Flags provide a means for dispositioning objects and for pivoting on future analysis.

+ **Add metadata** Discover as much information describing the object for future analysis.

## Example Usecase
The best way to introduce Laika BOSS is to give several examples of its use.

In example one, you feed Laika an email with Office document (OLE) attachment. Laika will parse the contents of the email and extract all of the message objects. In this case, it extracts a plain text object and an Office Word attachment. Before moving on, it generates metadata about the email (e.g. email addreses, IPs, domains, etc.). Next Laika moves on and determines that the Word document is in OLE format so it extracts the OLE streams. In one one of the streams, a VBA macro is discoverd so Laika extracts that too. All objects feed into and extracted by Laika are scanned by Yara and ClamAV. The conclusion is an output of the scan results and collected metadata in JSON format. Optionally, Laika will place the extracted contents into a folder for manual review.

In example two, you feed Laika a ZIP file. Laika extracts the single item from the ZIP file. It determines that the extracted item is an RTF. It extracts all of the embedded objects from the RTF of which one is an EXE. Liaka collects metadata on the EXE. The conclusion is an output of the scan results and collected metadata in JSON format. Optionally, Laika will place the extracted contents into a folder for manual review.

## Components

Laika is composed of the following pieces:

+ **Framework** (`laika.py`) This is the core of Laika BOSS. It includes the object model and the dispatching logic.

+ **laikad** This piece contains the code for running Laika as a deamonized, networked service using the ZeroMQ broker.

+ **cloudscan** A command-line client for sending a local system file to a running service instance of Laika (laikad).

+ **modules** The scan itself is composed of the running of modules. Each module is its own program that focuses on a particular sub-component of the overall file analysis.


## Getting Started



### Full Installation Instructions
Laika BOSS has been tested on the latest versions of CentOS, Fedora, and Ubuntu LTS

See the [Wiki](https://github.com/lmco/laikaboss/wiki)

#### Milter Integration

The Laika BOSS milter server allows you to integrate Laika BOSS with mail transfer agents such as Sendmail or Postfix. This enables better visibility (passive visibility can be hampered by TLS) and provides a means to block email according to Laika BOSS disposition.

```
+----------------+             +---------------+             +----------------+
|                |    email    |               |   email     |                |
|    sendmail    +------------->  laikamilter  +------------->     laikad     |
|                | accept/deny |               | scan result |                |
|                <-------------+               <-------------+                |
+----------------+             +---------------+             +----------------+
```

The Laika BOSS milter server requires the [python-milter](https://pythonhosted.org/milter) module and the Laika BOSS client library. Check out the comments in the source code for more details.

#### Suricata Integration Prototype

We have released a proof of concept feature for Suricata that allows it to store extracted files and their associated metadata in a Redis database. You will find this code under a [new branch](https://github.com/lmco/suricata/tree/file_extract_redis_prototype_v1) in our Suricata fork. We hope to refine the implementation and eventually have it accepted by the project.

Once you've enabled file extraction and the optional Redis integration in Suricata, you can extract these files from Redis and submit them to Laika BOSS for scanning by using the middleware script `laika_redis_client.py` as shown below. Note that it requires the `python-redis` module.

First, start `laikad.py` in async mode:

```shell
./laikad.py -a
```

Then launch the middleware script and give it the address of the `laikad` broker and Redis database (defaults shown below):

```shell
./laika_redis_client.py -b tcp://localhost:5558 -r localhost -p 6379
```

Note that you will need to use a logging module such as `LOG_FLUENT` to export the full scan result of the these file scans from `laikad`.

## Licensing

The Laika framework and associated modules are released under the terms of the Apache 2.0 license.
