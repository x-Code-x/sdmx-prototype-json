> Important information: SDMX-JSON format is under development and the specification changes constantly. As a result the documentation may not be consistent at all times and particularly this tutorial may not reflect changes in the latest specification.

# SDMX-JSON Tutorial: Introduction

SDMX-JSON is a format for returning statistical data from a Web Service that implements the SDMX RESTful Web Service Guidelines.  Following is a short tutorial that explains the data model and some of the requirements underlying the format.

SDMX-JSON is trying to solve a fairly complex problem: getting large amounts of statistical data efficiently from the server to the client for visualisation. It has number of features to do this so this tutorial starts with the a simple example and then adds features one by one.

Basic knowledge of JSON is assumed, please see the [JSON web site][1] for more information. Information about the SDMX standards is available on the [SDMX web site][2]. 

## 1. Data Model

We start by looking at the data model behind the SDMX-JSON format. A familiar metaphor for the data model is a simple table with columns, rows and cells. In this tutorial we will be working with the following table of data about population. Following is a JSON representation of the table. 

	{ 
	  "columns": [ "Country", "Type", "Year", "Population", "Status" ],
	  "rows": [
	    [ "Germany", "Total", "2009", 81.9, "Normal value" ],
	    [ "Germany", "Total", "2010", 81.7, "Normal value" ],
	    [ "Mexico", "Total", "2009", 107.6, "Break" ],
	    [ "Mexico", "Total", "2010", 112.3, "Normal value" ]
	  ]
	}

This is not by any means the only option for storing a table in JSON. The main advantage of above format is that it is very compact. Compactness is a key concern because SDMX-JSON format is designed to support interactive visualisation of large sets of data (thousands or even millions of rows). But the data has to be transmitted from the server to the client and someone will be waiting for the data to arrive. We want to reduce the wait as much as possible. 

Couple of things to note:

- Tables have a fixed structure, every row contains the same set of columns. Some rows cannot have more columns than others.
- Cells can be empty (null) so that data does not have to exist in every cell
- The order of the rows is not important but the order of the columns is fixed.
- We have included labels for columns so that the table is understandable. Columns also define the structure of the table.
- Table contains just four data values. Other information in the table is descriptive metadata (data about data). This is not coincidental. We want to include all the available metadata so that the data values are easier to understand and interpret.

## 2. Terminology 

First some terminology. In the SDMX data model the above table is called a _data set_. Rows are called _observations_ and each column has a different type:

- _Dimensions_ are used to identify observations.
- _Measures_ contain the actual data (only one measure is supported per data set).
- _Attributes_ provide additional information about each observation.

Following is the same table with SDMX compliant names. By convention dimensions are followed by measures and attributes in the observation arrays. 

	{ 
	  "dimensions": [ "Country", "Type", "Year" ],
	  "measures": [ "Population" ], 
	  "attributes: [ "Status" ],
	  "observations": [
	    [ "Germany", "Total", "2009", 81.9, "Normal value" ],
	    [ "Germany", "Total", "2010", 81.7, "Normal value" ],
	    [ "Mexico", "Total", "2009", 107.6, "Break" ],
	    [ "Mexico", "Total", "2010", 112.3, "Normal value" ]
	  ]
	}

Summary of changes so far:

- Split columns into three arrays: dimensions, measures and attributes
- rename rows into observations

## 3. Separate Data from Structure

An important feature of the SDMX data model is that the dataset and its structure are separated. This makes it possible to understand the structure of the data set without looking at the actual data.

	{
	  "dataSet" = { 
	    "observations": [
	      [ "Germany", "Total", "2009", 81.9, "Normal value" ],
	      [ "Germany", "Total", "2010", 81.7, "Normal value" ],
	      [ "Mexico", "Total", "2009", 107.6, "Break" ],
	      [ "Mexico", "Total", "2010", 112.3, "Normal value" ]
	    ]
	  },
	  "structure": {
	    "dimensions": [ "Country", "Type", "Year" ],
	    "measures": [ "Population" ], 
	    "attributes": [ "Status" ]
	  }
	}

