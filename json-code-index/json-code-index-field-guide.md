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
		"sdmx-proto-json":"2013-02-05",
		"id":"2b40b1fa-3f70-4837-8861-69ee15049677",
		"prepared":"2013-02-05T09:45:58",
		"sender":{
		  "id":"MT",
		  "name":"Metadata Technology"
		},
		"name":"BIS Effective exchange rates",
		"dimensions":[
		  # dimension objects #
		],
    	"attributes":[
      	  # attribute objects #
    	],
		"data":{
		  # data objects #   
		},
		"attribute-values":{
      	  # attribute value objects #
    	},
    	"errors": null
	}

### sdmx-proto-json

*String*. A string that specifies the version of the sdmx-proto-json response. A list of valid versions will
be released later on. Example:

    "sdmx-proto-json": "2012-11-15"

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

Example:

    "sender": {
      "id": "SDMX"
    }
	
### errors

*Array* *nullable*. RESTful web services indicates errors using the HTTP status
codes. In addition, whenever appropriate, the error can also be returned using
the error fields. Error is an array of error messages. If there are no errors
then error is null.

Example:

    "errors": [
      "Invalid number of dimensions in parameter key"
    ]

## <a name="Dimensions"></a>Dimensions

```Currently subset of the SMDX-ML functionality```

*Array* *nullable*. Dimensions provides the structural metadata necessary to interpret the data contained in the message.
It tells you which are the dimensions used in the message. It is an array of [Dimension](#Dimension) objects.

Example:

	"dimensions":[
	  # dimension object #
    ]
	
### <a name="Dimension"></a>Dimension

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

#### <a name="dimension_values"></a>Dimension code

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
	
## <a name="Attributes"></a>Attributes

*Array* *nullable*. Contains [attributes](Attribute) in an array. Example:

    "attributes": [
	  # attribute object #
	]
	
### <a name="Attribute"></a>Attribute object
	
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

#### role

*String* *nullable*. Defines the component role(s). For normal components the value
is null. Components can play various roles, such as, for example:

- **time**. Time dimension is a special dimension which designates the period in
time in which the data identified by the full series key applies.
- **measure**. Measure dimension is a special type of dimension which defines
multiple measures.

Example:

    "role": "time"

#### <a name="attachment"></a>attachment

*Array* *nullable*. An array of booleans representing the relationship between an attribute type and the dimensions. An attribute can be applied to multiple dimensions and this array stores the map of what dimensions the given attribute type applies to. If there is a true at a given index in this array then the attribute applies to the dimension with that index.

Example:

	"attachment": [
	  true,
	  true,
	  false
	]

#### codes

Array of [codes](#attribute_values) for the attributes. Example:

    "codes": [
	  # attribute code #
    ]
	
#### <a name="attribute_values"></a>Attribute code

*Object* *nullable*. A particular value for an attribute in a message. Example:

	{
	  "id": "EUR",
	  "name": "Euro",
	  "default": false
	}

##### id

*String*. Unique identifier for a code. Example:

    "id": "EUR"

##### name

*String*. Human-readable name for a code. Example:

    "name": "Euro"

##### description

*String* *nullable*. Description provides a plain text, human-readable
description of the code. Example:

    "description": "Euro Currency"

#### default

*String* or *Number* *nullable*. Defines a default value for the attribute. If
no value is provided then this value applies. Example:

    "default": "A"

## <a name="Data"></a>Data

*Object* *nullable*. Object with properties with the dimension values for each observation. The property names are colon-delimited indices of the values of the dimensions with the delimited number's index corresponding to the observation. The "0:1:0"=2 property represents an obesrvation where the first dimension's value is that with index 0, the second's has index 1 (and is therefore the second value for that dimension) and the third's has index 0, while the observation's value is 2.

Example:

	"data":{
      "0:0:0:0:0":98.6,
      "0:0:0:0:1":97.16
	}
	
## <a name="AttributeValues"></a>Attribute Values

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
