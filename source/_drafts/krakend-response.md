---
title: Request/Response Shaping with KrakenD
tags:
  - devops
category:
  - Release The KrakenD
excerpt:
    I wrote previously about configuring logging on KrakenD as well as setting up JWT Security. This week I'm looking at how KrakenD can shape Requests and Responses from back-end systems.    
---

I wrote previously about configuring logging on KrakenD as well as setting up JWT Security. This week I'm looking at how KrakenD can shape Requests and Responses from back-end systems.

KrakenD provides multiple facilities for Request/Response Shaping:
- Filtering. Removing specific fields from the response.
- Grouping. Placing a response within a specific tag.
- Mapping. Renaming certain fields.
- Collection Manipulation. Move, delete or append items in arrays.
- Plugins. Completely rewrite requests/reponses using custom code. Maximum performance but requires you to build and deploy the plugin.
- Lua. Same as for Plugins but via the Lua scripting language. You take a (potentially negligible) performance hit but gain the flexibility of scripting.

My requirement is to format the response from our Core Banking into JSON. The challenging part is that our Core Banking response format looks like this:

```
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<T24 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns="http://www.temenos.com/T24/OFSML/130" xsi:schemaLocation="http://www.temenos.com/T24/OFSML/130 ofsml13.xsd">
    <serviceResponse>
        <ofsStandardEnquiry name="E.CUST.FIN.SUMMARY.6.MCB.MU" status="OK">
            <enquiryColumn id="RETURN.CODE" label="RETURN.CODE" type="ALPHANUMERIC"/>

            <!--List of Enquiry Columns>

            <enquiryRecord>
                <column>0</column>

                <!--List of Result Columns>

            </enquiryRecord>
        </ofsStandardEnquiry>
    </serviceResponse>
</T24>
```

For each column in the response table there will be an 'enquiryColumn' record and an associated enquiryRecord/column record. The association is based on the index. So the enquiryColumn at position 0 will describe the column at position 0 in the enquiryRecord. For extra fun if the response contains more than one row the values will be separated by a | delimiter.

The only option that could achieve all of this were either a Plugin or Lua. I opted for the Lua route. Only problem with that was I didn't know Lua... 

# Lua

From the website: [Lua](https://www.lua.org) is a powerful, efficient, lightweight, embeddable scripting language. It supports procedural programming, object-oriented programming, functional programming, data-driven programming, and data description.

I found it really easy to pick up. Some pointers:

**Everything is a table** 
Seriously - everything! Want an array? Create a table with an integer as its first column. 

**Invoke methods with :**
For example:
```
print(t:len())
```
Invokes the len method of t. In this case printing the length of the table t.

**Creating new tables**
Like this:
```
local result = {}
```

**Setting/Getting Values**
Like this:
```
result["ResponseStatus"] = "foo"
print(result["ResponseStatus"])
```
**String Manipulation**
This was a little tricky. I think C# has spoiled me a little. Here's the code to split up a pipe-delimited string.
```
function split(value)

    if(string.sub(value,1,1) == "|") then
        value = " "..value
    end

    local t={}
    for str in string.gmatch(value, "([^|]+)") do
        local k = string.gsub(str, "[ ]+$", "")
        table.insert(t, k)
    end
    return t
end

```
Here's what this does:
- Start by appending an empty string to the start of the string. The .. is the concatenation operator.
- Create a new table, t, to store the results.
- Use a regex passed to the gsub method to find each part of the string.
- Use a regex passed to gsub to trim spaces.
- Add the resultant string to the table with table.insert. 

> It's entirely likely that an experienced Lua practitioner is recoiling in horror at this right now. Feel free to correct my poor Lua in the comments.

# Shaping the Response
Armed with my rudimentary understanding of Lua I was able to put together a working map between the Core Banking XML and Json. I won't reproduce the entire script but here are the highlights:

```
function post_proxy( resp )
 
    local r = resp.load()
    local responseData = r:data()
    local enquiryColumn = responseData:get("T24"):get("serviceResponse"):get("ofsStandardEnquiry"):get("enquiryColumn")
    local dataColumn = responseData:get("T24"):get("serviceResponse"):get("ofsStandardEnquiry"):get("enquiryRecord"):get("column")
    local size = enquiryColumn:len()
     
    local result = {}
    for i=0,size-1 do
      local key = enquiryColumn:get(i):get("-id")
      local value = dataColumn:get(i)
      result[key] = value;
    end

    local response = {}
    response["ResponseStatus"] = {}
    response["ResponseStatus"]["ReturnCode"] = 0

    response["ResponseData"] = {}
    local rd = response["ResponseData"]

    rd["Exception"] = getException(result)

    local items = luaList.new()
    gfs["Ac_Contracts"] = items

    local contractList = split(result["AC.CONTRACT.NO"])

    local contractCount = 1
    for k,v in pairs(contractList) do
        local acrData = getContractRecord(contractCount, result)
        items:set(contractCount-1,acrData)
        contractCount = contractCount + 1
    end

    responseData:set("response", response)
    responseData:del("T24")
end
```
Here are the highlights:
- Access the backend response via resp.load()
- We then get the enquiryColumn and dataColumn
- We can then loop through the enquiryColum and populate the results table. We can then treat the response as key-value pairs rather than 2 disconnected arrays. 
- We build up our response object by pulling values from the results array.
    - Note the use of lualist.new() this is a KrakenD helper to allow you to model a JSON array.
- Finally we remove the original response via the del function.

# Wiring it up
As with all things KrakenD it's just a matter of updating the endpoint definition:
```
"endpoints": [
{
    "endpoint": "/v1/getfinancialsummary",
    "method": "POST",
    "output_encoding": "json",
    "backend": [
    {
        "extra_config": {
        "modifier/lua-backend": {
            "sources": ["formatT24.lua"],
            "live": true,
            "allow_open_libs": true,
            "post": post_proxy(response);"
        }
        },
        "url_pattern": "/getfinancialsummary",
        "encoding": "xml",
        "sd": "static",
        "method": "POST",
        "disable_host_sanitize": true,
        "host": [
            "http://demo.mock.pstmn.io"
        ]
    }
    ]
}    
]
```
Here's how it works:
- sources - defines the list of Lua script files to load. It looks for them in the same directory as your krakend.json file.
- live - tells Krakend to reload the script when it is changed. Super-handy for development.
- allow_open_libs - allows Lua to pull in libraries from other sources.
- post - this is the entry point for our script. In this case we want to invoke our script after the response has been received from the backend. 

# That's all folks...
This worked really well. Even with mock data hosted in Postman we are getting a 400ms response time on a large response. The bulk of this is network latency. I'm not sure what voodoo has been worked on the Lua side to make this work but it's impressive. I'm also secure in the knowledge that if the performance ever becomes an issue I can drop to a dedicated plugin (would just need to learn Go first...)