Summary of changes:

- Add separate fields for the data set and structure
- Move dimensions, measures and attributes to the structure field

## 4. Multiple Data Sets

Now that there is a independent structure field for the data set it is easy to include support for multiple datasets with the same structure. With multiple data sets the format also supports  incremental updates of data. One data set can contain information that was added and another data set information that has been updated. 

We also add more information to each data set e.g. an optional name. Many other fields are supported for data sets.

	{
	  "dataSets" = [ 
	    {
	      "name": "World Population Statistics",
	      "observations": [
	        [ "Germany", "Total", "2009", 81.9, "Normal value" ],
	        [ "Germany", "Total", "2010", 81.7, "Normal value" ],
	        [ "Mexico", "Total", "2009", 107.6, "Break" ],
	        [ "Mexico", "Total", "2010", 112.3, "Normal value" ]
	      ]
	    } 
	  ],
	  "structure": {
	    "dimensions": [ "Country", "Type", "Year" ],
	    "measures": [ "Population" ], 
	    "attributes": [ "Status" ]
	  }
	}

Summary of changes:

- Rename dataSet field to dataSets and change it to an array
- Add optional name field to data set

## 5. Add Information to Structure

Next we start adding more information to the structure field. First we change dimensions, measures and attributes from strings into objects so that it becomes easy to add more fields to them e.g. unique id. Key position field holds the position of each dimension in the "key" request parameter.

	{
	  "dataSets" = [ 
	    {
	      "name": "World Population Statistics",
	      "observations": [
	        [ "Germany", "Total", "2009", 81.9, "Normal value" ],
	        [ "Germany", "Total", "2010", 81.7, "Normal value" ],
	        [ "Mexico", "Total", "2009", 107.6, "Break" ],
	        [ "Mexico", "Total", "2010", 112.3, "Normal value" ]
	      ]
	    } 
	  ],
	  "structure": {
	    "dimensions": [ 
	      { 
	        "id": "REF_AREA",
	        "name": "Country",
	        "keyPosition": 1
	      }, 
	      { 
	        "id": "REF_TYPE",
	        "name": "Type",
	        "keyPosition": 2
	      }, 
	      { 
	        "id": "TIME_PERIOD",
	        "name": "Year" 
	      }
	    ],
	    "measures": [ 
	      { 
	        "id": "OBS_VALUE",
	        "name": "Population" 
	      }
	    ], 
	    "attributes": [ 
	      { 
	        "id": "OBS_STATUS",
	        "name": "Status"
	      }
	    ]
	  }
	}

Changes:

- Change dimensions. measures and attributes from arrays of strings to arrays of objects
- Add new fields: name, id and keyPosition

## 6. Encode Dimension and Attribute Values

Next we move back to the data sets. One of the key concerns for the format is brevity so that data can be transmitted quickly. However if we have thousands or millions of observations in the data set current format is very verbose because the dimension and attribute values are repeated for each observation. We can solve this problem with encoding. 

Because dimensions and attributes typically take on a limited number of different values we can store them as integers in the dataset and keep the actual values as part of the structure. The integers map directly to the array indices in the values fields.    Statistical package R uses a similar method for storing factor (or categorical) variables. 

	{
	  "dataSets" = [ 
	    {
	      "name": "World Population Statistics",
	      "observations": [
	        [ 0, 0, 0, 81.9, 0 ],
	        [ 0, 0, 1, 81.7, 0 ],
	        [ 1, 0, 0, 107.6, 1 ],
	        [ 1, 0, 1, 112.3, 0 ]
	      ]
	    } 
	  ],
	  "structure": {
	    "dimensions": [ 
	      { 
	        "id": "REF_AREA",
	        "name": "Country",
	        "keyPosition": 1,
	        "values": [ "Germany", "Mexico" ]
	      }, 
	      { 
	        "id": "REF_TYPE",
	        "name": "Type",
	        "keyPosition": 2,
	        "values": [ "Total" ]
	      }, 
	      { 
	        "id": "TIME_PERIOD",
	        "name": "Year",
	        "values": [ "2009", "2010" ]
	      }
	    ],
	    "measures": [ 
	        "id": "OBS_VALUE",
	        "name": "Population" 
	    ], 
	    "attributes": [ 
	      { 
	        "id": "OBS_STATUS",
	        "name": "Status",
	        "values": [ "Normal value", "Break" ]
	      }
	    ]
	  }
	}

