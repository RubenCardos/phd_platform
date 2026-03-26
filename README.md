# Automated Experimentation Platform

## Table of contents

1. [Project Structure](#project-structure)
2. [How to use](#how-to-use)
3. [Other MLOps platforms](#other-mlops-platforms)
4. [Prerequisites](#rrerequisites)
5. [End-to-end real case scenario](#e2e-real-case-scenario)

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

## Platform Architecture

Following image presents a schematic representation of the platform architecture and illustrates the interaction between modules within the automated experimentation ecosystem.

![Graphical representation of the platform architecture](doc/images/PlatformArq.PNG)

### Services orchestrated with Docker Compose

The platform orchestrates the following services using Docker Compose:

1. **Airflow**:
* Description: A workflow orchestrator for complex workflows, it is a custom image with several dependencies already included, such as Git and MLFlow. 
* Exposed Port: 8080
* Build: Built from the Dockerfile located in the root directory.
* Dependencies: This service consists of multiple services/containers, which make up the various common modules of Airflow:
  * **postgres**: Database container that stores metadata, DAG states, and task information.
  * **flower**: Provides a web-based monitoring tool for Celery (as integrated library within Airflow) workers and task queues.
  * **redis**: Acts as a message broker for Celery (for Celery Executor), enabling communication between scheduler and workers.
  * **airflow-webserver**: Hosts the web UI where users can monitor, trigger, and manage workflows (DAGs).
  * **airflow-scheduler**: Responsible for scheduling tasks and triggering DAG runs based on defined intervals and dependencies.
  * **airflow-worker**: Executes the tasks assigned by the scheduler
  * **airflow-triggerer**: Handles deferred and asynchronous tasks, improving efficiency for long-running operations.
  * **airflow-init**: Initializes the environment, sets up the database, and creates the initial user.
  * **airflow-cli**: A lightweight utility container used to run Airflow CLI commands interactively or on demand, sharing the same environment as other Airflow services.

2. **Mlflow**:
* Description: Experiment tracking and evaluation store. Allos to easily stored results and compare results
* Exposed Port: 5050
* Build: Built from the Dockerfile located in the /mlflow directory.


## How To Use

To deploy the platform, execute the following command where the compose.yml file is located.

```bash
docker-compose up --build -d
```

If when execute the docker compose command the return is *no configuration file provided: not found* probabily path is not correct , execute *cd src* command to change path to *src* directory, where compose.yml file is located

This command reads the Compose specification and instantiates all required services, creating and orchestrating the corresponding containers, parameter description:

* **--build**: Rebuild the Docker images being used in the compose.yml file. If Docker images already exists in your Docker instance, you can remove this parameter, in case of doubt, it is recommended that you always use this option
* **-d**: Unattended mode, the terminal is not locked using the standard Docker exit. You can disable this option if you want to view the Docker standard output log, but we recommend viewing the log from Docker Desktop or using the *docker container logs* command.

This command will deploy all the Docker containers needed to use the platform, which mainly consists of Airflow as the orchestrator (made up of different containers to deploy all its functionality) and MLFlow as the Model/Experiment Registry. 

Experiments could be orchestrated through Apache Airflow using DAGs. The recommended execution strategy consists of dynamically retrieving experiment definitions from an external Git-based repository, executing the defined workflows, and optionally removing the downloaded repository afterwards. This approach ensures that the most recent version of the experimental code is always used while avoiding unnecessary storage consumption within the platform.

Airflow Docker image is custom, so some additional dependecies are installed, like Git or MLFlow.

After the platform is deployed, four new folder will appear:

* **config**: For override Airflow confing, it is not recommended to use it.
* **dags**: For Aiflow dags, **put all your dags and Python Scripts here**. (1)
* **logs**: Log generated by Airflow.
* **plugins**: For Airflow pluggins, it is not recommended to use it.

Once the platform is deployed, the neeeded Docker containers are created, the following image shown the containers deployed

![Graphical representation of the platform architecture](doc/images/Docker_Plataforma.PNG)

### Airflow

To use airflow:

* User interface could be open at http://localhost:8080/ and login with user/password airflow/airflow
* Dags should be added at dags folder, create when docker compose up cammands was executed
* For more info about how to create dags check Airflow documentation: https://airflow.apache.org/docs/apache-airflow/stable/tutorial/index.html

### MLFlow

To user MLFlow:

* User interface could be open at http://127.0.0.1:5000/ ,no login its requeired
* To create experiments and runs use the MLFLow Python SDK 

### Examples of use

If you want examples of real-world use of the platform, such as orchestrating an experiment in a workflow/DAGs, Python scripts, and connecting to MLFlow to send and save results, download the following repository https://github.com/RubenCardos/phd_workflows in the “DAGs” (1) directory, which contains several examples of real-world use of the platform.

* elastic_patterns_experiment_1.py : Experiment to classify handwritten numbers, MNIST dataset, and storage results in MLFlow.
* elastic_patterns_experiment_1_versioning.py : Same as experiment 1, but using a code repository to storage the source code of experiment since the platform have a source version control (git) tool.
* elastic_patterns_experiment_2: Experiment to classify brest cancer samples and storage results to MLFlow

For this examples its recommends to create two pools in airflow, to create a pool in Airflow UI go to Admin > Pools and add a new pool, recommended pools are:

* experiment_1_pool with two sloots
* experiment_2_pool with six sloots

## Comparision with other MLOps Platforms

| Feature                          | Proposed Platform | Kubeflow   | MLflow Recipes |
|----------------------------------|------------------|------------|----------------|
| Lightweight deployment           | Yes              | No         | Yes            |
| Combinatorial experiment support | Yes              | Partially  | No             |
| Parallel experimentation (research-oriented) | Yes | Partially | No |
| Reproducibility (end-to-end)     | Yes              | Yes        | Partially      |
| Domain-specific                  | No               | No         | No             |

Unlike general-purpose MLOps platforms, the proposed system is specifically designed to support research-oriented experimentation involving combinatorial parameter spaces and systematic multi-run evaluation.

Other MLops platform: 

* *Kubeflow* is an MLOps platform for scalable machine learning workflows on Kubernetes, supporting pipelines, distributed training, and cloud integration. However, it requires a Kubernetes cluster and significant configuration, which can add complexity in lightweight research environments.  \\

* *MLflow Recipes* (previously known as MLflow Pipelines) focus on structuring machine learning workflows and tracking experiments, provides experiment logging, model versioning, and reproducibility support \cite{zaharia2018accelerating}, but mainly offers templates and tracking, relying on external tools for orchestration and large-scale execution.

## Prerequisites

The only requirement for deploying and using the platform is that Docker be installed, we also recommend installing the Docker Desktop tool. For more information on how to install the needed tools, please visit the folling links:

* For Docker: https://docs.docker.com/engine/install/

* For Docker Desktop: https://docs.docker.com/desktop/setup/install/windows-install/

Please note that installing Docker on Windows may require the WSL component. For more information, please visit to the following links
https://learn.microsoft.com/es-es/windows/wsl/install  https://docs.docker.com/desktop/features/wsl/

## End-to-end real case scenario

To show the capabilities of the propose platform, a real case end-to-end scenario is gping to be show, with a step-by-step guide:

1. Download the repository, navigate to the *phd_platform* directory, and then to the *src* (phd_platform/src) directory.

2. Check that Docker is running, for example, open Docker Desktop and wait until "Engine Running" on a green backgound is show on the left botton corner.
   
3. Execute the command *docker-compose up --build -d* and wait until plafotm is deployed, you can verify this by accessing the Airflow and MLFlow interfaces; if both tools are displayed, the platform is deployed. Check section [How to use](#how-to-use) for more info.

4.  Navigate to the *dags* directory (phd_platform/src/dags) and download this repository https://github.com/RubenCardos/phd_workflows, path within the platorm should be *phd_platform/src/dags/phd_workflows*. The repository contains the following experiments, implemented as Airflow dags: 
      
      * Experiment 1: A experiment with Elastic Pattern, Elastic Patterns are created on the fly, to caracterize/classify MNIST Dataset, this experiment have two versions: 
  
        * Dag *elastic_patterns_experiment_1_versioning* (path to dag: phd_platform/src/dags/phd_workflows/src/elastic_patterns_experiment_1_versioning.py) this version have Git integration, so the experient is donwloaded form a Bitbucket repository, execute, and deleted.

        * Dag *elastic_patterns_experiment_1* (path to dag: phd_platform/src/dags/phd_workflows/src/elastic_patterns_experiment_1.py), same experiment withut Git integration.
      
      * Experiment 2: A experiment with Elastic Pattern, Elastic Patterns are created on the fly, to caracterize/classify potential carcinogens samples of Wisconsin Breast Cancer dataset. Dag for this experiment is *elastic_patterns_experiment_2* (path to dag: phd_platform/src/dags/phd_workflows/src/elastic_patterns_experiment_2.py)
  
5. Enabled Airflow pools, pools determine the ability to run experiments (DAGs) in parallel; depending on the memory consumption of the experiments, this configuration can be increased or decreased. To enabled pools, go to Airflow interface http://localhost:8080/ (with user/password airflow/airflow), Admin > Pools, and create two pools: *experiment_1_pool* with two sloots and *experiment_2_pool* with six sloots (recomended values, you can increase to execute more experiment runs in parallel). Below are screenshots of the steps described in the Airflow interface
  
  * Go to Admins > Pools

  ![Airflow Pools](doc/images/AirflowPools.PNG)

  * Create a Pool

  ![Airflow Pools2](doc/images/AirflowPools2.PNG)

  * Insert a Pool name (for example, experiment_1_pool), and a number of Slots(for example, two), then Save it.
  
  ![Airflow Pools3](doc/images/AirflowPools3.PNG)

6. Go to Experiment 2, in Airflow Home, go to the dag *elastic_patterns_experiment_2* and click on it .You can click on the Airflow logo in the upper-left corner of the screen to return to the home screen.

![Experiment Dag](doc/images/ExperimentDag.PNG)

7. Inside the Airflow dag, you can run the experiment by clicking the button with the play icon in the upper-right corner. Once you do so, a new run will appear in the DAG's execution window, located in the middle-left part of the screen. In this window, you can now view information such as previous runs, run times, run logs, etc.

![Experiment Dag trigger](doc/images/ExperimentDagTrigger.PNG)

8. Once the run has started, you can view its progress in the Runs window; a vertical gray bar typically appears and grows upward. Clicking on this bar will cause the Runs window to display the current run. Once the entire experiment has finished running, each tested configuration is saved as a run in MLflow. For this experiment, a task (a node in a DAG) is generated for each configuration based on the three factors that make up each possible configuration, resulting in 84 tasks.

9. Go to Mlflow inteface http://localhost:5000/ and select the experiment *elastic_patterns_experiment_2* (same name that the Airflow dag).

![MLflowHome](doc/images/MLFlow1.PNG)

10. The list of runs (a configuration of the three factors) is show, now some analysis could be easily done. The set of runs is displayed in a table. This table shows information such as the run's registration time (the duration of the MLflow context), which should not be confused with the experiment's execution time, input parameters, metrics obtained, etc.

![MLflowRuns](doc/images/MLFlow2.PNG)

11. Select all the runs, with the checkbox on the left side of the table showing the list of runs, this will select all runs, or you can choose only the runs you want, and click on *Compare* button

![MLflowRunsCompare](doc/images/MLFlow3.PNG)

12. An initial overview of the input parameters and available metrics is provided, allowing users to switch between different types of graphs and different sets of parameters and metrics. In the context of this experiment, *Experiment 2*, it can be seen that in terms of *accuracy*, the experiment configuration that yields the best results is the *Inverse* deformation method, using the *Mean* ggregation strategy, as indicated by the darker red lines.

![MLflowRunsCompareREsult](doc/images/MLFlow4.PNG)