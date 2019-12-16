#Consuming Data

## Introduction

The first step in a streaming integration flow is to consume the data to be cleaned, enriched, transformed or summarized 
to produce the required output. 
 
For the Streaming Integrator to consume events, the following is required.

* A message schema: The Streaming Integrator identifies the messages that it selects into a streaming integration flows by their schemas. The schema based on which the messages are selected are defined via a *stream*.

* A source: The messages are consumed from different sources including streaming applications, cloud-based applications, databases, and files. The source is defined via a *source configuration*.
  ![Receiving events](../images/consuming-messages/ConsumingMessages.png)
  
  A source configuration consists of the following:
  
  + **`@source`**: This annotation defines the source type via which the messages are consumed, and allows you to configure the source parameters (which change depending on the source type). For the complete list of supported source types, see [Siddhi Query Guide - Source](https://siddhi.io/en/v4.x/docs/query-guide/#source)
  + **`@map`**: This annotation specifies the format in which messages are consumed, and allows you to configure the mapping parameters (which change based of the mapping type/format selected). For the complete list of supported mapping types, see [Siddhi Query Guide - Source Mapper](https://siddhi.io/en/v4.x/docs/query-guide/#source-mapper)
  + **`@attributes`**: This annotation specifies a custom mapping based on which events to be selected into the streaming integration flow are identified. This is useful when the attributes of the incoming messages you want the Streaming Integrator to consume are different to the corresponding attribute name in the stream definition. e.g., In a scenario where the Streaming Integrator is reading employee records, the employee name might be defined as `emp No` in the database from which you are extracting the records. However, the corresponding attribute name in the stream definition is `employeeNo` because that is how yoy want to refer to the attribute in the streaming integration flow. In this instance, you need a custom mapping to indicate that `emp No` is the same as `employeeNo`.
  
  
  In a Siddhi application, you can define a source configuration inline or refer to a source configuration defined externally in a configuration file.

## Defining event source inline in the Siddhi application

To create a Siddhi application with the source configuration defined inline, follow the steps below.

1. Open the Streaming Integrator Studio and start creating a new Siddhi application. For more information, see [Creating a Siddhi Application](../develop/creating-a-Siddhi-Application.md).

2. Enter a name for the Siddhi application as shown below.<br/>
   `@App:name("<Siddhi_Application_Name>)`<br/>e.g., `@App:name("SalesTotalsApp")`<br/>
   
3. Define an input stream to define the schema based on which input events are selected to the streaming integrator flow as follows.

    `define stream <Stream_Name>(attribute1_name attribute1_type, attribute2_name attribute2_type, ...)`<br/>
    e.g., `define stream ConsumeSalesTotalsStream (transNo int, product string, price int, quantity int, salesValue long)`<br/>
    
    
4. Connect a source to the input stream you added as follows.
    ```
    @source(type='<SOURCE_TYPE>')
    define stream <Stream_Name>(attribute1_name attribute1_type, attribute2_name attribute2_type, ...);
    ```
    e.g., You can add a source of the `http` type to the `ConsumeSalesTotalsStream` input stream in the example of the previous step.
    ```
    @source(type='<http>')
    define stream ConsumeSalesTotalsStream (transNo int, product string, price int, quantity int, salesValue long);
    ```
    
5. Configure parameters for the source you added.
   e.g., You can specify an HTTP URL for the `http` source in the example used.
   
       ```
       @source(type='http', receiver.url='http://localhost:5005/SalesTotalsEP')
       define stream ConsumeSalesTotalsStream (transNo int, product string, price int, quantity int, salesValue long);
       ```
       
6. Add an `@map` annotation to the source configuration as shown below.
    ```
    @source(type='<SOURCE_TYPE>', <PARAMETER1_NAME>='<PARAMETER1_VALUE>', @map(type='<MAP_TYPE>'))
    define stream <Stream_Name>(attribute1_name attribute1_type, attribute2_name attribute2_type, ...);
    ```
    e.g., In the example used, you can specify `JSON` as the map type as follows:
    ```
    @source(type='http', receiver.url='http://localhost:5005/SalesTotalsEP', @map(type='json'))
    define stream ConsumerSalesTotalsStream(transNo int, product string, price int, quantity int, salesValue long);
    ```
    
    !!!info
        Mapping is explained in detail in the [Consuming a message in default format](/#consuming-a-message-in-default-format) and [Consuming a message in custom format](#consuming-a-message-in-custom-format)sections.
        However, note that you need to add a mapping type to complete a source configuration. If no mapping type is specified, an error is indicated.
        
7.  Add a Siddhi query to specify how the output is derived and the name of an output stream to which this output is directed.
    ```
    from <INPUT_STREAM_NAME>
    select <ATTRIBUTE1_Name>, <ATTRIBUTE2_NAME>, ... 
    group by <ATTRIBUTE_NAME>
    insert into <OUTPUT_STREAM_NAME>;
    ```
    
    e.g., Assuming that you are publishing the events with the existing values as logs in the output console without any further processing, you can define the query as follows.
    
    ```
    from ConsumerSalesTotalsStream
    select *
    group by product
    insert into PublishSalesTotalsStream;
    ```
    
8. Complete the Siddhi application by defining an output stream with a connected sink configuration.

    !!!tip
        In the example used, you can define the `PublishSalesTotals` stream that you already specified as the output stream in the query, and connect a `log` sink to it as follows. Publishing the output is explained in detail in the [Publishing Data guide](publishing-data.md).
        
        ```
        @sink(type='log', prefix='Sales Totals:')
        define stream PublishSalesTotalsStream (transNo int, product string, price int, quantity int, salesValue long);
        ```
        
        
9. Save the Siddhi Application. The completed application is as follows:
    ```
    @App:name("SalesTotalsApp")
    @App:description("Description of the plan")
    
    @source(type='http', receiver.url='http://localhost:5005/SalesTotalsEP', @map(type='json'))
    define stream ConsumerSalesTotalsStream(transNo int, product string, price int, quantity int, salesValue long);
    
    @sink(type='log', prefix='Sales Totals:')
    define stream PublishSalesTotalsStream (transNo int, product string, price int, quantity int, salesValue long);
    
    from ConsumerSalesTotalsStream
    select *
    group by product
    insert into PublishSalesTotalsStream;
    ```

    

## Defining event source externally in the configuration file

If you want to use the same source configuration in multiple Siddhi applications, you  can define it externally in the 
`<SI_HOME>/conf/server/deployment.yaml` file and then refer to it from Siddhi applications. To understand how to do this, 
follow the procedure below.


1. Open the `<SI_HOME>/conf/server/deployment.yaml` file.

2. Add a section named `siddi`, and then add a subsection named `refs:` as shown below.

    ```    
    siddhi:  
     refs:
      -
    ```
    
3. In the `refs` subsection, enter a parameter named `name` and enter a name for the source.
    ```    
    siddhi:  
     refs:
      -
       name:`<SOURCE_NAME>`
    ```
    
4. To specify the source type, add another parameter named `type` and enter the relevant source type.
    ```
    siddhi:  
     refs:
      -
       name:'<SOURCE_NAME>'
       type: '<SOURCE_TYPE>'
    ```
    
5. To configure other parameters for the source (based on the source type), add a subsection named `properties` as shown below.
    ```
    siddhi:  
     refs:
      -
       name:'SOURCE_NAME'
       type: '<SOURCE_TYPE>'
       properties
           <PROPERTY1_NAME>:'<PROPERTY1_VALUE>'
           <PROPERTY2_NAME>:'<PROPERTY2_VALUE>'
           ...
    ```
    
6. Save the configuration file.

e.g., The HTTP source used as the example in the previous section can be defined externally as follows:
```
siddhi:  
 refs:
  -
   name:'HTTPSource'
   type: 'http'
   properties
       receiver.url:'http://localhost:5005/SalesTotalsEP'
```

The source configuration you added can be referred to in a Siddhi application as follows:
```
@source(ref='SOURCE_NAME')
define stream ConsumeSalesTotalsStream (transNo int, product string, price int, quantity int, salesValue long);
```

e.g., The HTTP source that you previously created can be referred to as follows.

```
@source(ref='HTTP')
define stream ConsumeSalesTotalsStream (transNo int, product string, price int, quantity int, salesValue long);
```

### Supported event source types
<source categories table here> 

### Supported message formats

#### Consuming a message in default format

SI consumes a message in the default format when it makes no changes to the names of the attributes of the message 
schema before it processes the message. To understand how messages are consumed in default format, follow the procedure below.


1. Create a Siddhi application with a source configuration following the instructions in the [Defining event source inline in Siddhi Application](#defining-event-source-inline-in-the-siddhi-application) subsection.

2. In the source configuration, make sure that an `@map` annotation is included with the mapping type as shown below.
    ```
    @source(type='<SOURCE_TYPE>', <PARAMETER1_NAME>='<PARAMETER1_VALUE>', @map(type='<MAP_TYPE>'))
    define stream <Stream_Name>(attribute1_name attribute1_type, attribute2_name attribute2_type, ...);
    ```
    
    The map type specifies the format in which the messages are received.
    e.g., In the example used, you can specify `JSON` as the map type as follows:
    
    ```
    @source(type='http', receiver.url='http://localhost:5005/SalesTotalsEP', @map(type='json'))
    define stream ConsumerSalesTotalsStream(transNo int, product string, price int, quantity int, salesValue long);
    ```
    
3. Save the Siddhi application. If you save the Siddhi application that was created using the example configurations, the completed Siddhi application is as follows.

    ```
    @App:name("SalesTotalsApp")
    @App:description("Description of the plan")
    
    @source(type='http', receiver.url='http://localhost:5005/SalesTotalsEP', @map(type='json'))
    define stream ConsumerSalesTotalsStream(transNo int, product string, price int, quantity int, salesValue long);
    
    @sink(type='log', prefix='Sales Totals:')
    define stream PublishSalesTotalsStream (transNo int, product string, price int, quantity int, salesValue long);
    
    from ConsumerSalesTotalsStream
    select *
    group by product
    insert into PublishSalesTotalsStream;
    ```
    
4. To check whether the above Siddhi application works as expected, generate some messages.

    e.g., In the example used, the source type is HTTP. Therefore, you can issue a few curl commands similar to the following:
    
    ```
    curl -X POST \
      http://localhost:5005/SalesTotalsEP \
      -H 'content-type: application/json' \
      -d '{
      "event": {
        "transNo": "001",
        "product": "DDT",
        "price": "100",
        "quantity": "100",
        "salesValue" "10000"
      }
    }'
    ```

#### Consuming a message in custom format

SI consumes a message in the custom format when it makes changes to the names of the attributes of the message 
schema before it processes the message. To understand how messages are consumed in custom format, follow the procedure below.

!!!info
    For this section, you can edit the same Siddhi application that you saved in the [Consuming a message in default format](#consuming-a-message-in-default-format)subsection.
    
1. Open your Siddhi application with a source configuration.

2. In the @map annotation within the source configuration, add the `@attributes` annotation with mappings for different attributes. This can be done in two ways as shown below.

    - Defining attributes as keys and mapping content as values in the following format.

       ```
       @source(type='<SOURCE_TYPE>', <PARAMETER1_NAME>='<PARAMETER1_VALUE>', @map(type='<MAP_TYPE>', @attributes( attributeN='mapping_N', attribute1='mapping_1')))
       define stream <Stream_Name>(attribute1_name attribute1_type, attribute2_name attribute2_type, ...);
       ```
        
       e.g., In the Siddhi application used as an example in the previous section, assume that when receiving events, the `transNo` attribute is received as `transaction` and the `salesValue` attribute is received as `sales`.  The mapping type is JSON. therefore, you can  add the mappings as JSONPath expressions.

       | **Stream Attribute Name** | **JSON Event Attribute Name** | **JSONPath Expression** |
       |---------------------------|-------------------------------|-------------------------|
       | `transNo`                 | `transaction`                 | `$.transaction`         |
       | `salesValue`              | `sales`                       | `$.sales`               |
       
         The mapping can be defined as follows.
      
       ```
       @source(type='http', receiver.url='http://localhost:5005/SalesTotalsEP', @map(type='json', @attributes(transNo = '$.transaction', salesValue = '$.sales')))
       define stream ConsumerSalesTotalsStream(transNo int, product string, price int, quantity int, salesValue long);
       ```
        
    - Defining the mapping content of all attributes in the same order as how the attributes are defined in stream definition.

        ```
        @source(type='<SOURCE_TYPE>', <PARAMETER1_NAME>='<PARAMETER1_VALUE>', @map(type='<MAP_TYPE>', @attributes( 'mapping_1', 'mapping_N')))
        define stream <Stream_Name>(attribute1_name attribute1_type, attributeN_name attributeN_type, ...);
        ``` 
        
    e.g., If you consider the same example, mapping can be defined as follows.
              
    ```
    @source(type='http', receiver.url='http://localhost:5005/SalesTotalsEP', @map(type='json', @attributes(transNo = '$.transaction', product = product, quantity = quantity, salesValue = '$.sales')))
    define stream ConsumerSalesTotalsStream(transNo int, product string, price int, quantity int, salesValue long);
    ```
   
   
## Alerting the End of File after File Read

WSO2 Siddhi [file(source)](https://siddhi-io.github.io/siddhi-io-file/api/2.0.3/#file-source) provides the functionality for the user to feed data to the Siddhi from files and both text and binary files are supported by it.
Here, we are discussing using a property to identify the end of the file we read.

To create a sample Siddhi application with the source configuration defined inline, follow the steps below.

1. Open the Streaming Integrator Studio and start creating a new Siddhi application.
For more information, see [Creating a Siddhi Application](../develop/creating-a-Siddhi-Application.md). <br/>

2. Enter a name for the Siddhi application as shown below.<br/>
          
        @App:name("FileReadApp")
    
3. Define an input stream to define the schema based on which input events are selected to the streaming integrator flow as follows. 
Note that the end of file property (‘eof’) type is boolean and it’ll pass ‘true’ in the last event. The rest of the events, it will be false. <br/>
  
        define stream InStream (A string, B int, C string, eof bool, fp String);
    
4. Connect a source to the input stream as follows. <br/>
  You can add a source of the file type to the InStream input stream in the example of the previous step.<br/>
      
        @source(type = 'file', dir.uri = 'file:/home/methma/Desktop/resources/toprocess/', move.after.process = 'file:/home/methma/Desktop/resources/processed', move.after.failure = 'file:/home/methma/Desktop/resources/failure/', mode = 'LINE', tailing = 'false', action.after.process = 'MOVE', action.after.failure = 'MOVE', add.event.separator = 'false', @map(...)
      
5. Add an @map annotation to the source configuration as shown below.  
Note that the attribute mappings for the end of the file and the file path are 'trp:eof' and 'trp:file.path' respectively. <br/>

        @source(type = 'file', dir.uri = 'file:/home/methma/Desktop/resources/toprocess/', move.after.process = 'file:/home/methma/Desktop/resources/processed', move.after.failure = 'file:/home/methma/Desktop/resources/failureA = "0", B = "1", C = "2",A = "0", B = "1", C = "2",/', mode = 'LINE', tailing = 'false', action.after.process = 'MOVE', action.after.failure = 'MOVE', add.event.separator = 'false',
 
           @map(type='csv',header='false', @attributes(A = "0", B = "1", C = "2", eof = 'trp:eof', fp = 'trp:file.path')))

6. Add a Siddhi query to check the last event from the file and insert it to an alert stream.<br/>

          @info(name = 'SiddhiAppFileRead') 
          from InStream[ eof == true ]#log("last event ####")
          select *
          insert into AlertStream;
 
7. Complete the Siddhi application by defining an output stream with a connected sink configuration. <br/>  

        define stream OutStream (A string, B int, C string, eof bool, fp String);
  
8. Save the Siddhi Application. The completed application is as follows: <br/>
  ```
      @App:name("SiddhiAppFileRead")
      @App:description("Description of the plan")
     
      @source(type = 'file', dir.uri = 'file:/home/methma/Desktop/resources/toprocess/', move.after.process = 'file:/home/methma/Desktop/resources/processed', move.after.failure = 'file:/home/methma/Desktop/resources/failuer/', mode = 'LINE', tailing = 'false', action.after.process = 'MOVE', action.after.failure = 'MOVE', add.event.separator = 'false',
      @map(type='csv',header='false', @attributes(A = "0", B = "1", C = "2", eof = 'trp:eof', fp = 'trp:file.path')))
     
      define stream InStream (A string, B int, C string, eof bool, fp String);
      define stream AlertStream (A string, B int, C string, eof bool, fp String);
     
      @info(name = 'SiddhiAppFileRead')
      from InStream[ eof == true ]#log("last event ####")
      select *
      insert into AlertStream;
  ```
Furthermore, it is possible to read a text file and extract data using a regex.
In that case, the last event will pass the end of the file property true.

Following is a sample Siddhi app for the Regex use case. <br/>
```
   @App:name("RegexCSV")
   @App:description("Description of the plan")
  
   @source(type = 'file', dir.uri = "file:/home/methma/Desktop/resources/toprocess/", mode = "REGEX", begin.regex = "yes",  end.regex = "80",  tailing = "false", action.after.process = "MOVE", action.after.failure = "MOVE",
     move.after.process = "file:/home/methma/Desktop/resources/processed/", move.after.failure = "file:/home/methma/Desktop/resources/failure/", add.event.separator = 'false',
     file.read.wait.timeout = "1000",
     @map(type='csv',header='false', @attributes(symbol = "0", price = "1", volume = "2", eof = 'trp:eof', fp = 'trp:file.path')))
   define stream FooStream (symbol string, price int, volume int, eof bool, fp String);
  
   @info(name = 'RegexCSV')
   define stream barStream (symbol String, price int, volume int, eof bool, fp string);
   from FooStream
   select *
   insert into barStream;
``