Summary of changes:

- Encode dimension and attribute values in the data set with integers
- Add dimension and attribute values to structure

## 7. Add Information to Dimension and Attribute Values

In the previous step we added dimension and attribute values to the structure. This reduces the size of the transmission but it also has other benefits:

- All the available dimension and attribute values are stored in simple arrays. There is no need to scan the data sets in order to find them.
- It becomes possible to add more information to individual dimension and attribute values without major impact to the size of the transmission.

We can now change the simple arrays of dimension/attribute values from strings to objects and add more information e.g. optional unique ids and start/end dates for time periods.

	{
	  "dataSets" = [ 
	    {
	      "name": "World Population Statistics",
	      "observations": [
	        [ 0, 0, 0, 81.9, 0 ],
	        [ 0, 0, 1, 81.7, 0 ],
	        [ 1, 0, 0, 107.6, 1 ],
	        [ 1, 0, 1, 112.3, 0 ]
	      ]
	    } 
	  ],
	  "structure": {
	    "dimensions": [ 
	      { 
	        "id": "REF_AREA",
	        "name": "Country",
	        "keyPosition": 1,
	        "values": [
	          { 
	            "id": "DE",
	            "name": "Germany" 
	          },
	          { 
	            "id": "MX",
	            "name": "Mexico"
	          }
	        ]
	      }, 
	      { 
	        "id": "REF_TYPE",
	        "name": "Type",
	        "keyPosition": 2,
	        "values": [
	          {
	            "id": "T", 
	            "name": "Total" 
	          }
	        ]
	      }, 
	      { 
	        "id": "TIME_PERIOD",
	        "name": "Year",
	        "values": [
	          {
	            "id": "2009", 
	            "name": "2009",
	            "start": "2009-01-01T00:00:00.000Z",
	            "end": "2010-01-01T00:00:00.000Z"
	          },
	          {
	            "id": "2010",
	            "name": "2010"
	            "start": "2010-01-01T00:00:00.000Z",
	            "end": "2011-01-01T00:00:00.000Z"
	          }
	        ]
		      }
	    ],
	    "measures": [ 
	        "id": "OBS_VALUE",
	        "name": "Population" 
	    ], 
	    "attributes": [ 
	      { 
	        "id": "OBS_STATUS",
	        "name": "Status",
	        "values": [
	          {
	            "id": "A",
	            "name": "Normal value" 
	          },
	          {
	            "id": "B",
	            "name": "Break"
	          }
	        ]
	      }
	    ]
	  }
	}

Summary of changes:

- Change dimension/attribute value strings to objects
- Move strings into name fields
- Add new fields: id and start/end dates

## 8. Group Dimension and Attribute Values

So far we have repeated all dimension and attribute values for each observation. Often it would be useful to group similar values together so that they are not repeated. This will make the transmission smaller but it will also make it clearer which observations share similar values. Grouping is optional and it can also be requested by the client.

