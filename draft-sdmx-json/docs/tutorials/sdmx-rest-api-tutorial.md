# SDMX 2.1 RESTful Web Service Tutorial

**DRAFT VERSION**

## Introduction

This tutorial describes how to get access to the statistical data
and metadata in a SDMX 2.1 compliant Web Service via the [SDMX 2.1 RESTful API][1].

All the examples use [curl][2] command line tool. See the curl web site for more
details.


## Basic Concepts

In order to use the SDMX Restful API you need to know the URL of the SDMX web service.
This tutorial uses the following two Web Services as an example:

- SDMX Global Registry Web Service: http://test.sdmxregistry.org/ws/rest
- ECB Statistical Data Warehouse (SDW) Web Service (URL to come later)

Via the SDMX RESTful API you can retrieve:

- Data
- Structural Metadata
- XML Schemas

An example of structural metadata is a code list that contains codes for a statistical
concept (e.g. currency codes).
The following example requests a code list of currency codes from the SDMX Global Registry:

```Batchfile
curl -X GET \
  'http://test.sdmxregistry.org/ws/rest/codelist/ECB_FINAL/CL_CURRENCY/latest'
```

The part that comes after the `http://test.sdmxregistry.org/ws/rest` is
standard and it would work with any Web Service that contains the same code
list. Here is the same request to the ECB SDW:

```Batchfile
curl -X GET \
  'http://sdw-ws-entry-point/codelist/ECB/CL_CURRENCY/latest'
```

The server returns the response in one of the supported formats, such as SDMX-ML or SDMX-JSON.
For a full list of formats see the [Web Service Guidelines][1].
Other important information in
the response are the HTTP status code and the response and entity headers. The SDMX
RESTful API uses standard HTTP status codes. The status code for normal
responses is 200. Status codes in the 4xx class indicate that the client has
caused the error. Status codes in the 5xx class indicate an error on the server.
HTTP entity headers contain optional information about the response: language,
encoding, type, etc. See below for more information about error handling and
HTTP headers.

The SDMX RESTful API supports both HTTP and HTTPS
protocols but all the examples in this tutorial use plain HTTP.


## Requesting Data

Data is probably the most used resource on the SDMX Web Services. SDMX Global
Registry provides only structural metadata and schemas so all the examples in
the chapter use the ECB SDW.

Following example requests the daily USD/EUR exchange rates from the ECB SDW:

```Batchfile
curl -X GET \
  'http://sdw-ws-entry-point/data/EXR/D.USD.EUR.SP00.A'
```

The resource type for data requests is simply *data*. Data resouces are
indentified with a combination of three parameter values: `flowRef`, `key`, and
`providerRef`. The syntax for data requests is:

```
http://domain[:port]/[ws-path]/data/flowRef/[key]/[providerRef]?[query_string]
```

In the example above the flowRef is `EXR` and the key is `D.USD.EUR.SP00.A`.
The default value for providerRef is `all` and this example uses the default.


### FlowRef Parameter

FlowRef is used to reference to the dataflow describing the data that needs to
be returned.

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


### Key parameter

