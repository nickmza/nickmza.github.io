---
title: Adventures in Nifi
tags:
- integration
- Nifi
- OSS
category: ETL
ogimage: NifiLogo.png
excerpt:
    We are currently using a proprietary ETL tool. Whilst it is very capable, skills and support in the market are scarce. I wanted to see what Open Source alternatives are available and see if they could be used to simplify some of our ETL flows.
---

We are currently using a proprietary ETL tool. Whilst it is very capable, skills and support in the market are scarce. I wanted to see what Open Source alternatives are available and see if they could be used to simplify some of our ETL flows. A cursory search revealed a few names and after some more reading I decided to give [Apache Nifi](https://nifi.apache.org/) a go.

# What is Nifi?

Nifi allows you to build data pipelines using a graphical environment. The pipelines are created by connecting a series of Processors. Each Processor is responsible for performing some action on the data. This could be anything from reading from a Log to sorting, filtering or outputting files. 

Processors expose a set of properties that allows you to control their operation. Where you need to be more specific an Expression Language is used. 

That's the basics. Behind the scenes Nifi takes care of security, data provenance, scaling, persistence, retry semantics etc.

Here's a simple example:

Assume we are reading from a Log and we want to Route 'Info' and 'Debug' messages to Nifi's own logs but write everything else to a separate file. 

This is what it looks like:

<img src="image-5.png"/>

We add a *RouteOnAttribute* Processor and configure it with the following rules:

<img src="image-6.png"/>

This creates 3 outputs from the Processor: Debug, Info and Unmatched. 

We connect the Debug and Into outputs to the *LogMessage* Processor and the Unmatched output to the *PutFile* Processor.

Job done.

# What do we need?
As with any tool it's important to start by understanding what capabilities one requires. In our case we need to:
- Parse CSV and Fixed-width files.
- Convert data to XML and JSON.
- Handle various data-encodings.
- Filter incoming data.
- Support a variety of input and output adapters.

# POC scenario
To put Nifi through its paces I selected an example from our Paypal Reconciliation flow. We receive a summary of Paypal transactions in CSV format and need to convert it into a delimited Swift format. In the process we need to apply some rules to extract only certain messages and apply some specific format rules.

# Setup
You can get Nifi running locally in Docker using the following:

```
docker run --name nifi \
  -p 8443:8443 \
  -d \
  -e SINGLE_USER_CREDENTIALS_USERNAME=admin \
  -e SINGLE_USER_CREDENTIALS_PASSWORD=password \
  apache/nifi:latest
```

# Parse the CSV

This is what the incoming data looks like:

```
"RH",2024/08/21 02:44:19 -0700,"A","**********",009,
"FH",01
"SH",2024/08/20 00:00:00 -0700,2024/08/20 23:59:59 -0700,"","**********",",""
"CH","Transaction ID","Invoice ID","PayPal Reference ID","PayPal Reference ID Type","Transaction Event Code","Transaction Initiation Date","Transaction Completion Date","Transaction Debit or Credit","Gross Transaction Amount","Gross Transaction Currency","Fee Debit or Credit","Fee Amount","Fee Currency","Custom Field","Consumer ID","Payment Tracking ID","Store ID","Bank Reference ID","Credit Transactional Fee","Credit Promotional Fee","Credit Term"
"SB","","**********",","","**********",","","","T0000",2024/08/20 00:01:17 -0700,2024/08/20 00:01:17 -0700,"DR",3000,"USD","",,"","","**********",","","**********",","","","",,,
"SB", ... more lines.
```

I started by using the *GenerateFlowFile* Processor to inject my test data. This Processor generates a test file every minute and saves me having to copy in my test file every time. 

The first step is to split out the header. The approach I took was to split the input file into lines using the *SplitText* Processor and then select the lines I need using the *RouteOnAttribute* Processor. I used the query '${fragment.index:gt(4)}' to exclude first 4 lines of the file. Finally I used the *MergeContent* Processor to reconstitute the file. 

The flow now looks like this:

<img src="image.png"/>

The data would now look like this:

```
"SB","","**********",","","**********",","","","T0000",2024/08/20 00:01:17 -0700,2024/08/20 00:01:17 -0700,"DR",3000,"USD","",,"","","**********",","","**********",","","","",,,
"SB", ... more lines.
```

The next step is to parse the CSV data so that we can format some of the values. To do this I had to create an instance of the *CSVReader* service and configure it with a JSON schema corresponding to the format of the data. 

<img src="image-1.png"/>

> For the sake of this POC I've set the schema directly in the component itself but Nifi does support a central registry for your schemas.

I then updated the record values using the *UpdateRecord* Processor.

<img src="image-2.png"/>

What's happening here is that Nifi creates a Property with the value set from the Value field. Where the Property already exists the value is updated. if the property does not exist a new Property is created with the corresponding value. For example, the 'GrossTransactionAmount' property is defined by the schema provided. Here we are asking Nifi to add 2 decimal places. On the other hand 'CARDSSCV' is a field that will eventually be required by the output format and needs a fixed value - so we are just setting it here. 

The next step is to filter the data so that we only retain records with a specific Transaction Code. To do this we used the *QueryRecord* Processor.  This Processor allows us to treat the incoming data as a database table and filter it with the following query:

```
SELECT * FROM FLOWFILE WHERE TransactionEventCode = 'T9701'
```

Next step is to convert the incoming data format described by the CSVReader into the SWIFT delimited format. To do this I create a CSVRecordSetWriter Service and provide the JSON Schema for the required output. 

<img src="image-3.png"/>

The CSVRecordSetWriter is then passed to the *ConvertRecord* Processor to affect the conversion. The reason (I assume) for this separation is that it is not uncommon for the same input or output schema to be used by multiple components. This approach saves us from having to reconfigure schemas in multiple places.

The conversion itself is done by matching fields in the input message to fields in the output message or setting them explicitly via expressions. This works fine for a simple format but for more complicated scenarios we would need to investigate a custom component or Transformation tool.

Finally I used the *PutRecord* Processor to spit out the file in a folder. Here's the final steps:

<img src="image-4.png"/>

# Initial Thoughts

There are some things I like and somethings I am not sure about. I like the workflow of Nifi - the ability to pause steps and replay data makes for a pleasant developer experience. Vastly superior to the hours we used to spend copying files into folders in the BizTalk days! 

I also like the simplicity of it -  there are only a few concepts to understand  the expression language, Processors and Services. Once you have a basic grasp of these you can get quite a lot done.

What I'm not so sure about is how Nifi scales. Not so much in terms of performance but in scope. Having all your processes in one canvas, even when organised as Process Groups could become a nightmare to manage. You could split out processes to their own instances but I think this could just change the nature of the problem. Does anyone out there have experience running Nifi at scale?

Lastly the effort to parse the CSV format was much more than I expected. In BizTalk for example this would be a single FlatFile Schema attached to a pipeline. The rest of the work could be done in a Map. Here I was able to split out and discard the headers - but in a more complicated scenario where I needed the data in the headers this would have been more complicated. It is entirely possible that there is a much easier way to do this and I missed it because I am a noob. I'll keep looking to see if there's a better way. 
