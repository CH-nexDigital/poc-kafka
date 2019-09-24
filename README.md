# poc-kafka

# Prerequisite

- [Confluent Platform 5.3.1](https://docs.confluent.io/5.3.1/installation/index.html)
- [Elastisearch 7.3](https://www.elastic.co/guide/en/elasticsearch/reference/7.3/install-elasticsearch.html)
- [Logstash 7.3](https://www.elastic.co/guide/en/logstash/7.3/installing-logstash.html)
- [Kibana 7.3](https://www.elastic.co/guide/en/kibana/7.3/install.html)

# Part 1 : Configuring Kafka

## Create the topics 

1. Open Confluent Control Center with a browser (http://localhost:9021).

2. Create a topic ***log2kibana*** in the Topics submenu.

![alt text](img/topic-creation.PNG?raw=true)

## Create the spoolDirConnector

1. Navigate to the Connect Menu and select the default connect cluster.
2. Create a new connector by clicking on the *add connector* button.
3. Select the **SpoolDirCsvSourceConnector** connector.
4. Give it a name, for example *csv-connector*
5. In the **Common** part, fill the fields like following and leave others blank:
   - Key converter class : &nbsp;&nbsp; org.apache.kafka.connect.json.JsonConverter
   - Value converter class : &nbsp;&nbsp; org.apache.kafka.connect.json.JsonConverter
 
  ![alt text](img/common.PNG?raw=true)
6. In the **General** section, fill the fields as following and leave others blank:
   - topic : &nbsp;&nbsp; log2kibana
  
  ![alt text](img/general.PNG?raw=true)
7. In the **File System**, fill the filds as following and leave others blank:
   - input.path: &nbsp;&nbsp; *folder wich contains files to process*
   - finished.path : &nbsp;&nbsp; *folder for finished files*
   - error.path : &nbsp;&nbsp; *folder for files having an error*
   - input.file.pattern : &nbsp;&nbsp; ^.*\.csv$
  > Make sure that the three folders exists !
  
  ![alt text](img/filesystem.PNG?raw=true)
8. In the **Schema** section:
   - key.schema : &nbsp;&nbsp; {"name":"com.github.jcustenborder.kafka.connect.model.Key","type":"STRUCT","isOptional":false,"fieldSchemas":{}}
   - value.schema : &nbsp;&nbsp; 
  {"name":"com.github.jcustenborder.kafka.connect.model.Value","type":"STRUCT","isOptional":false,"fieldSchemas":{"Transaction_date":{"type":"STRING","isOptional":true},"Product":{"type":"STRING","isOptional":true},"Price":{"type":"INT32","isOptional":true},"Payment_Type":{"type":"STRING","isOptional":true},"Name":{"type":"STRING","isOptional":true},"City":{"type":"STRING","isOptional":true},"State":{"type":"STRING","isOptional":true},"Country":{"type":"STRING","isOptional":true},"Account_Created":{"type":"STRING","isOptional":true},"Last_Login":{"type":"STRING","isOptional":true},"Latitude":{"type":"STRING","isOptional":true},"Longitude":{"type":"STRING","isOptional":true}}}
  
  ![alt text](img/schema.PNG?raw=true)

9. **Schema Generation** section: 
    - schema.auto.generation : &nbsp;&nbsp; false

![alt text](img/schema2.PNG?raw=true)

10. **CSV Parsing** section: 
    - Treat first row as header : &nbsp;&nbsp; true
  
![alt text](img/csvparsing.PNG?raw=true)

11. In the **Additional Properties** section, please add a property named **value.converter.schemas.enable** and set its value to false.
    
 *This option allow you to remove the schema in the messages.*

![alt text](img/additionalprop.PNG?raw=true)

Once you fill all those fields, launch the connector.


# Part 2 : Configuring Logstash

You should add a pipeline to connect Logstash and Kafka.
1. Create a configuration file **/etc/logstash/conf.d/logstash-kafka.conf**
```
$ touch /etc/logstash/conf.d/logstash-kafka.conf
```

2. Configure the /etc/logstash/conf.d/logstash-kafka.conf to match the following:
```apache
input {
  kafka {
    id => "my_plugin_id"
    topics => ["log2kibana"]
    bootstrap_servers => "localhost:9092"
    auto_offset_reset => "earliest"
  }
}
filter {
  json {
    source => ["message"]
  }
  prune {
    blacklist_names => ["message"]
  }

}
output {
  elasticsearch { 
  	hosts => ["localhost:9200"]
  	ilm_rollover_alias => "kafka-poc"
    ilm_pattern => "1"
  }
}
```
3. Configure the **/etc/logstash/pipelines.yml** file to match the following:
```apache
# This file is where you define your pipelines. You can define multiple.
# For more information on multiple pipelines, see the documentation:
#   https://www.elastic.co/guide/en/logstash/current/multiple-pipelines.html

- pipeline.id: kafka-poc
  path.config: "/etc/logstash/conf.d/logstash-kafka.conf"
```
4. Restart the logstash service to activate the pipeline that you just created.
```bash
# If you are using Systemd
$ sudo systemctl restart logstash.service

# If you are using Upstart
$ sudo initctl restart logstash

# If you are using SysV
$ sudo /etc/init.d/logstash restart
```

# Part 3 : Ingesting data to Kafka


1. Clone the repository

<pre>
$ git clone https://github.com/CH-nexDigital/poc-kafka
</pre>

2. Copy the data files to the input repository.

<pre>
$ cp data ./data/part1.csv <b>/path/to/the/input/folder/you/created/in/the/previous/part/</b>
$ cp data ./data/part2.csv <b>/path/to/the/input/folder/you/created/in/the/previous/part/</b>
</pre>

3. Check the data from terminal
Execute the following command in the terminal to verify that the CSV file data have been injested to Kafka.
<pre>
$ kafka-console-consumer --bootstrap-server localhost:9092 --topic log2kibana --from-beginning
</pre>

# Part 4 : Configuring Kibana

Navigate to the Kibana home page at <http://localhost:5601/>.
1. In the left menu bar, click on the **Management** icon.
   
![alt text](img/management-logo.png?raw=true)


2. Click on the **Index Patterns** and select **Create index pattern**.

![alt text](img/indexpattern.png?raw=true)


1. In the step 1, enter **kafka-poc-1** in the index pattern field.
   
![alt text](img/createindex.PNG?raw=true)

4. In the step 2, select **@timestamp** as the Time Filter name.

![alt text](img/timestamp.PNG?raw=true)

5. Create the index pattern.

You can get a visualization of the traffic in the **Discover** menu.
![alt text](img/traffic2.PNG?raw=true)



# Part 3: Visualisations
Create your own visualizations for monitoring.

![alt text](img/visu.PNG?raw=true)

# Part 4: Kafka Streams

Please refer to this [page](https://github.com/CH-nexDigital/kafka-streams-poc) for KStreams Processing.