In the sample data the type dimension only contains one value "Total". We can group this value so that it applies to the whole data set. We need to also update the structure so that it reflects this change.

	{
	  "dataSets" = [ 
	    {
	      "name": "World Population Statistics",
	      "dimensions": [ 0 ],
	      "observations": [
	        [ 0, 0, 81.9, 0 ],
	        [ 0, 1, 81.7, 0 ],
	        [ 1, 0, 107.6, 1 ],
	        [ 1, 1, 112.3, 0 ]
	      ]
	    } 
	  ],
	  "structure": {
	    "dimensions": {
	      "dataSet": [
	        { 
	          "id": "REF_TYPE",
	          "name": "Type",
	          "keyPosition": 2,
	          "values": [
	            {
	              "id": "T", 
	              "name": "Total" 
	            }
	          ]
	        } 
	      ],
	      "observation": [
	        { 
	          "id": "REF_AREA",
	          "name": "Country",
	          "keyPosition": 1,
	          "values": [
	            { 
	              "id": "DE",
	              "name": "Germany" 
	            },
	            { 
	              "id": "MX",
	              "name": "Mexico"
	            }
	          ]
	        }, 
	        { 
	          "id": "TIME_PERIOD",
	          "name": "Year",
	          "values": [
	            {
	              "id": "2009", 
	              "name": "2009",
	              "start": "2009-01-01T00:00:00.000Z",
	              "end": "2010-01-01T00:00:00.000Z"
	            },
	            {
	              "id": "2010",
	              "name": "2010"
	              "start": "2010-01-01T00:00:00.000Z",
	              "end": "2011-01-01T00:00:00.000Z"
	            }
	          ]
		        }
	      ]
	    },
	    "measures": [ 
	        "id": "OBS_VALUE",
	        "name": "Population" 
	    ], 
	    "attributes": {
	      "observation": [ 
	        { 
	          "id": "OBS_STATUS",
	          "name": "Status",
	          "values": [
	            {
	              "id": "A",
	              "name": "Normal value" 
	            },
	            {
	              "id": "B",
	              "name": "Break"
	            }
	          ]
	        }
	      ]
	    }
	  }
	}

Summary of changes:

- Add dimensions field to data set
- Add dataSet and observation fields to dimensions and attributes in structure
## 9. Introduce Series Grouping

In the previous step we introduced grouping of dimension/attribute values on the data set level. Groupped values apply to all the observations in the data set. The format supports one intermediate level of grouping at the "series" level. This is again optional and can be requested by the client.

In order to support series level groups we need to add more fields to the data set and split the observations into series groups. In addition we add series field to dimensions/attribute in the structure. 

In the sample data we can group either by the "Country" or "Year" dimensions. Creating series groups that include only the time dimension at the observation level is quite common and the name "series" is indeed derived from "time-series".

	{
	  "dataSets" = [ 
	    {
	      "name": "World Population Statistics",
	      "dimensions": [ 0 ],
	      "series": [
	        {
	          "dimensions": [ 0 ], 
	          "observations": [
	            [ 0, 81.9, 0 ],
	            [ 1, 81.7, 0 ]
	          ]
	        },
	        {
	          "dimensions": [ 1 ],
	          "observations": [
	            [ 0, 107.6, 1 ],
	            [ 1, 112.3, 0 ]
	          ]
	        }
	      ]
	    } 
	  ],
	  "structure": {
	    "dimensions": {
	      "dataSet": [
	        { 
	          "id": "REF_TYPE",
	          "name": "Type",
	          "keyPosition": 2,
	          "values": [
	            {
	              "id": "T", 
	              "name": "Total" 
	            }
	          ]
	        } 
	      ],
	      "series": [
	        { 
	          "id": "REF_AREA",
	          "name": "Country",
	          "keyPosition": 1,
	          "values": [
	            { 
	              "id": "DE",
	              "name": "Germany" 
	            },
	            { 
	              "id": "MX",
	              "name": "Mexico"
	            }
	          ]
	        }
	      ], 
	      "observation": [
	        { 
	          "id": "TIME_PERIOD",
	          "name": "Year",
	          "values": [
	            {
	              "id": "2009", 
	              "name": "2009",
	              "start": "2009-01-01T00:00:00.000Z",
	              "end": "2010-01-01T00:00:00.000Z"
	            },
	            {
	              "id": "2010",
	              "name": "2010"
	              "start": "2010-01-01T00:00:00.000Z",
	              "end": "2011-01-01T00:00:00.000Z"
	            }
	          ]
		        }
	      ]
	    },
	    "measures": [ 
	        "id": "OBS_VALUE",
	        "name": "Population" 
	    ], 
	    "attributes": {
	      "observation": [ 
	        { 
	          "id": "OBS_STATUS",
	          "name": "Status",
	          "values": [
	            {
	              "id": "A",
	              "name": "Normal value" 
	            },
	            {
	              "id": "B",
	              "name": "Break"
	            }
	          ]
	        }
	      ]
	    }
	  }
	}

