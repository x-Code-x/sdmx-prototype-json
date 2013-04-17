# Draft Field Guide to SDMX-PROTO-JSON Objects

**json-code-index format** **v0.1.0**

Use this guide to better understand SDMX-PROTO-JSON objects.

- [Introduction](#Introduction)
- [Message](#Message)
- [Dimensions](#Dimensions)
- [Attributes](#Attributes)
- [Data](#Data)
- [Attribute Values](#AttributeValues)

New fields may be introduced in later versions of the field guide. Therefore
consuming applications should tolerate the addition of new fields with ease.

The ordering of fields in objects is undefined. The fields may appear in any order
and consuming applications should not rely on any specific ordering. It is safe to consider a nulled field and the absence of a field as the same thing.

Not all fields appear in all contexts. For example response with error messages
may not contain fields for data, dimensions and attributes.

----

## <a name="Introduction"></a>Introduction
Let's first start with a brief introduction of the SDMX information model.

In order to make sense of some statistical data, we need to know the concepts
associated with them. For example, on its own the figure 1.2953 is pretty meaningless,
but if we know that this is an exchange rate for the US dollar against the euro on
23 November 2006, it starts making more sense.

There are two types of concepts: dimensions and attributes. Dimensions, when combined,
allow to uniquely identify statistical data. Attributes on the other hand do not help
identifying statistical data, but they add useful information (like the unit of measure
or the number of decimals). Dimensions and attributes are known as "components".

The measurement of some phenomenon (e.g. the figure 1.2953 mentioned above) is known as an
"observation" in SDMX. Observations are grouped together into a "data set". However, there
can also be an intermediate grouping. For example, all exchange rates for the US dollar
against the euro can be measured on a daily basis and these measures can then be
grouped together, in a so-called "time series". Similarly, you can group a collection of
observations made at the same point in time, in a "cross-section" (for example,
the values of the US dollar, the Japanese yen and the Swiss franc against the euro at a
particular date). Of course, these intermediate groupings are entirely optional and you
may simply decide to have a flat list of observations in your data set.

The SDMX information model is much richer than this limited introduction,
however the above should be sufficient to understand the JSON format proposed here. For
additional information, please refer to the [SDMX documentation](http://sdmx.org/?page_id=10).

----
## <a name="Message"></a>Message

Message is the response you get back from the SDMX RESTful API. Message is the top
level object and it contains the data as well as the metadata needed to interpret those data.
Example:

    {
      "sdmx-proto-json": "2012-11-15",
      "header": {
        "name": "BIS Effective Exchange Rates",
        "id": "b1804c51-1ee3-45a9-bb75-795cd4e06489",
        "prepared": "2012-05-04T03:30:00"
      },
      "structure": {
        # structure objects #
      },
      "dataSets": [
        # data set objects #
      ],
      "errors": null
    }

### sdmx-proto-json

*String*. A string that specifies the version of the sdmx-proto-json response. A list of valid versions will
be released later on. Example:

    "sdmx-proto-json": "2012-11-15"

### header

*Object* *nullable*. *Header* contains basic information about the message. Example:

    "header": {
      "name": "BIS Effective Exchange Rates",
      "id": "b1804c51-1ee3-45a9-bb75-795cd4e06489",
      "prepared": "2012-05-04T03:30:00"
    }

### structure

*Object* *nullable*. *Structure* contains the information needed to interpret the data available in the message. Example:

    "structure": {
        "id": "ECB_EXR_WEB",
        "href": "http://sdw-ws.ecb.europa.eu/dataflow/ECB/EXR/1.0",
        "packaging": {
            # packaging object #
        }
    }

### dataSets

*Array* *nullable*. *DataSets* field is an array of *[DataSet](#DataSet)* objects. That's where the data (i.e.: the observations)
will be. In typical cases, the file will
contain only one data set. However, in some cases, such as when retrieving, from an SDMX 2.1 web service, what has
changed in the data source since in particular point in time, the web service might return more than one data set.
Example:

    "dataSets": [
      {
        "action": "Informational",
        "extracted": "2012-05-04T03:30:00",
        "data": {
          # data object #
        },
		"attribute-values": {
		  # attribute object #
		}
      }
    ]

### errors

*Array* *nullable*. RESTful web services indicates errors using the HTTP status
codes. In addition, whenever appropriate, the error can also be returned using
the error fields. Error is an array of error messages. If there are no errors
then error is null. Example:

    "errors": [
      "Invalid number of dimensions in parameter key"
    ]

----


## <a name="Header"></a>header

Header contains basic information about the message. Example:

    "header": {
      "id": "b1804c51-1ee3-45a9-bb75-795cd4e06489",
      "prepared": "2013-01-03T12:54:12",
      "name": "BIS Effective Exchange Rates",
      "sender: {
        "id": "SDMX"
      }
    }

### id

*String*. Unique string that identifies the message for further references.
Example:

    "id": "TEC00034"

### name

*String* *nullable*. Brief summary of the message contents. Example:

    "name": "Short-term interest rates: Day-to-day money rates"

### test

*Boolean* *nullable*. Test indicates whether the message is for test purposes or not. False for normal messages. Example:

    "test": false

### prepared

*String*. Prepared is the date the message was prepared. String representation
of Date formatted according to the ISO-8601 standard. Example:

    "prepared": "2012-05-04T03:30:00Z"

### sender

```Currently subset of the SMDX-ML functionality```

*Object*. Sender is information about the party that is transmitting the message.
Sender contains the following fields:

* id - *String*. The id attribute holds the identification of the party.
* name - *String* *nullable*. Name is a human-readable name of the party.
* contact - *Array* *nullable*. A collection of contact details.

Example:

    "sender": {
      "id": "ECB",
      "name": "European Central Bank"
      "contact": [
        # contact details #
      ]
    }
    
#### contact

*Array* *nullable*. A collection of contact details.

Each object in the collection may contain the following field:
* name - *String*. The contact's name.
* department - *String* *nullable*. The organisational structure within which the contact person works.
* role - *String* *nullable*. The responsibility of the contact person.
* telephone - *String* *nullable*. The telephone number for the contact person. 
* fax - *String* *nullable*. The fax number for the contact person.
* x400 - *String* *nullable*. The X.400 address for the contact person.
* uri - *String* *nullable*. URI holds an information URL for the contact person.
* email - *String* *nullable*. The email address for the contact person.
 
Example:

    "contact": [
        {
            "name": "Statistics hotline",
            "email": "statistics@xyz.org"
        }
    ]

### receiver

```Currently subset of the SMDX-ML functionality```

*Object* *nullable*. Receiver is information about the party that is receiving the message.
Receiver contains the following fields:

* id - *String*. The id attribute holds the identification of the party.
* name - *String* *nullable*. Name is a human-readable name of the party.
* contact - *Array* *nullable*. A collection of contact details.

Example:

    "receiver": {
      "id": "SDMX"
    }

### extracted

*String* *nullable*. Extracted is a timestamp indicating when the data have been extracted from the data source.
Example:

    "extracted": "2012-05-04T03:30:00"

### embargoDate

*String* *nullable*. EmbargoDate holds an ISO-8601 time period before which the data included in this message is not available. Example:

    "embargoDate": "2012-05-04"

### source

*Array* *nullable*. Source provides human-readable information about the source of the data. Example:

    "source": [
      "European Central Bank and European Commission"
    ]

----

## <a name="Structure"></a>structure

```Currently subset of the SMDX-ML functionality```

*Object* *nullable*. Structure provides the structural metadata necessary to interpret the data contained in the message.
It tells you which are the components (dimensions and attributes) used in the message and also describes to which
level in the hierarchy (data set, series, observations) these components are attached.

Example:

    "structure": {
        "id": "ECB_EXR_WEB",
        "href": "http://sdw-ws.ecb.europa.eu/dataflow/ECB/EXR/1.0",
		"dimensions": [
		  # dimension objects #
		],
		"attributes": [
      	  # attribute objects #
    	],
	}

### id

*String* *nullable*. An identifier for the structure. Example:

    "id": "ECB_EXR_WEB"

### href

*String* *nullable*. A link to an SDMX 2.1 web service resource where additional information regarding the structure is
available. Example:

    "href": "http://sdw-ws.ecb.europa.eu/dataflow/ECB/EXR/1.0"

### ref

*Object* *nullable*. Provides the 4 elements necessary to uniquely identify structural metadata that offers additional
information about the data contained in the message. Example:

    "ref": {
        "type": "dataStructure",
        "agencyID": "ECB",
        "id": "EXR",
        "version": 1.0
    }

The ref element may contain the following elements:

#### type

*String*. The type of structural metadata. The following values are allowed: dataStructure, dataflow, provisionAgreement.
For additional information about these, please refer to the [SDMX documentation](http://sdmx.org/?page_id=10). Example:

    "type": "dataStructure"

#### agencyID

*String*. The ID of the agency maintaining the structural metadata. Example:

    "agencyID": "ECB"

#### id

*String*. The ID of the structural metadata. Example:

    "id": "EXR"

#### version

*String* *nullable*. The version of the structural metadata. If not supplied, defaults to 1.0. Example:

    "version": 1.0

### <a name="Dimensions"></a>dimensions

```Currently subset of the SMDX-ML functionality```

*Array* *nullable*. Dimensions provides the structural metadata necessary to interpret the data contained in the message.
It tells you which are the dimensions used in the message. It is an array of [Dimension](#Dimension) objects.

Example:

	"dimensions":[
	  # dimension object #
    ]
	
#### <a name="Dimension"></a>dimension

A dimension contains basic information about the component
(such as its name and id) as well as the list of values used in the message for this particular component. Example:

      {
         "id":"EER_BASKET",
         "name":"Basket",
         "role":null,
         "codes":[
		   # value object #
         ]
	  }

Each of the dimensions may contain the following fields

#### id

*String*. Identifier for the component.
Example:

    "id": "FREQ"

#### name

*String*. Name provides a human-readable name for the object.
Example:

    "name": "Frequency"

#### description

*String* *nullable*. Provides a description for the object. Example:

    "description": "The time interval at which observations occur over a given time period."

#### role

*String* *nullable*. Defines the component role(s). For normal dimensions the value
is null. Dimensions can play various roles, such as, for example:

- **time**. Time dimension is a special dimension which designates the period in
time in which the data identified by the full series key applies.
- **measure**. Measure dimension is a special type of dimension which defines
multiple measures.

Example:

    "role": "time"

#### codes

Array of [codes](#dimension_values) for the dimensions. Example:

    "codes": [
      {
        "id": "M",
        "name": "Monthly",
        "orderBy": 0
      }
    ]

#### <a name="dimension_values"></a>dimension code

*Object* *nullable*. A particular value for a dimension in a message. Example:

    {
      "start":"2011-11-01T00:00:00",
      "end":"2011-11-30T23:59:59",
      "id":"2011-11",
      "name":"2011-11",
	  "selected":true
    }

##### id

*String*. Unique identifier for a value. Example:

    "id": "A"

##### name

*String*. Human-readable name for a value. Example:

    "name": "Missing value; data cannot exist"

##### description

*String* *nullable*. Description provides a plain text, human-readable
description of the value. Example:

    "description": "Provisional value"

##### orderBy

*Number* *nullable*. Default display order for the value. Example:

    "orderBy": 64

##### parent

*String* *nullable*. Parent value (for code hierarchies). If parent is null then
the value does not belong to any hierarchy. Hierarchy root values have special
value "ROOT" for the parent. There may be multiple roots. Each value has only one
parent. Example:

    "parent": "U2"

##### start

*String* *nullable*. Start date for the period in a time dimension.
This field is useful only when the value represents a period for a time dimension
(dimension type is 'time'). Value is a date in ISO format for the beginning of the
period. Example:

    "start": "2007-02-01T00:00:00.000Z"

##### end

*String* *nullable*. End date for period in a time dimension.
This field is useful only when the value represents a period for a time dimension
(dimension type is 'time'). Value is a date in ISO format for the end of the
period. Example:

    "end": "2007-10-31T23:59:59.000Z"

##### selected

*Boolean* *nullable*. Dimension codes include the property “selected” indicating if a corresponding filter setting was made in the URL query. All structural objects have consistently the properties “id” and “name”. No placeholder for missing values is needed thus sparse datasets are represented efficiently. Example:

	"selected":true
	
### <a name="Attributes"></a>attributes

*Array* *nullable*. Contains [attributes](Attribute) in an array. Example:

    "attributes": [
	  # attribute object #
	]
	
#### <a name="Attribute"></a>attribute object
	
*Object* *nullable*. A particular value for an attribute in a message. Example:

	{
	  "id": "UNIT",
	  "name": "Unit",
	  "role": "unit",
	  "attachment": [
		true,
		true,
		true,
		true,
		true,
		true,
		false
	  ],
	  "codes": [
	    # attribute codes #
	  ]
	}

##### id

*String*. Unique identifier for an attribute. Example:

    "id": "UNIT"

##### name

*String*. Human-readable name for an attribute. Example:

    "name": "Unit"

##### role

*String* *nullable*. Defines the component role(s). For normal components the value
is null. Components can play various roles, such as, for example:

- **time**. Time dimension is a special dimension which designates the period in
time in which the data identified by the full series key applies.
- **measure**. Measure dimension is a special type of dimension which defines
multiple measures.

Example:

    "role": "time"

##### <a name="attachment"></a>attachment

*Array* *nullable*. An array of booleans representing the relationship between an attribute type and the dimensions. An attribute can be applied to multiple dimensions and this array stores the map of what dimensions the given attribute type applies to. If there is a true at a given index in this array then the attribute applies to the dimension with that index.

Example:

	"attachment": [
	  true,
	  true,
	  false
	]

##### codes

Array of [codes](#attribute_values) for the attributes. Example:

    "codes": [
	  # attribute code #
    ]
	
###### <a name="attribute_values"></a>attribute code

*Object* *nullable*. A particular value for an attribute in a message. Example:

	{
	  "id": "EUR",
	  "name": "Euro",
	  "default": false
	}

###### id

*String*. Unique identifier for a code. Example:

    "id": "EUR"

###### name

*String*. Human-readable name for a code. Example:

    "name": "Euro"

###### description

*String* *nullable*. Description provides a plain text, human-readable
description of the code. Example:

    "description": "Euro Currency"

###### default

*String* or *Number* *nullable*. Defines a default value for the attribute. If
no value is provided then this value applies. Example:

    "default": "A"

###### geocode

*Object* *nullable*. Represents the geographic location of this code (country,
reference area etc.). The inner coordinates array is formatted as [geoJSON]
(http://www.geojson.org) (longitude first, then latitude). Example:

    "geocode": {
      "type": "Point",
      "coordinates": [
        62.4302,
        24.7271
      ]
    }

----

## <a name="DataSets"></a>DataSets

DataSets object is an array of data set objects. Example:

    "dataSets": [
      {
        "action": "Informational",
        "extracted": "2012-05-04T03:30:00",
        "data": {
          # data object #
        },
		"attribute-values": {
		  # attribute object #
		}
      }
    ]

There are between 2 and 3 levels in a data set object, depending on the way the data in the message is organized.

A data set may contain a flat list of observations. In this scenario, we have 2 levels in the data part of the message:
the data set level and the observation level.

A data set may also organize observations in logical groups called series. These groups can represent time series or
cross-sections. In this scenario, we have 3 levels in the data part of the message: the data set level, the
series level and the observation level.

Dimensions and attributes may be attached to any of these 3 levels. The way this is done in a particular message is
documented in the [packaging element](#packaging).

Observations will be found directly under a data set object, in case the data set is a flat list of observations. In
case the data set represents time series or cross sections, the observations will be found under the series elements.

### id

*String* *nullable*. Id provides an identifier for the data set. Example:

    "id": "ECB_EXR_2011-06-17"

### name

*String* *nullable*. A human-friendly name for the dataset. Example:

    "name": "Bilateral exchange rates"
    
### description

*String* *nullable*. Description provides a plain text, human-readable
description of the dataset. Example:

    "description": "The nominal effective exchange rate (EER) index is a summary measure of the external value of 
    a currency vis-á-vis the currencies of the most trading partners, while the real EER - obtained by deflating 
    the nominal rate with appropriate price or cost indices - is the most commonly used indicator of international 
    price and cost competitiveness. Daily spot exchange rates provided by the Front Office Division."

### action

*String* *nullable*. Action provides a list of actions, describing the intention of the data transmission
from the sender's side. ```Default value is Informational```

* Append - this is an incremental update for an existing data set or the provision of new data or documentation
(attribute values) formerly absent. If any of the supplied data or metadata are already present, it will not replace
these data.

* Replace - data are to be replaced, and may also include additional data to be appended.

* Delete - data are to be deleted.

* Informational - data are being exchanged for informational purposes only, and not meant to update a system.

Example:

    "action": "Informational"

### provider

*Object* *nullable*. Provider is information about the party that provides the data contained in the data set.
Provider contains the following fields:

* id - *String*. The id attribute holds the identification of the party.
* name - *String* *nullable*. Name is a human-readable name of the party.

Example:

    "provider": {
      "id": "EUROSTAT"
    }

### reportingBegin

*String* *nullable*. ReportingBegin provides the start of the time period covered by the message. Example:

    "reportingBegin": "2012-05-04"

### reportingEnd

*String* *nullable*. ReportingEnd provides the end of the time period covered by the message. Example:

    "reportingEnd": "2012-06-01"

### validFrom

*String* *nullable*. The validFrom indicates the inclusive start time indicating the validity of the information in the data.

    "validFrom": "2012-01-01T10:00:00Z"

### validTo

*String* *nullable*. The validTo indicates the inclusive end time indicating the validity of the information in the data.

    "validTo": "2013-01-01T10:00:00Z"

### publicationYear

*String* *nullable*. The publicationYear holds the ISO 8601 four-digit year.

    "publicationYear": "2005"

### publicationPeriod

*String* *nullable*. The publicationPeriod specifies the period of publication of the data in terms of whatever
provisioning agreements might be in force (i.e., "2005-Q1" if that is the time of publication for a data set
published on a quarterly basis).

    "publicationPeriod": "2005-Q1"
	
	
### <a name="Data"></a>data

*Object* *nullable*. Object with properties with the dimension values for each observation. The property names are colon-delimited indices of the values of the dimensions with the delimited number's index corresponding to the observation. The "0:1:0"=2 property represents an obesrvation where the first dimension's value is that with index 0, the second's has index 1 (and is therefore the second value for that dimension) and the third's has index 0, while the observation's value is 2.

Example:

	"data":{
      "0:0:0:0:0":98.6,
      "0:0:0:0:1":97.16
	}
	
### <a name="AttributeValues"></a>attribute-values

*Object* *nullable*. Object with properties with the dimension- and attribute values for each observation similarly to the [data](#Data) message property.

Each property name consists of two parts: the first part describes the dimensions the attribute is attache to, the part after the ';' character describes the attribute and the value is either a code index in the attribute type representing that attribute's value or the attribute value itself. So the line '"0:0:0;1":0' means that the attribute is attached to all dimensions (also see [attachment](#attachment) in [attribute types](#Attribute)), it is the second attribute type (index 1 after the semicolon) and has a code index of 0, representing the first value for that attribute type.

If an attribute is not attached to a dimension that dimension's index is left empty such as in '":0:;3":5'. Here the attribute type is only attached to the second dimension, it is the fourth attribute type and has the attribute value with code index 5, that is the sixth value.

Example:
	
	"attribute-values": {
	  "0:0:0:0:0:0:;1": 0,
	  "0:0:0:0:0:0:14;0": 1,
	  "0:0:0:0:0:1:;1": 0,
	  "0:0:0:0:0:0:65;0": 2
	}
