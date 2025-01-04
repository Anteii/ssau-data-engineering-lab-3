# Data Engineering. Practice #3. 
[![en](https://img.shields.io/badge/lang-en-red.svg)](https://github.com/Anteii/ssau-data-engineering-lab-3/blob/main/README.md)
[![pt-br](https://img.shields.io/badge/lang-ru--ru-green.svg)](https://github.com/Anteii/ssau-data-engineering-lab-3/blob/main/README.ru-ru.md)


# Training pipeline

Models were trained on iris dataset from scikit-learn.

Train and validation sets were formed using code below

    X, y = datasets.load_iris(return_X_y=True, as_frame=True)

    X_train, X_test, y_train, y_test = train_test_split(
        X, y, test_size=0.3, random_state=42
    )
    X_test, X_val, y_test, y_val = train_test_split(
        X, y, test_size=0.5, random_state=42
    )


To ensure reproducibility random_state is fixed in `train_model.py` and `validate_model.py` scripts.

Model training steps:
1. Wait for new config file in `configs` folder. (one config - one run - one model)
2. Run train script
    1. For each config
        1. Parse model's parameters and metrics names
        2. Create and fit model
        3. Evaluate metrics against X_val split
        4. Upload model, parameters and metrics values to MLFlow

<p align="center">
  <img width="800" height="350" src="https://github.com/Anteii/ssau-data-engineering-lab-3/blob/main/screenshots/train-model-airflow.png"/>
</p>
<p style="text-align: center;">Airflow model training pipline</p>

<p align="center">
  <img width="800" height="300" src="https://github.com/Anteii/ssau-data-engineering-lab-3/blob/main/screenshots/mlflow-experiment-ui.png"/>
</p>
<p style="text-align: center;">MLFlow experiment </p>

Encountered issues:
1. MLFlow server silently failed to connect to S3 storage and failed to display experiment's artifacts. Attempts to get artifacts directly from python API also failed. Solved by installing boto3 in container with MLFlow server.
 

# Validation pipeline

Validation (startup on timer):
1. For each config in `config` folder
    1. Read model's name and parameters; run's parameters and evaluated metrics names
    2. Search for experiment by name
    3. Get all runs by experiment-id
    4. Retrieve an exact run by matching run's parameters
    5. Get artifact of this run
    6. Evaluate model against X_test split and <b>log results in the same run with `_test` suffix</b>
    7. Find best run by target metric (metric name with `_test` suffix).If there's no best we select current run.
    8. Change model's stage of this run to `Production`

<p align="center">
  <img width="800" height="300" src="https://github.com/Anteii/ssau-data-engineering-lab-3/blob/main/screenshots/validate-model-airflow.png">
</p>
<p style="text-align: center;">Validation pipeline Airflow</p>

<p align="center">
  <img width="800" height="250" src="https://github.com/Anteii/ssau-data-engineering-lab-3/blob/main/screenshots/mlflow-stage-ui.png"/>
</p>
<p style="text-align: center;">Staging MLFlow </p>

Encountered issues:
1. Got `Bad Credentials` error when fetching artifact with MLFlow Python API (don't remember solution).
2. Unintuitive MLFlow parameter `artifact_path`. If `artifact_path` is set to "", then MLFlow place them in `{run_id}\artifacts`. If it's set to something, then MLFlow place artifacts to `{run_id}\<specified_path>`