Summary of changes:

- Add series field to data set
- Group dimensions and observations into series objects
- Add series field to dimensions/attributes in structure
## 10. Add Header field

Finally we just need to add a mandatory header field and we have a complete transmission ready. Header contains mostly technical information that helps to maintain robust data communications. However it also contains information about the sender of the information and this could be useful for users.

	{
	  "header": {
	    "id": "b1804c51-1ee3-45a9-bb75-795cd4e06489",
	    "prepared": "2013-01-03T12:54:12",
	    "sender: {
	      "id": "SDMX"
		    }
	  },
	  "dataSets" = [ 
	    {
	      "name": "World Population Statistics",
	      "dimensions": [ 0 ],
	      "series": [
	        {
	          "dimensions": [ 0 ], 
	          "observations": [
	            [ 0, 81.9, 0 ],
	            [ 1, 81.7, 0 ]
	          ]
	        },
	        {
	          "dimensions": [ 1 ],
	          "observations": [
	            [ 0, 107.6, 1 ],
	            [ 1, 112.3, 0 ]
	          ]
	        }
	      ]
	    } 
	  ],
	  "structure": {
	    "dimensions": {
	      "dataSet": [
	        { 
	          "id": "REF_TYPE",
	          "name": "Type",
	          "keyPosition": 2,
	          "values": [
	            {
	              "id": "T", 
	              "name": "Total" 
	            }
	          ]
	        } 
	      ],
	      "series": [
	        { 
	          "id": "REF_AREA",
	          "name": "Country",
	          "keyPosition": 1,
	          "values": [
	            { 
	              "id": "DE",
	              "name": "Germany" 
	            },
	            { 
	              "id": "MX",
	              "name": "Mexico"
	            }
	          ]
	        }
	      ], 
	      "observation": [
	        { 
	          "id": "TIME_PERIOD",
	          "name": "Year",
	          "values": [
	            {
	              "id": "2009", 
	              "name": "2009",
	              "start": "2009-01-01T00:00:00.000Z",
	              "end": "2010-01-01T00:00:00.000Z"
	            },
	            {
	              "id": "2010",
	              "name": "2010"
	              "start": "2010-01-01T00:00:00.000Z",
	              "end": "2011-01-01T00:00:00.000Z"
	            }
	          ]
		        }
	      ]
	    },
	    "measures": [ 
	        "id": "OBS_VALUE",
	        "name": "Population" 
	    ], 
	    "attributes": {
	      "observation": [ 
	        { 
	          "id": "OBS_STATUS",
	          "name": "Status",
	          "values": [
	            {
	              "id": "A",
	              "name": "Normal value" 
	            },
	            {
	              "id": "B",
	              "name": "Break"
	            }
	          ]
	        }
	      ]
	    }
	  }
	}

Summary of changes:

- Add header field

Now we are finally done with the complete SDMX-JSON message. The key changes from the original table are:

- Separate structure from data
- Add much more information about the structure
- Encode dimension and attribute values 
- Add more information about dimension/attribute values
- Optionally group the dimensions and attributes at data set and series levels

[1]:	www.json.org
[2]:	www.sdmx.org
