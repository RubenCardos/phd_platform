# Automated Experimentation Platform

## Table of contents

1. [Project Structure](#project-structure)
1. [How to use](#how-to-use)

## Project Structure
The repository is organized as follows:


```bash
src
├── compose.yml  # Docker Compose file to orchestrate the complete deployment of the platform
├── mlflow/
    ├── Dockerfile # Dockerfile to build amd Docker image for MLFlow server
    ├── requirements.txt   # Requirements file for MLFLow server
    └── Dockerfile   # Dockerfile to create custom Airflow image, yo include git and all requeriments
├── License  # License File, MIT license
├── README.md  # Readme file 
└── gitignore.yml # Git ignore file, to ignore some non-needed file in the reposotory
```

## How To Use

To deploy the platform, execute the following command in the same directory atTo deploy the platform, run the command shown below where the compose.yml file is located.

```bash
docker compose up
```

This command will deploy all the Docker containers needed to use the platform, which mainly consists of Airflow as the orchestrator (made up of different containers to deploy all its functionality) and MLFlow as the Model/Experiment Registry. 

Airflow Docker image is custom, so some additional dependecies are installed, like Git or MLFlow.

### Airflow

To use airflow:

* User interface could be open at http://localhost:8080/ and login with user/password airflow/airflow
* Dags should be added at dags folder, create when docker compose up cammands was executed
* For more info about how to create dags check Airflow documentation: https://airflow.apache.org/docs/apache-airflow/stable/tutorial/index.html

### MLFlow

To user MLFlow:

* User interface could be open at http://127.0.0.1:5000/ ,no login its requeired
* To create experiments and runs use the MLFLow Python SDK 