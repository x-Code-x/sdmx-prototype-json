# Draft Field Guide to SDMX-PROTO-JSON Objects

**json-code-index format** **v0.1.0**

Use this guide to better understand SDMX-PROTO-JSON objects.

- [Introduction](#Introduction)
- [Message](#Message)
- [Header](#Header)
- [Structure](#Structure)
- [DataSets](#DataSets)
- [Tutorial: Handling component values](#handling_values)

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

