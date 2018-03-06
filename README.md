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


### Prediction Engineering
- **prediction_window** - Window of time with which to produce labels, include unit
- **min_training_data** - Minimum amount of data recorded for each time-varying instance to generate labels for that instance
- **lead** - Time in the future to predict, include unit (e.g. "28 days")
- **labeling_function** - Path to executable labeling function

### Feature Engineering
List of feature engineering methods used

- **method** - Name of method to use for (automated) feature engineering, such as Deep Feature Synthesis
***method-specific parameters***
- **training_window** - Amount of historical data to use to compute features for each instance
- **aggregate_primitives** - List of strings of Featuretools aggregation primitives
- **transform_primitives** - List of strings of Featuretools transform primitives
- **ignore_variables** - Lists of names of columns per entity to ignore when building features
- **feature_selection** - method name and hyperparameters associated with feature selection, if any

### Modeling
Describes the machine-learning methods to search through and the AutoML method to use

- **methods** - List of machine learning methods, each containing a "method" name and "hyperparameter_options" path to a separate JSON
- **budget** - Resources to allocate in terms of either compute time or number of models to try
- **automl_method** - path to another JSON spec file detailing the AutoML method (such as ATM, AutoSKLearn, etc)
- **cost_function** - path to an executable script that computes a cost given true and predicted labels, and access to the underlying data

### Data Splits

List of information about each split
Each split contains an object:
- **id** - Name of split
- **start_time** - Timestamp
- **end_time** - Timestamp
- **label_search_parameters** - Label Search Parameters Object

#### Label Search Parameters Object
- **offset** - amount of time between successive labeling windows in search
- **strategy** - one of ["random", ]
- **examples_per_instance** - Number of elements to sample per element of "by" column
- **gap** - Amount of time to space between each sample of same element


### Training Setup
- **training** - Validation Type Object
- **tuning** - Validation Type Object
- **testing** -  Validation Type Object

#### Validation Type Object
- **data_split**: id of data split
- **validation_method**: Path to validation spec JSON file

### Results
Keys are the data split id, each value is a list for each trial run. Generally models will be trained & evaluated
several times with different random seeds to measure stability.
Each element for each trial run is an object:
- **random_seed** - int
- **threshold** - tuned decision threshold, float
- **precision** - float
- **recall** - float
- **fpr** - float
- **auc** - float


### Deployment
- **deployment_executable** - Path to an file that can generate predictions
- **deployment_parameters** -  Deployment Parameters object
- **data_fields_used** - Data Fields Used Object
- **integration_and_validation** - Integration And Validation Object

#### Deployment Parameters Object
- **feature_list_path** -  Path to a file that can be used by the feature computation method to compute new feature matrices
- **model_path** -  Path to serialized fitted model that can be loaded in memory and used to make predictions
- **threshold** - Threshold used to binarize predictions

#### Integration And Validation Object
- **data_fields_used** - Data Fields Used Object
- **expected_feature_value_ranges** - Expected Feature Value Ranges Object

##### Expected Feature Value Ranges Object
Object listing the expected ranges of each feature in the feature matrix
Keys are feature names values are objects containing:
- **min** - min value
- **max** - max value

##### Data Fields Used Object
Object listing the field names per table/entity used in feature engineering and fed into the machine learning model
- **data_fields_object**: {"entity_name": ["Field1", "Field2", "Field3"]}