Key is used to define the dimension values of the data to be returned. As
explained [below](#metadata), the combination of dimensions allows statistical
data to be uniquely identified. Such a combination is known as a series key in
SDMX and this is what is needed in the *key* parameter.

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


### ProviderRef Parameter

ProviderRef is used to identify the data provider if the same dataflow is
available from many different providers. The default value for ProviderRef is
'all' and it returns all data matching the supplied key and belonging to the
specified dataflow that has been provided by any data provider.


### Additional Parameters for Data Requests

The SDMX RESTful API supports additional parameters for data requests. They are
used to further describe (or filter) the desired results, once the data resource
has been identified with the three parameters above. These additional parameters
are passed as part of the query string.


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

The frequency in the parameter does not have to match the frequency of the data
that is returned. The response will contain all the observations (unless
restricted by other parameters) where the time is greater than or equal to the
startPeriod and less than the endPeriod. For weekly data it is best to use the
weekly frequency in the parameter. Otherwise the results can be hard to
interpret because the start and end of other frequencies may not align with
start and end of weeks.


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

For example, the percent-encoded representation for 2009-05-15T14:15:00+01:00 would be: 2009-05-15T14%3A15%3A00%2B01%3A00

Developers who update their local databases with data stored in the Web Service
should make use of the *updatedAfter* parameter as this will significantly
improve performance: Instead of systematically downloading data that have not
changed, you will only receive the changes made in our database since the last
time your client performed the same query.


#### The detail query parameter: Defining the amount of details

Using the *detail* parameter, it is possible to specify the desired amount of
information to be returned by the web service. Possible options are:

- *full*:  The data - series and observations - and the attributes will be returned. This is the default.
- *dataonly*: The attributes will be excluded from the returned message.
- *serieskeysonly*: Only the series, but without the attributes and the observations, will be returned. This can be useful for performance reasons, to return the series that match a certain query, without returning the actual data.
- *nodata*: The series, including the attributes, will be returned, but the observations will not be returned.


#### The firstNObservations and lastNObservations query parameters: Defining the number of observations to be returned

Using the *firstNObservations* and/or *lastNObservations* parameters, it is
possible to specify the maximum number of observations to be returned for each
of the matching series, starting from the first observation
(*firstNObservations*) or counting back from the most recent observation
(*lastNObservations*).


### Data Request Examples

To retrieve more exchange rates from the EXR dataflow, using wildcarding for the second dimension:

```Batchfile
curl -X GET \
  'http://sdw-ws-entry-point/data/EXR/M..EUR.SP00.A'
```

Retrieve exchange rate for USD, GBP and JPY with the OR operator (+) in the key parameter:

```Batchfile
curl -X GET \
  'http://sdw-ws-entry-point/data/EXR/M.USD+GBP+JPY.EUR.SP00.A'
```

Restrict the time range in the request with the `startPeriod` and `endPeriod` parameters:

```Batchfile
curl -X GET \
  'http://sdw-ws-entry-point/data/EXR/D.USD.EUR.SP00.A?startPeriod=2009-05-01&endPeriod=2009-05-31'
```

To retrieve the updates and revisions for the data matching the supplied series key:

```Batchfile
curl -X GET \
  'http://sdw-ws-entry-point/data/EXR/M.USD.EUR.SP00.A?updatedAfter=2009-05-15T14%3A15%3A00%2B01%3A00'
```

To retrieve all the data for the EXR dataflow:

```Batchfile
curl -X GET \
  'http://sdw-ws-entry-point/data/EXR'
```

## <a name="metadata"></a> Request for Structural Metadata

Structural metadata defines the structures for the data in the Web Service. The
SDMX RESTful API support 21 different types of structural metadata resources.
Some of the most commonly used resources are: `conceptscheme`, `codelist`,
`datastructure`, `dataflow`, and `agencyscheme`. Please see the [SDMX User
Guide][3] for more information on structural metadata.

Following example requests a code list of currency codes from the SDMX Global Registry:

```Batchfile
curl -X GET \
  'http://test.sdmxregistry.org/ws/rest/codelist/ECB_FINAL/CL_CURRENCY/latest'
```

The following syntax is used for all structural metadata queries:

```
http://domain[:port]/[ws-path]/resource/[agencyID]/[resourceID]/[version]?[detail=value]&[references=value]
```

In the example above:

- `domain` = `test.sdmxregistry.org`
- `ws-path` = `/ws/rest`
- `resource` = `codelist`
- `agencyID` = `ECB_FINAL`
- `resourceID` = `CL_CURRENCY`
- `version` = `latest`

The default value for `agencyID` and `resourceID` is `all`. The default value
for `version` is `latest`. If none of the parameters is provided then the server
will return latest versions of all resources of specified type.

### Path parameters

The path parameters are used to identify a particular metadata resource. In
SDMX, a resource of a particular type (such as a code list) can be uniquely
identified by combining the identifier of the agency maintaining the resource,
the identifier of the resource and the version number. When the version number
is not supplied, the latest version of the resource is returned.

Let's take the example of the first version (1.0) of the frequency code list
(with id CL_FREQ), maintained by the European Central Bank (id is ECB). In order
to retrieve this, the following should be used:

```Batchfile
curl -X GET \
  'http://sdw-ws-entry-point/codelist/ECB/CL_FREQ/1.0'
```

### Query parameters

The query parameters are used to further describe the desired results. There are
two allowed parameters: `references` and `detail`.

#### References Parameter

Using the `references` parameter, you can for example instruct the web service
to return (or not) the artefacts referenced by the artefact to be returned (for
example, the code lists and concepts used by the data structure definition
matching the query). You can also retrieve the artefacts that use the matching
artefact (for example, the dataflows that use the data structure definition
matching the query). Possible values are:

- `none` No references will be returned. This is the default.
- `parents` The artefacts that use the artefact matching the query (for example, the dataflows that use the data structure definition matching the query) will be returned.
- `parentsandsiblings` The artefacts that use the artefact matching the query, as well as the artefacts referenced by these artefacts will be returned.
- `children` The artefacts referenced by the matching artefact will be returned (for example, the concept schemes and code lists used in a DSD).
- `descendants` References of references, up to any level, will also be returned.
- `all` The combination of parentsandsiblings and descendants.
- In addition, a concrete type of resource, may also be used (for example, references=codelist).

#### Detail parameter

Using the `detail` parameter, you can specify the desired amount of information
to be returned. For example, it is possible to instruct the web service to
return only basic information about the resource (i.e.: its id, agency id,
version and name. This is also known in SDMX as a *stub*). The allowed values
are:

- `allstubs` All artefacts will be returned as stubs.
- `referencestubs` The referenced artefacts will be returned as stubs.
- `full` All available information for all artefacts will be returned. This is the default.

### Structural Metadata Request Samples

Request all code lists for the maintenance agency ECB_FINAL:

```Batchfile
curl -X GET \
  'http://test.sdmxregistry.org/ws/rest/codelist/ECB_FINAL'
```

Request specific version of the CL\_CURRENCY code list maintained by ECB\_FINAL:

```Batchfile
curl -X GET \
  'http://test.sdmxregistry.org/ws/rest/codelist/ECB_FINAL/CL_CURRENCY/1.0'
```

Request latest version of the CL\_CURRENCY code lists maintained by ECB\_FINAL
with full details and no references (same as above):

```Batchfile
curl -X GET \
  'http://test.sdmxregistry.org/ws/rest/datastructure/ECB_FINAL/ECB_EXR1/latest/?detail=full&references=none'
```

Request latest version of data structure ECB\_EXR1 maintained by ECB\_FINAL with
full details and all structures references by it. References structures will be
"stubs" with no details:

```Batchfile
curl -X GET \
  'http://test.sdmxregistry.org/ws/rest/datastructure/ECB_FINAL/ECB_EXR1/latest/?detail=referencestubs&references=children'
```

The `structure` keyword can be used to retrieve metadata regardless of their
type (code list, concept scheme, etc.). For example, to retrieve, as stubs, the
latest version in production of all maintainable artefacts maintained by the
ECB_FINAL:

```Batchfile
curl -X GET \
  'http://test.sdmxregistry.org/ws/rest/structure/ECB_FINAL?detail=allstubs'
```


## XML Schema Requests

XML schemas can be used to validate SDMX-ML Structure Specific Data messages.
SDMX Web Services can provide these schemas for clients. Following example
requests XML schema for the data structure ECB\_EXR1 maintained by ECB\_FINAL:

```Batchfile
curl -X GET \
  'http://test.sdmxregistry.org/ws/rest/schema/datastructure/ECB_FINAL/ECB_EXR1'
```

In order to retrieve the schemas, the following syntax can be used:

```
http://domain[:port]/[ws-path]/schema/context/agencyID/resourceID[/version][&query_params]
```

In the example above:

- `domain` = `test.sdmxregistry.org`
- `ws-path` = `/ws/rest`
- `context` = `datastructure`
- `agencyID` = `ECB_FINAL`
- `resourceID` = `ECB_EXR1`

The default value for version is `latest` and the query parameters are optional.

### Context

The context determines the constraints that need to be taken into account when
generating the schema. Supported values are: `datastructure`,
`metadatastructure`, `dataflow`, `metadataflow` and `provisionagreement`.

### Information about the data structure definition (DSD)

The id of the agency maintaining the DSD, the id of the DSD and its version also
need to be supplied. To retrieve the version currently in production of a DSD,
use the keyword *latest*.

### Query Parameters

Schema request support two optional parameters:

- `dimensionAtObservation` takes a dimension value that is attached at the observation level.
- `explicitMeasure` indicates if the observations are strongly typed in the schema. This applies to cross-sectional data validation.


### Examples

In order to retrieve the XML schema that defines the validation rules for data
structured according to the latest version of the ECB_EXR1 DSD maintained by the
ECB_FINAL, the following query should be executed:

```Batchfile
curl -X GET \
  'http://test.sdmxregistry.org/ws/rest/schema/dataflow/ECB_FINAL/ECB_WEB'
```

The keyword `latest` can also be used for the version parameter:

```Batchfile
curl -X GET \
  'http://test.sdmxregistry.org/ws/rest/schema/datastructure/ECB_FINAL/ECB_EXR1/latest'
```

Query parameter `dimensionAtObservation` can also be used to change the
dimension at observation from TIME_PERIOD to FREQ:

```Batchfile
curl -X GET \
  'http://test.sdmxregistry.org/ws/rest/schema/datastructure/ECB_FINAL/ECB_EXR1?dimensionAtObservation=FREQ'
```


## Content Negotiation

SDMX RESTful API uses content negotiation to select the best representation of a
resource for a response. Content negotiation mechanism is part of the [HTTP
specification][4]. The Web Service may vary the content by media type, language,
encoding, and charset. The client should use the appropriate request-header
fields to describe its preferences.


### Media Type

Web Service may support different media types for the available resources. The
standard media types are listed in the [Web Services Guidelines][1]. Client
specifies the desired format and version of the resource using the `Accept` HTTP
header. In the following example SDMX-ML 2.0 structure message is specified as
the preferred format:

```Batchfile
curl -i -X GET \
  -H 'Accept:application/vnd.sdmx.structure+xml;version=2.0' \
  'http://test.sdmxregistry.org/ws/rest/codelist/ECB_FINAL/CL_FREQ/1.0'
```

If no `Accept` header field is present, then it is assumed that the client
accepts all media types and the will send the response in the most recent
version of SMDX-ML it supports. If an Accept header field is present, and if the
server cannot send a response which is acceptable according to the combined
Accept field value, then the server should send a 406 (not acceptable) response


### Language

Client can specify the natural language it prefers with the `Accept-Language`
HTTP header. If the Web Service contains the information in multiple languages,
it will send the response in the language of the client’s preference, if
available. In the following example the client prefers french but accepts also
british english and english:

```Batchfile
curl -i -X GET \
  -H 'Accept-Language:fr, en-gb;q=0.8, en;q=0.7' \
  'http://test.sdmxregistry.org/ws/rest/codelist/ECB_FINAL/CL_FREQ/1.0'
```

If no `Accept-Language` header is present in the request, the server assumes
that all languages are equally acceptable. If an `Accept-Language` header is
present, then all languages which are assigned a quality factor greater than 0
are acceptable.

The `Content-Language` response-header field may contain information about the
natural language of the response.


### Encoding

Data compression is an extremely important functionality in the SDMX Web
Services. Compressed messages are typically around 15 times smaller than
uncompressed messages, which can lead to some improvements when transferring
large data sets over the network. Data compression is negotiated with the
`Accept-Encoding` request-header field. In the following example client accepts
responses in either `compress` or `gzip` formats:

```Batchfile
curl -i -X GET \
  -H 'Accept-Encoding:compress,gzip' \
  'http://test.sdmxregistry.org/ws/rest/codelist/ECB_FINAL/CL_FREQ/1.0'
```

According to the HTTP specification if an `Accept-Encoding` field is present in
a request, and if the server cannot send a response which is acceptable
according to the `Accept-Encoding` header, then the server should send an error
response with the 406 (Not Acceptable) status code. If no `Accept-Encoding`
field is present in a request, the server may assume that the client will accept
any content coding.


### Charset

All SDMX-ML and SDMX-JSON messages use the UTF-8 character set, while SDMX-EDI
uses the ISO 8879-1 character set. The client may include these in the `Accept-
Charset` request-header field. Better alternative is not to provide `Accept-
Charset` field to the server.


## Handling Errors

Clients should always check the status code in the server response. The common
response code for successful request is 200, all other codes should be handled
as errors. The SDMX response should contain detailed error messages. The
following status codes can be returned:

### Bad Request (400)

There was a syntactic or semantic error in request parameters. Client should not
repeat the request. Example with erroneous query parameter:

```Batchfile
curl -i -X GET \
  'http://test.sdmxregistry.org/ws/rest/codelist/ECB_FINAL/CL_FREQ/1.0?referencess=all'
```

### Unauthorized (401)

The request did not contain proper authorization for the requested resource.
Client should provide proper authorization for this resource.

### Forbidden (403)

Server understood the request but refuses to fulfill it. More information about
the specific condition should be in the error messages. This could indicate that
the request results in a response that is larger than the server is willing or
able to process. Client should not repeat the request.

### Not Found (404)

There were no resources matching the request. Example with missing code list:

```Batchfile
curl -i -X GET \
  'http://test.sdmxregistry.org/ws/rest/codelist/ECB_FINAL/CL_MISSING_CODE_LIST'
```

### Not Acceptable (406)

The resource exists but not in the format preferred by the client. The client
should not repeat the request. Example where client requests a code list to be
return in Generic Data message:

```Batchfile
curl -i -X GET \
  -H 'Accept:application/vnd.sdmx.genericdata+xml;version=2.0' \
  'http://test.sdmxregistry.org/ws/rest/codelist/ECB_FINAL/CL_FREQ'
```

### Internal Server Error (500)

There was an unexpected error at the server. The request is probably fine.

### Not implemented (501)

Web Service may support a subset of the functionality offered by the SDMX
RESTful web service specification. If the request contains features that are not
implemented, a 501 HTTP status code will be returned. The error message should
contain information about the unsupported features. Client should not repeat the
request.

### Bad Gateway (502)

Server received invalid responses from another server (e.g. proxy). The request
is probably fine.

### Service Unavailable (503)

Web Service is temporarily unavailable but should be available in the future.


[1]: http://sdmx.org/wp-content/uploads/2012/12/SDMX_2_1-SECTION_07_WebServicesGuidelines_201211.doc "SDMX 2.1 RESTful web service specification"
[2]: http://curl.haxx.se "curl and libcurl"
[3]: http://sdmx.org/index.php?page_id=38 "SDMX User Guide"
[4]: http://www.w3.org/Protocols/rfc2616/rfc2616-sec12.html "Content Negotiation"
