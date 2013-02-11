# TODO

- Add a sample file with multiple data sets and different dataSetActions: replace, delete, update.
- Look into annotations. Are they needed in the format? and if yes how should they be implemented?
- Look into linking to related SDMX structures (like the current "href" field in structure).
- Look into grouping fields in packaging into dimensions and attributes, e.g. packaging.dimensions.dataSet.
- Look into the following suggestion from the web developers:

> The JSON response message should include the filter settings specified in the request URL. 
> In order to build up a user interface one needs to know which dimensions have selections on them. 
> Currently the only way to do this is to know what URL was issued. 
> However, it’s frequently much easier if the response contains all the state necessary to build up the user interface. 
> This can be accomplished by adding a “selected” key to the dimension code definitions.

