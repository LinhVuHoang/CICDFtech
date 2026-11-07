# Student Performance Prediction

An end-to-end Machine Learning project for predicting students' mathematics scores based on demographic information, parental education, lunch type, test preparation status, and reading and writing scores.

The project explores the complete ML engineering workflow, from data ingestion and preprocessing to model training, API development, containerization, CI/CD automation, Kubernetes deployment, Canary releases, and centralized secrets management.

**Project Status:** Experimental / Learning Project

## 1. Project Overview

This project aims to develop and deploy a regression-based Machine Learning application to predict students' mathematics performance.

Users can submit student information through a web interface. The application processes the input using a fitted preprocessing pipeline and generates a predicted mathematics score.

Beyond model development, the project explores practical ML engineering and DevOps concepts:

- Modular Machine Learning pipelines.
- Model training and hyperparameter optimization.
- REST API development with FastAPI.
- Docker-based application packaging.
- CI/CD automation using GitHub Actions.
- Kubernetes deployment and Canary releases with Argo Rollouts.
- Centralized secrets management using HashiCorp Vault and External Secrets Operator (ESO).

## 2. System Architecture

The project consists of four main workflows.

### 2.1 Machine Learning Pipeline

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

### 2.2 Prediction Pipeline

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

### 2.3 CI/CD and Deployment Pipeline

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

### 2.4 Secrets Management Architecture

```text
HashiCorp Vault
       |
       v
External Secrets Operator (ESO)
       |
       v
Kubernetes Secret
       |
       v
Application Pod
       |
       v
Environment Variable
       |
       v
    SECRET_KEY
```

External Secrets Operator retrieves secrets from HashiCorp Vault and synchronizes them into Kubernetes Secrets. The application consumes the required configuration through Kubernetes environment variables.

## 3. Machine Learning Workflow

### 3.1 Data Ingestion

The data ingestion component is responsible for loading and preparing the dataset.

Main operations:

- Load the student performance dataset from CSV.
- Store a copy of the original dataset.
- Split the dataset into training and testing sets.
- Use an 80/20 split with `random_state=42`.
- Save the resulting datasets as CSV files.

Generated files:

- `Raw_Data.csv`
- `Train_Data.csv`
- `Test_Data.csv`

### 3.2 Data Transformation

The preprocessing workflow uses Scikit-learn's `ColumnTransformer` and `Pipeline` to handle numerical and categorical features independently.

**Numerical features:**

- `reading_score`
- `writing_score`

Preprocessing steps:

1. Median imputation using `SimpleImputer`.
2. Feature scaling using `StandardScaler`.

**Categorical features:**

- `gender`
- `race_ethnicity`
- `parental_level_of_education`
- `lunch`
- `test_preparation_course`

Preprocessing steps:

1. Most frequent value imputation.
2. One-hot encoding using `OneHotEncoder`.
3. Feature scaling using `StandardScaler(with_mean=False)`.

The preprocessing object is fitted on the training dataset and reused to transform the testing dataset and new prediction inputs.

The target variable is `math_score`.

The fitted preprocessing pipeline is serialized using Joblib and saved as `Preprocessor.pkl`.

### 3.3 Model Training and Hyperparameter Optimization

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

The training workflow:

1. Initialize the candidate regression models.
2. Define hyperparameter search spaces.
3. Perform cross-validation using GridSearchCV.
4. Retrieve the best hyperparameters for each model.
5. Fit each model using the training dataset.
6. Evaluate model performance using R².
7. Select the model with the highest test-set R² score.
8. Save the selected model if its R² score is at least 0.6.

**Model Evaluation**

The project uses the R² score to compare regression model performance on the training and testing datasets.

The current implementation selects the best model using test-set R². Using a separate validation strategy for model selection and reserving the test set for final evaluation is a planned improvement.

### 3.4 Model Artifacts

The project stores the generated datasets and trained ML artifacts in the `Artifacts/` directory.

| File | Description |
|---|---|
| `Raw_Data.csv` | Original dataset |
| `Train_Data.csv` | Training dataset |
| `Test_Data.csv` | Testing dataset |
| `Model.pkl` | Selected trained regression model |
| `Preprocessor.pkl` | Fitted preprocessing pipeline |

Joblib is used to serialize and load the model and preprocessing objects.

## 4. API Development

The prediction application is developed using FastAPI.

**Available endpoints:**

| Method | Endpoint | Description |
|---|---|---|
| GET | `/` | Returns an API availability message |
| GET | `/health` | Returns application health status |
| GET | `/secure` | Checks SECRET_KEY configuration and returns a SHA-256 signature |
| POST | `/` | Accepts student information and returns a prediction through the HTML interface |

**Prediction input features:**

- Gender
- Race/ethnicity
- Parental level of education
- Lunch type
- Test preparation course
- Reading score
- Writing score

**Prediction workflow:**

