#CSV2JSON
All you need nodejs csv to json converter. Support big json data, CLI, web server, powerful nested JSON, customised parser, stream, pipe, and more!

#IMPORTANT!!
Since version 0.3, the core class of csvtojson has been inheriting from stream.Transform class. Therefore, it will behave like a normal Stream object and CSV features will not be available any more. Now the usage is like:
```js
//Converter Class
var fs = require("fs");
var Converter = require("csvtojson").Converter;
var fileStream = fs.createReadStream("./file.csv");
//new converter instance
var converter = new Converter({constructResult:true});
//end_parsed will be emitted once parsing finished
converter.on("end_parsed", function (jsonObj) {
   console.log(jsonObj); //here is your result json object
});
//read from file
fileStream.pipe(converter);
```

To convert from a string, previously the code was:
```js
csvConverter.from(csvString);
```

Now it is:
```js
csvConverter.fromString(csvString, callback);
```

The callback function above is optional. see [Parse String](#parse-string).

After version 0.3, csvtojson requires node 0.10 and above.

##Menu
* [Installation](#installation)
* [Example](#example)
* [CLI Usage](#usage)
    * [CLI Tool](#command-line-tools)
    * [Web Service](#webservice)
* [Demo Product](#demo-product)
* [Quick Start](#quick-start)
* [Parameters](#params)
* [Customised Parser](#parser)
* [Webserver](#webserver)
* [Events](#events)
* [Built-in Parsers](#default-parsers)
* [Example](#example)
* [Big CSV File Streaming](#big-csv-file)
* [Process Big CSV File in CLI](#convert-big-csv-file-with-command-line-tool)
* [Column Array](#column-array)
* [Parse String](#parse-string)
* [Empowered JSON Parser](#empowered-json-parser)
* [Field Type](#field-type)
* [Change Log](#change-log)

GitHub: https://github.com/Keyang/node-csvtojson

##Installation
>npm install -g csvtojson


##Features

* Powerful library for you nodejs applications processing csv data.
* Extremly straight forward
* Multiple input support: CSV File, Readable Stream, CSV String etc.
* Highly extendible with your own rules and parsers for outputs.
* Multiple interfaces (webservice, command line)


##Usage

###Command Line Tools

>csvtojson <csv file path>

Example

>csvtojson ./myCSVFile

Or use pipe:

>cat myCSVFile | csvtojson

To start a webserver

>csvtojson startserver [options]

Advanced usage with parameters support, check help:

>csvtojson --help

### WebService
After webserve being initialised, it is able to use http post with CSV data as body.
For example, we start web server with default configuration:
>csvtojson startserver

And then we use curl to perform a web request:
>curl -X POST -d "date,\*json\*employee.name,\*json\*employee.age,\*json\*employee.number,\*array\*address,\*array\*address,\*jsonarray\*employee.key,\*jsonarray\*employee.key,\*omit\*id
>
>2012-02-12,Eric,31,51234,Dunno Street,Kilkeny Road,key1,key2,2
>
>2012-03-06,Ted,28,51289,Cambridge Road,Tormore,key3,key4,4" http://127.0.0.1:8801/parseCSV

#Demo Product
To write a demo app, simply use csvtojson web interface. Paste following code to index.js:

```js
var server = require("csvtojson").interfaces.web;
server.startWebServer({
	"port":8801
});
```
Then run the app:
```
node ./index.js
```
Now you can post any csv data to http://localhost:8801/parseCSV

It uses HTTP Request as readable stream and HTTP Response as writable stream.

# Quick Start
Use csvtojson library to your own project.

Import csvtojson to your package.json or install through npm:

>npm install csvtojson

~~The core of the tool is Converter class. It is based on node-csv library (version 0.3.6). Therefore it has all features of [node-csv](http://www.adaltas.com/projects/node-csv/).~~ To start a parse, simply use following code:

```js
//Converter Class
var fs = require("fs");
var Converter = require("csvtojson").Converter;
var fileStream = fs.createReadStream("./file.csv");
//new converter instance
var param={};
var converter = new Converter(param);

//end_parsed will be emitted once parsing finished
converter.on("end_parsed", function (jsonObj) {
   console.log(jsonObj); //here is your result json object
});

//read from file
fileStream.pipe(converter);
```
# Params
The parameters for Converter constructor are:

* constructResult: true/false. Whether to constrcut final json object in memory which will be populated in "end_parsed" event. Set to false if deal with huge csv data. default: true.
* delimiter: delimiter used for seperating columns. default: ","
* quote: If a column contains delimiter, it is able to use quote character to surround the column content. e.g. "hello, world" wont be split into two columns while parsing. default: " (double quote)
* trim: Indicate if parser trim off spaces surrounding column content. e.g. "  content  " will be trimmed to "content". Default: true
* checkType: This parameter turns on and off weather check field type. default is true. See [Field type](#field-type)
* toArrayString: Stringify the stream output to JSON array. This is useful when pipe output to a file which expects JSON array. default is false and only JSON will be pushed to downstream.
* ignoreEmpty: Ignore the empty value in CSV columns. If a column value is not giving, set this to true to skip them. Defalut: false.

# Parser
CSVTOJSON allows adding customised parsers which concentrating on what to parse and how to parse.
It is the main power of the tool that developer only needs to concentrate on how to deal with the data and other concerns like streaming, memory, web, cli etc are done automatically.

How to add a customised parser:

```js
//Parser Manager
var parserMgr=require("csvtojson").parserMgr;

parserMgr.addParser("myParserName",/^\*parserRegExp\*/,function (params){
   var columnTitle=params.head; //params.head be like: *parserRegExp*ColumnName;
   var fieldName=columnTitle.replace(this.regExp, ""); //this.regExp is the regular expression above.
   params.resultRow[fieldName]="Hello my parser"+params.item;
});
```

parserMgr's addParser function take three parameters:

1. parser name: the name of your parser. It should be unique.

2. Regular Expression: It is used to test if a column of CSV data is using this parser. In the example above any column's first row starting with *parserRegExp* will be using it.

3. Parse function call back: It is where the parse happens. The converter works row by row and therefore the function will be called each time needs to parse a cell in CSV data.

The parameter of Parse function is a JSON object. It contains following fields:

**head**: The column's first row's data. It generally contains field information. e.g. *array*items

**item**: The data inside current cell.  e.g. item1

**itemIndex**: the index of current cell of a row. e.g. 0

**rawRow**: the reference of current row in array format. e.g. ["item1", 23 ,"hello"]

**resultRow**: the reference of result row in JSON format. e.g. {"name":"Joe"}

**rowIndex**: the index of current row in CSV data. start from 1 since 0 is the head. e.g. 1

**resultObject**: the reference of result object in JSON format. It always has a field called csvRows which is in Array format. It changes as parsing going on. e.g.

```json
{
   "csvRows":[
      {
          "itemName":"item1",
          "number":10
      },
      {
         "itemName":"item2",
         "number":4
      }
   ]
}
```

# WebServer
It is able to start the web server through code.

```js
var webServer=require("csvtojson").interfaces.web;

var server=webServer.startWebServer({
   "port":"8801",
   "urlpath":"/parseCSV"
});
```

~~It will return an [expressjs](http://expressjs.com/) Application. You can add your own  web app content there.~~ It will return http.Server object.

# Events

Following events are used for Converter class:

* end_parsed: It is emitted when parsing finished. the callback function will contain the JSON object if constructResult is set to true.
* record_parsed: it is emitted each time a row has been parsed. The callback function has following parameters: result row JSON object reference, Original row array object reference, row index of current row in csv (header row does not count, first row content will start from 0)

To subscribe the event:

```js
//Converter Class
var Converter=require("csvtojson").Converter;

//end_parsed will be emitted once parsing finished
csvConverter.on("end_parsed",function(jsonObj){
    console.log(jsonObj); //here is your result json object
});

//record_parsed will be emitted each time a row has been parsed.
csvConverter.on("record_parsed",function(resultRow,rawRow,rowIndex){
   console.log(resultRow); //here is your result json object
});
```

# Default Parsers
There are default parsers in the library they are

~~**Array**: For columns head start with "\*array\*" e.g. "\*array\*fieldName", this parser will combine cells data with same fieldName to one Array.~~

~~**Nested JSON**: For columns head start with "\*json\*" e.g. "\*json\*my.nested.json.structure", this parser will create nested nested JSON structure: my.nested.json~~

~~**Nested JSON Array**: For columns head start with "\*jsonarray\*" e.g. "\*jsonarray\*my.items", this parser will create structure like my.items[].~~

**JSON**: Any valid JSON structure (array, nested json) are supported. see [Empowered JSON Parser](#empowered-json-parser)

**Omitted column**: For columns head start with "\*omit\*" e.g. "\*omit\*id", the parser will omit the column's data.

#~~Example:~~(This example is deprecated see [Empowered JSON Parser](#empowered-json-parser))

Original data:

    date,*json*employee.name,*json*employee.age,*json*employee.number,*array*address,*array*address,*jsonarray*employee.key,*jsonarray*employee.key,*omit*id
    2012-02-12,Eric,31,51234,Dunno Street,Kilkeny Road,key1,key2,2
    2012-03-06,Ted,28,51289,Cambridge Road,Tormore,key3,key4,4

Output data:

```json
{
   "csvRows": [
      {
         "date": "2012-02-12",
         "employee": {
            "name": "Eric",
            "age": "31",
            "number": "51234",
            "key": [
              "key1",
              "key2"
            ]
          },
          "address": [
            "Dunno Street",
            "Kilkeny Road"
          ]
        },
        {
          "date": "2012-03-06",
          "employee": {
            "name": "Ted",
            "age": "28",
           "number": "51289",
            "key": [
              "key3",
              "key4"
            ]
         },
         "address": [
            "Cambridge Road",
            "Tormore"
         ]
      }
   ]
}
```

# Big CSV File
csvtojson library was designed to accept big csv file converting. To avoid memory consumption, it is recommending to use read stream and write stream.

```js
var Converter=require("csvtojson").Converter;
var csvConverter=new Converter({constructResult:false}); // The parameter false will turn off final result construction. It can avoid huge memory consumption while parsing. The trade off is final result will not be populated to end_parsed event.

var readStream=require("fs").createReadStream("inputData.csv");

var writeStream=require("fs").createWriteStream("outpuData.json");

readStream.pipe(csvConverter).pipe(writeStream);
```

The constructResult:false will tell the constructor not to combine the final result which would drain the memory as progressing. The output is piped directly to writeStream.

# Convert Big CSV File with Command line tool
csvtojson command line tool supports streaming in big csv file and stream out json file.

It is very convenient to process any kind of big csv file. It's proved having no issue to proceed csv files over 3,000,000 lines (over 500MB) with memory usage under 30MB.

Once you have installed [csvtojson](#installation), you could use the tool with command:

```
csvtojson [path to bigcsvdata] > converted.json
```

Or if you prefer streaming data in from another application:

```
cat [path to bigcsvdata] | csvtojson > converted.json
```

They will do the same job.

# Column Array
To convert a csv data to column array,  you have to construct the result in memory. See example below

```js
var columArrData=__dirname+"/data/columnArray";
var rs=fs.createReadStream(columArrData);
var result = {}
var csvConverter=new CSVAdv();
//end_parsed will be emitted once parsing finished
csvConverter.on("end_parsed", function(jsonObj) {
    console.log(result);
    console.log("Finished parsing");
    done();
});

//record_parsed will be emitted each time a row has been parsed.
csvConverter.on("record_parsed", function(resultRow, rawRow, rowIndex) {

    for (var key in resultRow) {
        if (!result[key] || !result[key] instanceof Array) {
            result[key] = [];
        }
        result[key][rowIndex] = resultRow[key];
    }

});
rs.pipe(csvConverter);
```

Here is an example:

    TIMESTAMP,UPDATE,UID,BYTES SENT,BYTES RCVED
    1395426422,n,10028,1213,5461
    1395426422,n,10013,9954,13560
    1395426422,n,10109,221391500,141836
    1395426422,n,10007,53448,308549
    1395426422,n,10022,15506,72125

It will be converted to:

```json
{
  "TIMESTAMP": ["1395426422", "1395426422", "1395426422", "1395426422", "1395426422"],
  "UPDATE": ["n", "n", "n", "n", "n"],
  "UID": ["10028", "10013", "10109", "10007", "10022"],
  "BYTES SENT": ["1213", "9954", "221391500", "53448", "15506"],
  "BYTES RCVED": ["5461", "13560", "141836", "308549", "72125"]
}
```

# Parse String
To parse a string, simply call fromString(csvString,callback) method. The callback parameter is optional.

For example:

```js
var testData=__dirname+"/data/testData";
var data=fs.readFileSync(testData).toString();
var csvConverter=new CSVConverter();

//end_parsed will be emitted once parsing finished
csvConverter.on("end_parsed", function(jsonObj) {
    //final result poped here as normal.
});
csvConverter.fromString(data,function(err,jsonObj){
    if (err){
      //err handle
    }
    console.log(jsonObj);
});

```

#Empowered JSON Parser
Since version 0.3.8, csvtojson now can replicate any complex JSON structure.
As we know, JSON object represents a graph while CSV is only 2-dimension data structure (table).
To make JSON and CSV containing same amount information, we need "flatten" some information in JSON.

Here is an example. Original CSV:

```
fieldA.title, fieldA.children[0].name, fieldA.children[0].id,fieldA.children[1].name, fieldA.children[1].employee[].name,fieldA.children[1].employee[].name, fieldA.address[],fieldA.address[], description
Food Factory, Oscar, 0023, Tikka, Tim, Joe, 3 Lame Road, Grantstown, A fresh new food factory
Kindom Garden, Ceil, 54, Pillow, Amst, Tom, 24 Shaker Street, HelloTown, Awesome castle

```
The data above contains nested JSON including nested array of JSON objects and plain texts.

Using csvtojson to convert, the result would be like:

```json
[{
    "fieldA": {
        "title": "Food Factory",
        "children": [{
            "name": "Oscar",
            "id": "0023"
        }, {
            "name": "Tikka",
            "employee": [{
                "name": "Tim"
            }, {
                "name": "Joe"
            }]
        }],
        "address": ["3 Lame Road", "Grantstown"]
    },
    "description": "A fresh new food factory"
}, {
    "fieldA": {
        "title": "Kindom Garden",
        "children": [{
            "name": "Ceil",
            "id": "54"
        }, {
            "name": "Pillow",
            "employee": [{
                "name": "Amst"
            }, {
                "name": "Tom"
            }]
        }],
        "address": ["24 Shaker Street", "HelloTown"]
    },
    "description": "Awesome castle"
}]
```

Here is the rule for CSV data headers:

* Use dot(.) to represent nested JSON. e.g. field1.field2.field3 will be converted to {field1:{field2:{field3:< value >}}}
* Use square brackets([]) to represent an Array. e.g. field1.field2[< index >] will be converted to {field1:{field2:[< values >]}}. Different column with same header name will be added to same array.
* Array could contain nested JSON object. e.g. field1.field2[< index >].name will be converted to {field1:{field2:[{name:< value >}]}}
* The index could be omitted in some situation. However it causes information lost. Therefore Index should **NOT** be omitted if array contains JSON objects with more than 1 field (See example above fieldA.children[1].employee field, it is still ok if child JSON contains only 1 field).

Since 0.3.8, JSON parser is the default parser. It does not need to add "\*json\*" to column titles. Theoretically, the JSON parser now should have functionality of "Array" parser, "JSONArray" parser, and old "JSON" parser.

This mainly purposes on the next few versions where csvtojson could convert a JSON object back to CSV format without losing information.
It can be used to process JSON data exported from no-sql database like MongoDB.

#Field Type

From version 0.3.14, type of fields are supported by csvtojson.
The parameter checkType is used to whether to check and convert the field type.
See [here](#params) for the parameter usage.

Thank all who have contributed to ticket [#20](https://github.com/Keyang/node-csvtojson/issues/20).

##Implict Type

When checkType is turned on, parser will try to convert value to its implicit type if it is not explicitly specified.

For example, csv data:
```csv
name, age, married, msg
Tom, 12, false, {"hello":"world","total":23}

```
Will be converted into:
```json
{
  "name":"Tom",
  "age":12,
  "married":false,
  "msg":{
    "hello":"world",
    "total":"23"
  }
}
```
If checkType is turned **OFF**, it will be converted to:
```json
{
  "name":"Tom",
  "age":"12",
  "married":"false",
  "msg":"{\"hello\":\"world\",\"total\":23}"
}
```

##Explicit Type
CSV header column can explicitly define the type of the field.
Simply add type before column name with a hash symbol (#).

###Supported types:
* string
* number
* date

### Define Type
To define the field type, see following example
```csv
string#appNumber, string#finished, date#startDate
201401010002, true, 2014-01-01
```
The data will be converted to:
```json
{
  "appNumber":"201401010002",
  "finished":"true",
  "startDate":Wed Jan 01 2014 00:00:00 GMT+0000 (GMT)
}
```
### Invalid Value
If parser meets invalid value for a type while parsing a value, it will fallback to use string value.

For example:
```csv
number#order, date#shipDate
A00001, Unknown
```

It will be converted to:
```json
{
  "order":"A00001",
  "shipDate":"Unknown"
}
```

#Change Log

##0.3.21
* Refactored Command Line Tool.
* Added ignoreEmpty parameter.

##0.3.18
* Fixed double qoute parse as per CSV standard.

##0.3.14
* Added field type support
* Fixed some minor bugs

##0.3.8
* Empowered built-in JSON parser.
* Change: Use JSON parser as default parser.
* Added parameter trim in constructor. default: true. trim will trim content spaces.

##0.3.5
* Added fromString method to support direct string input

##0.3.4
* Added more parameters to command line tool.

##0.3.2
* Added quote in parameter to support quoted column content containing delimiters
* Changed row index starting from 0 instead of 1 when populated from record_parsed event

##0.3
* Removed all dependencies
* Deprecated applyWebServer
* Added construct parameter for Converter Class
* Converter Class now works as a proper stream object
