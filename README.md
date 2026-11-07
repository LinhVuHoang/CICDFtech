# Student Performance Prediction

A Machine Learning project that predicts students' mathematics scores based on demographic information, parental education, lunch type, test preparation status, and reading and writing scores.

The project covers the end-to-end ML workflow, including data ingestion, preprocessing, model training, hyperparameter tuning, model evaluation, API development, containerization, CI/CD automation, and Canary Deployment with Kubernetes and Argo Rollouts.

**Project Status:** Experimental / Learning Project

## 1. Project Overview

This project aims to build and deploy a regression-based Machine Learning application for predicting students' mathematics performance.

The application accepts student information through a web interface, processes the input using a trained preprocessing pipeline, and generates a predicted mathematics score.

The project also explores deployment automation through Docker, GitHub Actions, Kubernetes, and Argo Rollouts.

## 2. System Architecture

The project consists of three main workflows:

### Machine Learning Pipeline

```text
Raw Dataset
    |
    v
Data Ingestion
    |
    v
Train / Test Split (80 / 20)
    |
    v
Data Transformation
    |
    v
Feature Engineering & Preprocessing
    |
    v
Model Training & Hyperparameter Tuning
    |
    v
Model Evaluation (R² Score)
    |
    v
Best Model Selection
    |
    v
Model.pkl + Preprocessor.pkl
```

### Prediction Pipeline

```text
User Input
    |
    v
FastAPI Application
    |
    v
CustomData
    |
    v
Pandas DataFrame
    |
    v
Load Preprocessor
    |
    v
Transform Input Features
    |
    v
Load Trained Model
    |
    v
Generate Prediction
    |
    v
Display Predicted Mathematics Score
```

### CI/CD and Deployment Pipeline

```text
GitHub Push / Pull Request
          |
          v
     GitHub Actions
          |
          v
 Install Dependencies
          |
          v
    Start FastAPI
          |
          v
      API Test
          |
          v
     Flake8 Lint
          |
          v
   Build Docker Image
          |
          v
    Push to Docker Hub
          |
          v
 Update Kubernetes Manifest
          |
          v
 Commit & Push Manifest
          |
          v
    Argo Rollouts
          |
          v
    Canary Deployment
     20% -> 50% -> 100%
```

## 3. Machine Learning Workflow

### 3.1 Data Ingestion

- Load the student performance dataset from CSV.
- Store a copy of the raw dataset.
- Split the dataset into training and testing sets.
- Use an 80/20 split with `random_state=42`.
- Save the resulting datasets as CSV files.

### 3.2 Data Transformation

The preprocessing pipeline uses Scikit-learn's `ColumnTransformer` and `Pipeline`.

**Numerical features:**
- `reading_score`
- `writing_score`

Preprocessing:
- Median imputation using `SimpleImputer`.
- Feature scaling using `StandardScaler`.

**Categorical features:**
- `gender`
- `race_ethnicity`
- `parental_level_of_education`
- `lunch`
- `test_preparation_course`

Preprocessing:
- Most frequent value imputation.
- One-hot encoding.
- Standard scaling with `with_mean=False`.

The preprocessing object is fitted on the training dataset and reused to transform the testing dataset and new prediction inputs.

The target variable is `math_score`.

### 3.3 Model Training and Evaluation

The project experiments with six regression algorithms:

| Model | Algorithm |
|---|---|
| Random Forest | RandomForestRegressor |
| Decision Tree | DecisionTreeRegressor |
| Gradient Boosting | GradientBoostingRegressor |
| Linear Regression | LinearRegression |
| CatBoost | CatBoostRegressor |
| AdaBoost | AdaBoostRegressor |

**Hyperparameter Optimization**

`GridSearchCV` with 3-fold cross-validation is used to search for suitable hyperparameters for each model.

**Model Evaluation**

- Evaluate model performance using the R² score.
- Compare the regression models.
- Select the model with the highest test-set R² score.
- Require a minimum R² score of 0.6 before saving the selected model.

The selected model and preprocessing object are serialized using Joblib.

### 3.4 Model Artifacts

Generated artifacts:

| File | Description |
|---|---|
| `Raw_Data.csv` | Original dataset |
| `Train_Data.csv` | Training dataset |
| `Test_Data.csv` | Testing dataset |
| `Model.pkl` | Selected trained regression model |
| `Preprocessor.pkl` | Fitted preprocessing pipeline |

## 4. API Development

The prediction application is built with FastAPI.

**Available endpoints:**

| Method | Endpoint | Description |
|---|---|---|
| GET | `/` | API availability message |
| GET | `/health` | Health check endpoint |
| GET | `/secure` | Checks SECRET_KEY configuration |
| POST | `/` | Accepts student information and returns a prediction through the HTML interface |