1. Receive student information through an HTML form.
2. Convert the input into a Pandas DataFrame using `CustomData`.
3. Load the fitted preprocessing pipeline.
4. Transform the input features.
5. Load the trained regression model.
6. Generate the predicted mathematics score.
7. Render the prediction result using the HTML template.

The application uses Jinja2 templates and static CSS assets for its web interface.

## 5. CI/CD Automation

GitHub Actions automates application validation, Docker image building, and image delivery.

The configured workflow includes:

1. Trigger on pushes and pull requests targeting the `master` branch.
2. Set up Python 3.10.
3. Install project dependencies.
4. Start the FastAPI application.
5. Check API availability.
6. Send a sample prediction request.
7. Run Flake8 for linting.
8. Build a Docker image.
9. Push the image to Docker Hub.
10. Update the image tag in the Kubernetes Rollout manifest.
11. Commit and push the updated manifest.

**Docker Image Versioning**

Docker images are tagged using the Git commit SHA, allowing individual builds to be identified by their source revision.

**GitHub Actions Secrets**

Docker Hub credentials are configured through GitHub Actions Secrets rather than being hardcoded in the workflow.

The workflow also updates the Kubernetes manifest with the newly built image tag.

## 6. Kubernetes and Canary Deployment

The project uses Kubernetes and Argo Rollouts to define a Canary Deployment strategy.

**Deployment components:**

| File | Purpose |
|---|---|
| `rollout.yaml` | Defines the application Rollout and Canary strategy |
| `service.yaml` | Defines the stable application Service |
| `canary-service.yaml` | Defines the Canary application Service |
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

The Rollout configuration defines progressive traffic weights:

- 20% traffic to the Canary version.
- Health check through an AnalysisTemplate.
- 50% traffic to the Canary version.
- Another health check.
- 100% traffic to the new version.

**Health Analysis**

The AnalysisTemplate performs an HTTP health check against the Canary Service:

```text
http://student-performance-canary.default.svc.cluster.local:8000/health
```

The analysis succeeds when the endpoint returns HTTP status 200.

This provides a basic deployment health validation before progressing through the configured rollout steps.

## 7. Secrets Management

The project uses HashiCorp Vault and External Secrets Operator (ESO) to manage application secrets.

**Secrets management workflow:**

1. Store sensitive configuration in HashiCorp Vault.
2. Configure External Secrets Operator to retrieve secrets from Vault.
3. Synchronize the retrieved values into Kubernetes Secrets.
4. Inject the required secret into the application container through Kubernetes environment variables.
5. Access the configuration inside the application using `os.getenv()`.

For example, the FastAPI application retrieves its secret through:

```python
SECRET_KEY = os.getenv("SECRET_KEY")
```

The Kubernetes Rollout references the generated Kubernetes Secret:

```yaml
env:
  - name: SECRET_KEY
    valueFrom:
      secretKeyRef:
        name: student-performance-secret
        key: SECRET_KEY
```

**Technologies:**

- HashiCorp Vault
- External Secrets Operator (ESO)
- Kubernetes Secrets

This approach separates sensitive application configuration from source code and deployment manifests while providing centralized secrets management.

## 8. Technology Stack

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
| Secrets Management | HashiCorp Vault, External Secrets Operator |
| Code Quality | Flake8 |

## 9. Project Structure

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

## 10. Key Engineering Concepts

This project demonstrates practical experience with:

- Designing modular Machine Learning pipelines.
- Building numerical and categorical preprocessing workflows.
- Performing cross-validation and hyperparameter optimization.
- Comparing regression algorithms using evaluation metrics.
- Serializing and reusing trained ML artifacts.
- Developing prediction APIs with FastAPI.
- Packaging ML applications with Docker.
- Automating validation and image delivery with GitHub Actions.
- Managing Kubernetes application deployments.
- Exploring progressive delivery through Canary Deployment.
- Implementing deployment health checks with Argo Rollouts.
- Managing application secrets with HashiCorp Vault and External Secrets Operator.
- Injecting secrets into application containers through Kubernetes Secrets.

## 11. Future Improvements

Potential improvements include:

- Introduce a separate validation strategy for model selection.
- Optimize model loading and reuse in the API.
- Add automated unit and integration tests.
- Separate CI validation from CD deployment.
- Add model performance monitoring and prediction logging.
- Improve health checks to verify model readiness.
- Add deployment observability and automated rollback.
- Introduce model versioning and experiment tracking.
- Improve API input validation and error handling.
- Add automated secret rotation and synchronization validation.

## 12. Project Status

This is an experimental learning project developed to explore an end-to-end Machine Learning engineering and deployment workflow.

The project includes modular ML pipelines, a FastAPI prediction application, Docker-based packaging, GitHub Actions automation, Kubernetes deployment manifests, Canary rollout configuration, and centralized secrets management.

The deployment and security components demonstrate practical exploration of MLOps and DevOps concepts. The project is not presented as a production-ready ML platform.