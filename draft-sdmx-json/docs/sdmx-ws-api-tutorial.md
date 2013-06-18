# ECB SDMX 2.1 RESTful Web Service Documentation

- [Data queries](#data)
- [Metadata queries](#metadata)
- [Schema queries](#schemas)
- [Content negotiation](#negotiation)
- [Error handling](#errors)
- [Contact us](#contact)

## Overview

The ECB [SDMX 2.1 RESTful web service][1] offers programmatic access to the
statistical data and metadata disseminated via the [ECB Statistical Data
Warehouse][2].

All the data stored in the Statistical Data Warehouse can be retrieved using the
syntax described below.


## <a name="data"></a> Query syntax

The following syntax should be used for data queries:

    protocol://wsEntryPoint/resource/flowRef/key?startPeriod=value&endPeriod=val
    ue&updatedAfter=value&firstNObservations=value&lastNObservations=value&detai
    l=value&includeHistory=value


#### Protocol

The web service is available over http and https.


#### Web service entry point

The web service entry point is available at the following location:

    http://a-sdw-ecb-wsrest.ecb.de/service/


#### Resource

The resource for data queries is *data*.


#### The flowRef path parameter: Defining the dataflow reference

A reference to the dataflow describing the data that needs to be returned.

The syntax is the identifier of the agency maintaining the dataflow, followed by
the identifier of the dataflow, followed by the dataflow version, separated by a
*comma*. For example: AGENCY_ID,FLOW_ID,VERSION

If the parameter contains only one of these 3 elements, it is considered to be
the identifier of the dataflow. The value for the identifier of the agency
maintaining the dataflow will default to *all*, while the value for the dataflow
version will default to *latest*.

If the string contains only two of these 3 elements, they are considered to be
the identifier of the agency maintaining the dataflow and the identifier of the
dataflow. The value for the dataflow version will default to *latest*.

In order to see the dataflows available in the ECB statistical data warehouse, a
metadata query for all dataflows can be performed:

    http://a-sdw-ecb-wsrest.ecb.de/service/dataflow


#### The key path parameter: Defining the dimension values of the data to be returned

As explained [below](#metadata), the combination of dimensions allows
statistical data to be uniquely identified. Such a combination is known as a
series key in SDMX and this is what is needed in the *key* parameter.

Let's say for example that exchange rates can be uniquely identified by the
frequency at which they are measured (e.g.: on a daily basis - code *D*), the
currency being measured (e.g.: US dollar - code *USD*), the currency against
which a currency is being measured (e.g.: Euro - code *EUR*), the type of
exchange rates (Foreign exchange reference rates - code *SP00*) and the series
variation (such as average or standardised measure for given frequency, code
*A*). In order to build the series key, you need to take the value for each of
the dimensions (in the order in which the dimensions are defined in the DSD) and
join them with a *dot*. The series key for the example above therefore becomes:
D.USD.EUR.SP00.A

Wildcarding is supported by omitting the value for the dimension to be
wildcarded. For example, the following series key can be used to retrieve the
data for all daily currencies against the euro: D..EUR.SP00.A

The OR operator is supported using the *+* character. For example, the following
key can be used to retrieve the exchange rates against the euro for both the US
dollar and the Japanese Yen: D.USD+JPY.EUR.SP00.A

You can of course combine wildcarding and the OR operator. For example, the
following key can be used to retrieve daily or monthly exchange rates of any
currency against the euro: D+M..EUR.SP00.A


#### The startPeriod and endPeriod query parameters: Defining a date range

It is possible to define a date range for which observations should be returned
by using the *startPeriod* and/or *endPeriod* parameters. The values should be
given according to the syntax defined in ISO 8601 or as SDMX reporting periods.
The values will vary depending on the frequency. The supported formats are:

- *YYYY* for annual data (e.g.: 2013).
- *YYYY-S[1-2]* for semi-annual data (e.g.: 2013-S1).
- *YYYY-Q[1-4]* for quarterly data (e.g.: 2013-Q1).
- *YYYY-MM* for monthly data (e.g.: 2013-01).
- *YYYY-W[01-53]* for weekly data (e.g.: 2013-W01).
- *YYYY-MM-DD* for daily data (e.g.: 2013-01-01).


#### The updatedAfter query parameter: Retrieving deltas

By supplying a percent-encoded timestamp to the *updatedAfter* parameter, it is
possible to only retrieve the latest version of changed values in the database
since a certain point in time (so-called updates and revisions). This will
include:

- The observations that have been added since the last time the query was
performed.
- The observations that have been revised since the last time the query was
performed.
- The observations that have been deleted since the last time the query was
performed.

For example, the percent-encoded representation for 2009-05-15T14:15:00+01:00
would be: 2009-05-15T14%3A15%3A00%2B01%3A00

Developers who update their local databases with data stored in the ECB
Statistical Data Warehouse should make use of the *updatedAfter* parameter as
this will significantly improve performance: Instead of systematically
downloading data that have not changed, you will only receive the changes made
in our database since the last time your client performed the same query.


#### The detail query parameter: Defining the amount of details

Using the *detail* parameter, it is possible to specify the desired amount of
information to be returned by the web service. Possible options are:

- *full*:  The data - series and observations - and the attributes will be
returned. This is the default.
- *dataonly*: The attributes will be excluded from the returned message.
- *serieskeysonly*: Only the series, but without the
attributes and the observations, will be returned. This can be useful for
performance reasons, to return the series that match a certain query, without
returning the actual data.
- *nodata*: The series, including the attributes,
will be returned, but the observations will not be returned.


#### The firstNObservations and lastNObservations query parameters: Defining the number of observations to be returned

Using the *firstNObservations* and/or *lastNObservations* parameters, it is
possible to specify the maximum number of observations to be returned for each
of the matching series, starting from the first observation
(*firstNObservations*) or counting back from the most recent observation
(*lastNObservations*).


#### Using the *includeHistory* query parameter: Retrieving how a time series evolved over time

Using the *includeHistory* parameter, you can instruct the web service to return
previous versions of the matching data. This is useful to see how the data have
evolved over time, i.e. when new data have been released or when data have been
revised or deleted. Possible options are:

- *false*: Only the version currently in production will be returned. This is
the default.
- *true*: The version currently in production, as well as all previous versions,
will be returned.



### Examples

To retrieve the data for the series M.USD.EUR.SP00.A for the EXR dataflow:

    http://a-sdw-ecb-wsrest.ecb.de/service/data/EXR/M.USD.EUR.SP00.A

To retrieve the data  for the EXR dataflow, for the supplied series keys, using wildcarding for the second dimension:

    http://a-sdw-ecb-wsrest.ecb.de/service/data/EXR/M..EUR.SP00.A

To retrieve the updates and revisions for the data matching the supplied series keys, using the OR operator for the second dimension:

    http://a-sdw-ecb-wsrest.ecb.de/service/data/EXR/M.USD+GBP+JPY.EUR.SP00.A?updatedAfter=2009-05-15T14%3A15%3A00%2B01%3A00

As mentioned previously, the ISO 8601 timestamp needs to be percent-encoded.

To retrieve the data matching the supplied series key and restricting the start and end dates:

    http://a-sdw-ecb-wsrest.ecb.de/service/data/EXR/D.USD.EUR.SP00.A?startPeriod=2009-05-01&endPeriod=2009-05-31

To retrieve all the data for the EXR dataflow:

    http://a-sdw-ecb-wsrest.ecb.de/service/data/EXR



## <a name="metadata"></a> Metadata queries

Various metadata resources can be retrieved via this web service:

- *Concept schemes*: In order to make sense of some statistical data, we need to
know the concepts associated with them. For example, on its own, the figure
1.2953 is pretty meaningless, but if we know that this is an exchange rate for
the US dollar against the euro on 23 November 2006, it starts making more sense.
The various concepts used in the SDW are grouped into collections known as
concept schemes.

- *Code lists:* Some of the concepts can be free text (such as a comment about a
particular observation value) but others take their values from a controlled
vocabulary list (such as, for example, a list of countries). These are known as
*code lists* in SDMX.

- *Data structure definitions:* All the concepts that describe a particular domain
(such as exchange rates or inflation) are grouped into a *data structure
definition (DSD)*. In a DSD, concepts are divided into *dimensions* and
*attributes*. Dimensions, when combined, allow to uniquely identify statistical
data. Attributes on the other hand do not help identifying statistical data, but
add useful information (like the unit of measure or the number of decimals). In
order to perform granular data queries, one must know the concepts that are used
as dimensions, as well as their allowed values (as defined in the code lists).

- *Dataflows:* *Dataflows* represent the data that cover a particular domain (such
as, for example, balance of payments). A dataflow provides a reference to the
data structure definition that applies for a particular domain, thereby
indicating how the data for that domain will look like.

- *Organisation schemes:* Organisations that play a role in a particular
statistical context are defined in *organisation schemes*. The type of
organisation scheme will depend on the role played by a particular organisation
(data consumer, data provider, etc.). Organisations that define the metadata
used in a particular context are known as maintenance agencies and are grouped
together in an *agency scheme*.


### Query syntax

The following syntax is used for metadata queries:

    protocol://wsEntryPoint/resource/agencyID/resourceID/version?detail=value&references=value

The syntax is further explained in the sections below.

#### Protocol

The web service is available over http and https.

#### Web service entry point

The web service entry point is available at the following location:

    http://a-sdw-ecb-wsrest.ecb.de/service/

#### Resources

The following resources are currently made available by the web service:
conceptscheme, codelist, datastructure, dataflow and agencyscheme.

#### Path parameters

The path parameters are used to identify a particular metadata resource. In
SDMX, a resource of a particular type (such as a code list) can be uniquely
identified by combining the identifier of the agency maintaining the resource,
the identifier of the resource and the version number. When the version number
is not supplied, the latest version of the resource is returned.

Let's take the example of the first version (1.0) of the frequency code list
(with id CL_FREQ), maintained by the European Central Bank (id is ECB). In order
to retrieve this, the following should be used:

    http://a-sdw-ecb-wsrest.ecb.de/service/codelist/ECB/CL_FREQ/1.0

#### Query parameters

The query parameters are used to further describe the desired results. There are
two allowed parameters: *references* and *detail*.

- Using the *references* parameter, you can for example instruct the web service
to return (or not) the artefacts referenced by the artefact to be returned (for
example, the code lists and concepts used by the data structure definition
matching the query). You can also retrieve the artefacts that use the matching
artefact (for example, the dataflows that use the data structure definition
matching the query). Possible values are:

    - *none*: No references will be returned. This is the default.
    - *parents*: The artefacts that use the artefact matching the query
    (for example, the dataflows that use the data structure definition
    matching the query) will be returned.
    - *parentsandsiblings*: The artefacts that use the artefact
    matching the query, as well as the artefacts referenced by these
    artefacts will be returned.
    - *children*: The artefacts referenced by the matching artefact will be returned
    (for example, the concept schemes and code lists used in a DSD).
    - *descendants*: References of references, up to any level, will also be returned.
    - *all*: The combination of parentsandsiblings and descendants.
    - In addition, a concrete type of resource,
    may also be used (for example, references=codelist).

- Using the *detail* parameter, you can specify the desired amount of information
to be returned. For example, it is possible to instruct the web service to
return only basic information about the resource (i.e.: its id, agency id,
version and name. This is also known in SDMX as a *stub*). The allowed values
are:

    - *allstubs*: All artefacts will be returned as stubs.
    - *referencestubs*: The referenced artefacts will be returned as stubs.
    - *full*: All available information for all artefacts will be returned. This is the default.

#### Representation

Metadata are returned in the SDMX-ML 2.1 Structure format.


### Examples

To retrieve version 1.0 of the DSD with id ECB_EXR1 maintained by the ECB,
as well as the code lists and the concepts used in the DSD:

    http://a-sdw-ecb-wsrest.ecb.de/service/datastructure/ECB/ECB_EXR1/1.0?references=children

To retrieve the latest version in production of the DSD with id
ECB_EXR1 maintained by the ECB, without the code lists and concepts of the DSD:

    http://a-sdw-ecb-wsrest.ecb.de/service/datastructure/ECB/ECB_EXR1

You can also use the keyword *latest*:

    http://a-sdw-ecb-wsrest.ecb.de/service/datastructure/ECB/ECB_EXR1/latest

To retrieve all DSDs maintained by the ECB, as well as the dataflows using these DSDs:

    http://a-sdw-ecb-wsrest.ecb.de/service/datastructure/ECB?references=dataflow

You can also use the keywords *all* and *latest*:

    http://a-sdw-ecb-wsrest.ecb.de/service/datastructure/ECB/all/latest?references=dataflow

To retrieve the latest version in production of all code lists maintained by all
maintenance agencies, but without the codes:

    http://a-sdw-ecb-wsrest.ecb.de/service/codelist?detail=allstubs

You can also use the *all* and *latest* keywords:

    http://a-sdw-ecb-wsrest.ecb.de/service/codelist/all/all/latest?detail=allstubs

The *structure* keyword can be used to retrieve metadata regardless of their
type (code list, concept scheme, etc.). For example, to retrieve, as stubs, the
latest version in production of all maintainable artefacts maintained by the
ECB:

    http://a-sdw-ecb-wsrest.ecb.de/service/structure/ECB?detail=allstubs

Again, you can also use the *all* and *latest* keywords:

    http://a-sdw-ecb-wsrest.ecb.de/service/structure/ECB/all/latest?detail=allstubs


## XML schema queries

XML schemas can be used to validate SDMX-ML Structure Specific Data messages returned by our web service.
In order to retrieve the schemas, the following syntax can be used:

    http://a-sdw-ecb-wsrest.ecb.de/resource/context/agencyID/resourceID/version

The syntax is further explained in the sections below.


### Resource

The resource to be used to retrieve an XML schema is *schema*.


### Context

The context determines the constraints that need to be taken into account when
generating the schema. For the time being, the only supported context is
*datastructure*.


### Information about the data structure definition (DSD)

The id of the agency maintaining the DSD, the id of the DSD and its version also
need to be supplied. To retrieve the version currently in production of a DSD,
use the keyword *latest*.


### Example

In order to retrieve the XML schema that defines the validation rules for data
structured according to the latest version of the ECB_EXR1 DSD maintained by the
ECB, the following query should be executed:

    http://a-sdw-ecb-wsrest.ecb.de/schema/datastructure/ECB/ECB_EXR1

The keyword *latest* can also be used for the version parameter:

    http://a-sdw-ecb-wsrest.ecb.de/schema/datastructure/ECB/ECB_EXR1/latest


## Using HTTP content negotiation

Using the HTTP content negotiation mechanism, you can select the representation
to be returned and you can also instruct the service to compress the data to be
returned.

For metadata queries, the only supported representation supported at this stage
is the SDMX-ML 2.1 Structure format.

For data queries, by default, the web service will return data in SDMX-ML 2.1
Generic Data format. If you wish to retrieve the data in the SDMX-ML 2.1
Structure Specific Data format instead, the following mime type should be
supplied in the Accept HTTP header:

    application/vnd.sdmx.structurespecificdata+xml;version=2.1

For additional information about the various SDMX-ML formats, please refer to the
[SDMX documentation][3].

You can also enable data compression using the Accept-Encoding HTTP Header
field. Compressed messages are typically around 15 times smaller than
uncompressed messages, which can lead to some improvements when transferring
large data sets over the network.


## Error handling

The web service returns errors using an HTTP status code. The following errors can be returned:

### Syntax error (400)

If there is a syntactic or semantic issue with the parameters you supplied, a
400 HTTP status code will be returned.

### No results found (404)

A 404 HTTP status code will be returned if there are no results matching the
query.

### Internal Server Error (500)

When there is an issue on our side, a 500 HTTP status code will be returned.
Feel free to try again later or to contact our [support hotline][4].

### Not implemented (501)

This web service offers a subset of the functionality offered by the SDMX
RESTful web service specification. When you use a feature that we have not yet
implemented, a 501 HTTP status code will be returned.

### Service unavailable (503)

If our web service is temporarily unavailable, a 503 HTTP status code will be
returned.


## Contact us

If the documentation does not contain the information you require, or if you have any general comments or
feedback regarding our web service, please [contact us][5].


[1]: http://sdmx.org/wp-content/uploads/2012/12/SDMX_2_1-SECTION_07_WebServicesGuidelines_201211.doc "SDMX 2.1 RESTful web service specification"
[2]: http://sdw.ecb.europa.eu "ECB Statistical Data Warehouse"
[3]: http://sdmx.org/wp-content/uploads/2011/08/SDMX_2-1-1_SECTION_3A_SDMX_ML_201108.zip "SDMX documentation"
[4]: mailto:statistics@ecb.europa.eu "support hotline"
[5]: mailto:statistics@ecb.europa.eu?subject=[SDMX%20Web%20Service] "contact"
