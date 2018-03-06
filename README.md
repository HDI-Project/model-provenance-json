# Model Provenance Specification

This repo contains guidelines for constructing a `model_provenance.json` file for time-based predictive machine learning problems adhering to ML 2.0 principles.

The purpose of the `model_provenance.json` file is to document an entire machine learning workflow, from raw data to deployment, in such a way that anyone along any step of the process can recreate the results.
At a high-level, the file specifies the following steps:

 * Data preprocessing/loading (by linking to another file: [`metadata.json`](https://github.com/HDI-Project/MetaData.json))
 * Prediction Engineering
 * Feature Engineering
 * Data Splits
 * Modeling + AutoML
 * Training/Testing setup
 * Results
 * Deployment

### Contributing

If you feel we are missing something that you would want in a machine learning workflow, we're happy to listen to your input.
Follow the following steps:

 1. File an issue detailing what is lacking/problematic with the current spec, and ideally how you would propose to fix it.
 2. We may then ask you to file a Pull Request, in which case, please modify BOTH `model_provenance_example.json` and `README.md` to reflect the changes.

### Concerns

If you have questions or concerns, please file an issue on Github.

## Documentation


### Label Spec
- **binarize** - Boolean logic on how to binarize floating point values (e.g. ">0 & <1")
- **window** - Window of time with which to produce labels, include unit
- **offset** - Amount of time between each window
- **min_training_data** - Minimum amount of data recorded for each time-varying instance to generate labels for that instance
- **lead** - Time in the future to predict, include unit (e.g. "28 days")
- **label_function** - Label Function Object
- **reduce** - Function to reduce all values in prediction window for each
    instance/cutoff-time, produced by **label_function**, to a single value.
    Either a string representing one of the aggregations below:
      - "sum"
      - "mean"
      - "min"
      - "max"
      - "mode"
      - "last"
      - "first"
    Or, a function (wrapped in quotes) returning a single value from an array in a programming language:
      - {"python": "lambda x: np.sum(x)"}


#### Label Function Object
Key: programming language (e.g. **python**)
Value: Function writting in programming language wrapped in quotes.
    Function takes in the equivalent of DataFrame, and outputs an array
    representing the label for each row in the DataFrame
Python example:
{"python": "lambda x: x['Price'] - x['Cost']"}


### Data Splits

List of information about each split
Each split contains an object:
- **purpose** - Name of split
- **start_time** - Timestamp
- **end_time** - Timestamp
- **sampling** - Sampling Object

#### Sampling Object
- **strategy** - one of ["random", ]
- **by** - Column name to group by
- **n** - Number of elements to sample per element of "by" column
- **gap** - Amount of time to space between each sample of same element


### Training Setup
- **training** - Validation Type Object
- **tuning** - Validation Type Object
- **testing** -  Validation Type Object

#### Validation Type Object
- **data_split**: id of data split
- **cross_validation**: boolean

### Training Results
List for each training run. Generally models will be trained & evaluated
several times with different random seeds to measure stability.
Each element is an object:
- **random_seed** - int
- **threshold** - tuned decision threshold, float
- **precision** - float
- **recall** - float
- **fpr** - float
- **auc** - float
- **feature_importances** - List of sorted feature names by importance (first is most important)


### Deployment Information
- **model_path**: "/path/to/serialized_fitted_model.p"
- **feature_list_path**: "/path/to/serialized_feature_list.p"
- **deployment_executable**: "/path/to/deployment_executable.py"
- **data_fields_used**: Data Fields Used Object

#### Data Fields Used Object
Object listing the field names and types per table/entity used in feature engineering and fed into the machine learning model
- **entity_name**: [{"name": "FloatFieldName", "type": "float"},
                    {"name": "DateFieldName", "type": "datetime", "format": "YY/mm/dd"}]
