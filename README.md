# Security Logs Logstash Plugins Repository

**Are you using or planning to use ELK and need to digest logs from various products? You’re in the right place…**

empow’s SIEM Logstash pipeline is an open source repository containing logstash-based configuration pipelines for the digestion of logs generated by various products and vendors. The output is mapped to the [Elastic Common Schema (ECS)](https://www.elastic.co/guide/en/ecs/current/index.html). In some cases, additional fields that have security value and do not exist in the ECS can be added.

In addition, some plugins can be used to enrich security logs with information about the attacker’s intent according to the cyber-kill-chain and [MITRE ATT&CK](https://attack.mitre.org/) representation language, using empow’s [threat classification plugin](https://www.elastic.co/guide/en/logstash/current/plugins-filters-threats_classifier.html).

**What do you get?**

1. Parsers for logs from various products and vendors
2. Logstash pipeline-based structure to streamline the digestion of various logs
3. Output to Elastic (or other destination) based on ECS
4. Enrichment for intent classification, powered by empow (optional)
5. Since Logstash is configurable, you can modify the plugins and pipeline for your specific needs
6. Community to share and keep up-to-date plugins


**Note**: the plugins are based on log samples and vendors’ documentation. We are continuously updating and enriching the plugins. We encourage you to share examples and enhancements to improve the plugins and to keep them up-to-date with latest product versions. Please contact us at support@empow.co for and questions or updates.


## Getting Started

If you’re an ELK user and have Logstash and Elasticsearch installed, you’re all set.

The plugins are based on a typical ELK setup (if you don’t output to Elasticsearch you’ll need only Logstash). The only requirement is that the Logstash supports multiple pipelines feature (Logstash/ELK version 6.3 and above).
For downloading and installing Logstash please refer to [Logstash installation guide](https://www.elastic.co/downloads/logstash)

For downloading and installing ELK stack please refer to [ELK installation guide](https://www.elastic.co/downloads/).

**Recommended** : If you’d like to take advantage of the intent classification enrichment, powered by empow, you’ll need to install [empow’s threat classification plugin](https://www.elastic.co/guide/en/logstash/current/plugins-filters-threats_classifier.html) and register to [empow's threat classification center](https://empow.co/opensource/) (**both are for free!**) as described [further on in this section](https://github.com/empow/logstash-parsers/blob/master/README.md#log-enrichment-using-empows-threat-classification-plugin).

## Installation guide
#### What you will need?
1. [ELK installed](https://www.elastic.co/downloads/) (Logstash is the only mandatory component).
2. empow’s SIEM pipelines (optional)
#### Supported platform
The set of pipelines will run on any platform running Logstash version 6.3 and above.

We will use [Ubuntu 18.04](http://releases.ubuntu.com/18.04/) as the reference platform for this note.

### Get started with Logstash and the empow SIEM pipeline
Now that you have Logstash in place:

#### Configure Logstash
Add this line to *logstash.yml* to enable automatic loading of configuration files:

> config.reload.automatic: true

Install the required additional plugins:

```sh
sudo /usr/share/logstash/bin/logstash-plugin install logstash-filter-empowclassifier logstash-filter-translate logstash-filter-prune

```

Restart Logstash:

```sh
sudo service logstash restart

```

### Installing and Using the Pipelines
The set of Logstash pipelines consists of multiple pipelines connected using [pipeline-to-pipeline communication feature](https://www.elastic.co/guide/en/logstash/current/pipeline-to-pipeline.html) enabling to easily configure, add and maintain incoming log parsers.

![Pipeline-to-pipeline default configuration](https://github.com/empow/logstash-parsers/blob/master/images/pipeline_default.jpeg)
*Figure 1: Pipeline-to-pipeline default configuration*

Each configuration file contains a single pipeline. At the first stage, logs are entered to a virtual input pipeline that dispatches the incoming logs to a per product pipeline (a file per product). This dispatching is done based on keywords found in the logs or based on port numbers (per product UDP/TCP port number) depending on the virtual input (there are multiple available virtual inputs).

At the second stage, the logs are entered to a specific product pipeline that analyzes the log, extracts tokens and maps them according to the [Elastic Common Schema](https://www.elastic.co/guide/en/ecs/current/index.html). Once the log is analyzed it leaves the pipeline and enters the next stage that may be one of the available virtual outputs (e.g. Elastic virtual output that stores the data to Elastic DB). Optionally, the log may enter an additional common processing before leaving the pipelines, such as [threat classification](https://www.elastic.co/guide/en/logstash/current/plugins-filters-threats_classifier.html) (see below).

#### Download SIEM pipeline
To download and extract the pipelines:

```sh
wget https://github.com/empow/logstash-parsers/archive/master.zip
unzip master.zip
```

#### Logstash Configuration
Configuring Logstash [pipeline-to-pipeline](https://www.elastic.co/guide/en/logstash/current/pipeline-to-pipeline.html) is done by adding the set of pipelines to the *pipeline.yml* configuration file. For each pipeline file the following lines should be added:

> \- pipeline.id: <pipeline identifier> <br>
> path.config: <full path of the pipeline>

For instance, in order to support the default configuration depicted in *Figure 1* consisting of single port virtual input (that receives all the logs on a single UDP port and dispatches them based on keywords found in the log), Carbon Black, Snort, Fortinet, and  Symantec parsers, empow threat classification (for intent based enrichment) and Elastic virtual output (that stores the logs in Elasticsearch DB), the following lines should be added to pipline.yml (where <BASE_DIR> should be replaced by the path in which the pipelines were extracted:


> \- pipeline.id: single_port <br>
> path.config: "<BASE_DIR>/logstash-parsers/virtual_input/single_port.conf" <br>
> \- pipeline.id: elastic_output <br>
> path.config: "<BASE_DIR>/logstash-parsers/virtual_output/elastic_output.conf" <br>
> \- pipeline.id: empow_classifier_output <br>
> path.config: "<BASE_DIR>/logstash-parsers/virtual_output/empow_classifier_output.conf" <br>
> \- pipeline.id: default <br>
> path.config: "<BASE_DIR>/logstash-parsers/parsers/default_pipeline.conf" <br>
> \- pipeline.id: snort <br>
> path.config: "<BASE_DIR>/logstash-parsers/parsers/snort/snort.conf" <br>
> \- pipeline.id: fortinet <br>
> path.config: "<BASE_DIR>/logstash-parsers/parsers/fortinet/fortinet.conf" <br>
> \- pipeline.id: cb <br>
> path.config: "<BASE_DIR>/logstash-parsers/parsers/carbonblack/cbdefense.conf" <br>
> \- pipeline.id: symantec <br>
> path.config: "<BASE_DIR>/logstash-parsers/parsers/symantec/symantec.conf" <br>


### Pipeline Configuration
Each pipeline can be configured and modified by editing the file. Such a configuration may include: changing the incoming port, changing the Elasticsearch index name, adding or removing extracted fields, changing parser logic, adding or removing pipeline-to-pipeline staged, etc.

#### Input Configuration
Each virtual input can be added to the configuration by adding its identity (any unique name) and full path to the pipeline.yml configuration file. For instance, in order to add the single port virtual input, the following lines should be added:

> \- pipeline.id: single_port <br>
> path.config: "<BASE_DIR>/logstash-parsers/virtual_input/single_port.conf" <br>


Once a virtual input has been added (note that multiple inputs can be added), it can be configured by editing the pipeline itself.

For example, the default input in the single port virtual input is UDP port 2055, the port number can be changed by simply changing it to a different number:

<pre>
input { 
  udp{
    port => 2055
  }
}
filter{
...
</pre>
 &nbsp; &nbsp; &nbsp; &darr;
 
<pre>
input { 
  udp{
    port => <b>517</b>
  }
}
filter{
...
</pre>

the transport protocol can be changed from UDP to TCP

<pre>
input { 
  udp{
    port => 2055
  }
}
filter{
...
</pre>
 &nbsp; &nbsp; &nbsp; &darr;
 
<pre>
input { 
  <b>tcp</b>{
    port => 2055
  }
}
filter{
...
</pre>

For other types of available inputs and their configuration, refer to [Logstash input plugins reference guide](https://www.elastic.co/guide/en/logstash/current/input-plugins.html).

Dispatching the logs to specific parsers is done by searching for a specific keyword in the log (single port virtual input) or by assigning a dedicated input (e.g. dedicated port number) for each product (multi-port virtual input). In the first case, the set of keywords and regular expressions used (defined in the filter section in each conf file) can be refined and modified while adding a new product requires to add a new entry.


### Parser Configuration
Each product has its own parser pipeline that extracts relevant information using various [Logstash filter plugins](https://www.elastic.co/guide/en/logstash/current/filter-plugins.html). By default, the output section of the pipeline is determined to be Elasticsearch virtual output that stored the normalized log in the Elasticsearch database. The parser maps fields from the logs to the ECS schema. In some cases additional fields that do not exist in the ECS are required. You can modify the parser to suit your specific needs. You can add / remove / modify the fields and mapping.

## Log Enrichment using empow’s Threat Classification Plugin
To add empow’s intent threat classification to the output based on the setup described above, the following steps are required:

[Register to empow’s classification center](https://empow.co/register-joinus-2/) (**it’s free and no private information is sent to empow’s classification center**) – to register please fill out the form at the bottom of this page.
<u>NOTE</u>: the username and password in the registration form shall be used as the plugin credentials.
Use empow’s classification virtual output, by modifying the output of each parser as follows:

<pre>
output {  
  pipeline{
    send_to => [elastic_output]
  }
}
</pre>

&nbsp; &nbsp; &nbsp; &darr;


<pre>
output {  
  pipeline{
    send_to => [<b>empow_classifier_output</b>]
  }
}
</pre>

![Pipeline-to-pipeline classification configuration](https://github.com/empow/logstash-parsers/blob/master/images/pipeline_enrich.png)
*Figure 1: Pipeline-to-pipeline threat classification configuration*


Once the empow classification pipeline is used, it should be configured with the empow classification center credentials (the username and password as entered in the registration form).


<pre>
filter{
  if "empow_classification" in [tags] {
    empowclassifier{
      threat_field => "empow"
      product_type_field => "[observer][type]"
      product_name_field => "[observer][product]"
      bulk_request_interval => 1
      bulk_request_size => 50
      username => "<b>username</b>"
      password => "<b>password</b>"
    }

    ...  
</pre>

## Supported Products

Product Name  | Vendor Name  | Service Type
--------------|--------------|-------------
(**New!**) ASA| Cisco        | Firewall
Snort         | Snort        | IDS
Fortigate     | Fortinet     | IDS
cbDefence     | Carbon Black | EDR
SEP           | Symantec     | Anti Virus


**For any questions or clarifications please contact us at <support@empow.co>**

## Multi Pipeline Visualization
To validate and visualize your multi pipeline configuration use the [Logstash Multi Pipeline Visualization](https://github.com/empow/logstash-parsers/tree/master/tools) tool