The prediction endpoint accepts:
- Gender
- Race/ethnicity
- Parental level of education
- Lunch type
- Test preparation course
- Reading score
- Writing score

The application converts the input into a Pandas DataFrame, applies the saved preprocessing pipeline, and generates a mathematics score prediction.

## 5. CI/CD Automation

GitHub Actions automates the application validation and Docker image delivery workflow.

The workflow includes:

1. Triggering on pushes and pull requests targeting the `master` branch.
2. Setting up Python 3.10.
3. Installing project dependencies.
4. Starting the FastAPI application.
5. Checking API availability.
6. Sending a sample prediction request.
7. Running Flake8 for linting.
8. Building a Docker image.
9. Pushing the image to Docker Hub.
10. Updating the image tag in the Kubernetes Rollout manifest.
11. Committing and pushing the updated manifest.

Docker image tags use the Git commit SHA to identify individual builds.

Docker Hub credentials are configured through GitHub Actions Secrets.

## 6. Kubernetes and Canary Deployment

The project uses Kubernetes and Argo Rollouts to define a Canary Deployment strategy.

**Deployment components:**

| File | Purpose |
|---|---|
| `rollout.yaml` | Defines the application Rollout and Canary strategy |
| `service.yaml` | Stable application Service |
| `canary-service.yaml` | Canary application Service |
| `analysis-template.yaml` | Defines deployment health analysis |

**Canary rollout strategy:**

```text
New Version Deployment
          |
          v
    20% Traffic
          |
          v
     Health Check
          |
          v
    50% Traffic
          |
          v
     Health Check
          |
          v
    100% Traffic
```

The AnalysisTemplate performs an HTTP health check against the Canary Service's `/health` endpoint.

The rollout proceeds through the configured traffic weights when the analysis succeeds.

## 7. Technology Stack

| Category | Technologies |
|---|---|
| Programming Language | Python |
| Data Processing | Pandas, NumPy |
| Machine Learning | Scikit-learn, CatBoost |
| Model Optimization | GridSearchCV |
| Model Serialization | Joblib |
| Backend | FastAPI, Uvicorn |
| Frontend | HTML, CSS, Jinja2 |
| Containerization | Docker |
| CI/CD | GitHub Actions |
| Container Orchestration | Kubernetes |
| Progressive Delivery | Argo Rollouts |
| Code Quality | Flake8 |

## 8. Project Structure

```text
Student-Performance-Prediction/
|
|-- .github/
|   |-- workflows/
|       |-- python-app.yml
|
|-- Artifacts/
|   |-- Model.pkl
|   |-- Preprocessor.pkl
|   |-- Raw_Data.csv
|   |-- Train_Data.csv
|   |-- Test_Data.csv
|
|-- Notebook_Experiments/
|   |-- Exploratory_data_analysis.ipynb
|   |-- Model_Training.ipynb
|   |-- Data/
|
|-- src/
|   |-- components/
|   |   |-- Data_ingestion.py
|   |   |-- Data_transformation.py
|   |   |-- Model_trainer.py
|   |
|   |-- pipeline/
|   |   |-- Training_pipeline.py
|   |   |-- Prediction_pipeline.py
|   |
|   |-- exception.py
|   |-- logger.py
|   |-- utils.py
|
|-- k8s/
|   |-- analysis-template.yaml
|   |-- canary-service.yaml
|   |-- rollout.yaml
|   |-- service.yaml
|
|-- static/
|   |-- styles.css
|
|-- templates/
|   |-- home.html
|
|-- app.py
|-- Dockerfile
|-- requirements.txt
|-- setup.py
|-- template.py
|-- README.md
```

## 9. Key Engineering Concepts

This project demonstrates practical experience with:

- Designing modular Machine Learning pipelines.
- Building numerical and categorical preprocessing workflows.
- Performing hyperparameter optimization and model comparison.
- Serializing and reusing trained ML artifacts.
- Developing prediction APIs with FastAPI.
- Packaging ML applications with Docker.
- Automating validation and image delivery with GitHub Actions.
- Defining Kubernetes deployment resources.
- Exploring progressive delivery through Canary Deployment and health analysis.

## 10. Future Improvements

Potential improvements include:

- Introduce a separate validation dataset for model selection.
- Optimize model loading and reuse in the API.
- Add automated unit and integration tests.
- Separate CI validation from CD deployment.
- Add model performance monitoring and prediction logging.
- Improve health checks to verify model readiness.
- Add automated rollback and deployment observability.
- Introduce model versioning and experiment tracking.

## 11. Project Status

This is an experimental learning project developed to explore an end-to-end Machine Learning deployment workflow.

The Kubernetes manifests define a Canary Deployment strategy, while the GitHub Actions workflow automates Docker image building and manifest updates.

The project is not presented as a production-ready ML platform.