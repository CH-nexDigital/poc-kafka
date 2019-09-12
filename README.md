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


# Part 1 : Kafka & ELK



# Part 2 : Kafka Streams