# TODO

## To do

- Add "measure" or "measures" into packaging?
- Look into grouping fields in packaging into dimensions and attributes, e.g. packaging.dimensions.dataSet or
packaging.dataSet.dimensions.
- Add "ref" field to dataSet? Already defined for structure.
- Look into annotations. Are they needed in the format? and if yes how should they be implemented?
- Add sample file with examples of all supported fields in the message.

## Waiting for further input

- Look into the following suggestion from the web developers:

> The JSON response message should include the filter settings specified in the request URL. 
> In order to build up a user interface one needs to know which dimensions have selections on them. 
> Currently the only way to do this is to know what URL was issued. 
> However, it’s frequently much easier if the response contains all the state necessary to build up the user interface. 
> This can be accomplished by adding a “selected” key to the dimension code definitions.

## Done
- Move components from the components array into the arrays in packaging and remove components array. Add field for the key position into dimension components.
- Add a sample file with multiple data sets and different dataSetActions: Append, Delete, Update. 
Sample file exr-action-delete contains and example of deletions. Append and Update are basically same as Informational.
- Look into linking to related SDMX structures (like the current "href" field in structure). Structure contains field "ref"
with information linking back to datastructure, provisionagreement or dataflow.
