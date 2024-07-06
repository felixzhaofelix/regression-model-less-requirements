# Maintaining a previously regression Model

Applying adaptive and corrective maintenance on a previously built and deployed regression ML model.

This is post explains the steps made to maintain the original code which is used in this 
[blog post](https://www.tekhnoal.com/regression-model.html) by schmidtbri

## Requirements

Python 3

## Installation 

The Makefile included with this project contains targets that help to automate several tasks.

To download the original non-functionning code, execute this command:

```bash
git clone https://github.com/schmidtbri/regression-model
```

```bash
git clone https://github.com/felixzhaofelix/regression-model-less-requirements
```

Then create a virtual environment and activate it:

```bash
# go into the project directory
cd regression-model

make venv

source venv/bin/activate
```

As of this writing, the following versions of the dependencies are known to work:
make sure they follow these versions in the requirements.in file (even after upgrading)
pydantic, fastapi and starlette are used to create the REST service and should remain at the same version
```bash
featuretools==0.24.0
scikit-learn>=1.3.2
tpot>=0.12.1
yellowbrick>=1.5
pydantic==1.10.13
fastapi==0.66.0
starlette==0.14.2

```
Use pip-tools to generate a new requirements.txt file from the requirements.in file:
```bash
pip install pip-tools

pip-compile requirements.in --upgrade
```

Then install the dependencies:
```bash
make dependencies
```

The requirements.txt file includes also the dependencies needed to train the model.

## Running the Unit Tests
First make the same upgrades with test dependencies:
```bash
pip-compile test_requirements.in --upgrade
```
Then install the test dependencies:
```bash
make test-dependencies
```

Then try to run the unit tests:
```bash
make test
```

There's gonna be an error with numpy in the transformer.py file
in this case, open the file and change this line:
(This is incoherency with versions of numpy used in the requirements.txt file)
```python
# import numpy as np
# from numpy import bool_, inf, int, float
```
to this:
```python
# import numpy as np
# from numpy import bool_, inf, int_, float_
```

Then try to run the unit tests again:
```bash
make test 
```

Then there's gonna be three tests that pass and two that fail among all five:
If it's the first two, follow these instructions to retrain the model:

If it's not, it's still useful to retrain the model with the current the version of packages
(especially scikit-learn) as the model should not be serialized and deserilized with different versions of packages

1-Find these files in the project directory:
(These files have been newly added at the commit in November 2023)
```bash
#regression-model/data_exploration.ipynb
#regression-model/data_preparation.ipynb
#regression-model/model_training.ipynb
#regression-model/model_validation.ipynb
#regression-model/insurance.csv
```
2-Open them in the same order after having configured a Jupyter Notebook kernel in the virtual environment
and run all the cells in each file sequentially and a new file will be created in the project directory:
```bash
regression-model/model2023.joblib
```

2.1 You can also rerun the last cell in regression-model/model_training.ipynb to create the model in a different name
like this:
```Jupyter Notebook
joblib.dump(model, "yourModelName.joblib")
```
2.2 Here's the code to load the model in the service in case some modifications are made to the model:
Open the file: regression-model/insurance_charges_model/prediction/model.py
and modlify the code in the constructor like this:
```python
# def __init__(self):
#     """Class constructor that loads and deserializes the model parameters.
# 
#     .. note::
#         The trained model parameters are loaded from the "model_files" directory.
# 
#     """
#     dir_path = os.path.dirname(os.path.dirname(os.path.realpath(__file__)))
#     with open(os.path.join(dir_path, "model_files", "1", "yourModelName.joblib"), 'rb') as file:
#         self._svm_model = joblib.load(file)
```

3-Then run the unit tests again:
```bash
make test
```

clean up the unit tests
```bash
make clean-test
```

## Running the Service

To start the service locally, execute these commands:

```bash
uvicorn rest_model_service.main:app --reload
```

## Generating an OpenAPI Specification

To generate the OpenAPI spec file for the REST service that hosts the model, execute these commands:

```bash
export PYTHONPATH=./
generate_openapi --output_file=service_contract.yaml
```

## Docker

To build a docker image for the service, run this command:

```bash
docker build -t insurance_charges_model:0.1.0 .
```

To run the image, execute this command:

```bash
docker run -d -p 80:80 insurance_charges_model:0.1.0
```

To watch the logs coming from the image, execute this command:

```bash
docker logs $(docker ps -lq)
```

To stop the docker image, execute this command:

```bash
docker kill $(docker ps -lq)
```
