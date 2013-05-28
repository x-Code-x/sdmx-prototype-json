# Prototypes for the SDMX JSON format.

## Pre-Draft

- **json-pre-draft**. Latest pre-draft version. Combination of two prototypes below.

## Prototypes

- **json-slice**. Data part of the message contains up to three levels: data set, series and observation. Dimensions and attributes may be attached to any of these three levels. The way this is done in a particular message is documented in the packaging element.
- **json-code-index**. The data of one data set are not logically grouped by any level or dimension but enumerated in a flat list. Each observation is directly accessible through a specific property key within the data object. Attributes are managed in a similar way and can be attached at any level within the data set, including any combination of dimensions.

## Further information

- See <http://www.sdmx.org> for information on SDMX
- See <http://www.json.org> for information on JSON
