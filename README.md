# MLOps Pipeline for Screentime Analysis using Apache Airflow

This project demonstrates how to build an **MLOps pipeline** using **Apache Airflow** to automate the preprocessing, training, and evaluation of a machine learning model for predicting app usage based on screentime data. The pipeline is designed to be scalable, reproducible, and efficient, showcasing the power of **MLOps** in real-world machine learning workflows.

---

## Table of Contents
1. [Overview](#overview)
2. [Dataset](#dataset)
3. [Pipeline Workflow](#pipeline-workflow)
4. [Installation](#installation)
5. [Usage](#usage)
6. [DAG Structure](#dag-structure)
7. [Results](#results)
8. [Contributing](#contributing)
9. [License](#license)

---

## Overview

The goal of this project is to automate the process of analyzing screentime data using **Apache Airflow**. The pipeline performs the following tasks:
1. **Data Preprocessing**: Cleans, transforms, and scales the dataset.
2. **Model Training**: Trains a **Random Forest Regressor** to predict app usage.
3. **Automation**: Schedules and automates the workflow using Airflow DAGs.

This project is a great example of how **MLOps** can streamline machine learning workflows, making them more efficient and scalable.

---

## Dataset

The dataset contains app usage behavior with the following columns:
- **Date**: The day of usage.
- **App**: The name of the app (e.g., Instagram, WhatsApp).
- **Usage (minutes)**: Time spent on the app.
- **Notifications**: Number of alerts received.
- **Times Opened**: Number of times the app was launched.

Download the dataset from [here](#).

---

## Pipeline Workflow

### 1. Data Preprocessing
- Load the dataset and check for missing values and duplicates.
- Convert the `Date` column to datetime and extract features like `DayOfWeek` and `Month`.
- Encode the categorical `App` column using **one-hot encoding**.
- Scale numerical features (`Notifications` and `Times Opened`) using **MinMaxScaler**.
- Perform feature engineering:
  - Create a lagged feature: `Previous_Day_Usage`.
  - Create an interaction feature: `Notifications_x_TimesOpened`.
- Save the preprocessed data to a new CSV file.

### 2. Model Training
- Split the preprocessed data into training and testing sets.
- Train a **Random Forest Regressor** model.
- Evaluate the model using **Mean Absolute Error (MAE)**.

### 3. Automation with Apache Airflow
- Define an Airflow **DAG** to automate the preprocessing task.
- Schedule the DAG to run daily.

---

## Installation

### Prerequisites
- Python 3.8+
- Apache Airflow
- Pandas
- Scikit-learn

### Steps
1. Clone the repository:
   ```bash
   git clone https://github.com/your-username/mlops-airflow-screentime.git
   cd mlops-airflow-screentime
   ```

2. Install the required Python packages:
   ```bash
   pip install -r requirements.txt
   ```

3. Install Apache Airflow:
   ```bash
   pip install apache-airflow
   ```

4. Initialize the Airflow database:
   ```bash
   airflow db init
   ```

5. Start the Airflow webserver:
   ```bash
   airflow webserver --port 8080
   ```

6. Start the Airflow scheduler (in a new terminal):
   ```bash
   airflow scheduler
   ```

7. Access the Airflow UI at `http://localhost:8080`.

---

## Usage

1. Place the dataset (`screentime_analysis.csv`) in the project directory.
2. Define the DAG in the `dags/` folder (see [DAG Structure](#dag-structure)).
3. Enable the `data_preprocessing` DAG in the Airflow UI.
4. Manually trigger the DAG to execute the preprocessing task.
5. Check the output file (`preprocessed_screentime_analysis.csv`) for the preprocessed data.

---

## DAG Structure

The Airflow DAG is defined as follows:

```python
from airflow import DAG
from airflow.operators.python import PythonOperator
from datetime import datetime
import pandas as pd
from sklearn.preprocessing import MinMaxScaler

def preprocess_data():
    file_path = 'screentime_analysis.csv'
    data = pd.read_csv(file_path)

    data['Date'] = pd.to_datetime(data['Date'])
    data['DayOfWeek'] = data['Date'].dt.dayofweek
    data['Month'] = data['Date'].dt.month

    data = data.drop(columns=['Date'])

    data = pd.get_dummies(data, columns=['App'], drop_first=True)

    scaler = MinMaxScaler()
    data[['Notifications', 'Times Opened']] = scaler.fit_transform(data[['Notifications', 'Times Opened']])

    preprocessed_path = 'preprocessed_screentime_analysis.csv'
    data.to_csv(preprocessed_path, index=False)
    print(f"Preprocessed data saved to {preprocessed_path}")

dag = DAG(
    dag_id='data_preprocessing',
    schedule_interval='@daily',
    start_date=datetime(2025, 1, 1),
    catchup=False,
)

preprocess_task = PythonOperator(
    task_id='preprocess',
    python_callable=preprocess_data,
    dag=dag,
)
```

---

## Results

- **Mean Absolute Error (MAE)**: `15.3985`
  - This indicates that, on average, the model's predicted screentime differs from the actual screentime by approximately **15.4 minutes**.

- **Preprocessed Data**:
  - Saved to `preprocessed_screentime_analysis.csv`.

---

## Contributing

Contributions are welcome! If you'd like to improve this project, please follow these steps:
1. Fork the repository.
2. Create a new branch (`git checkout -b feature/YourFeature`).
3. Commit your changes (`git commit -m 'Add some feature'`).
4. Push to the branch (`git push origin feature/YourFeature`).
5. Open a pull request.

---

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

---

## Acknowledgments
- [Apache Airflow](https://airflow.apache.org/) for workflow orchestration.
- [Scikit-learn](https://scikit-learn.org/) for machine learning tools.
- [Pandas](https://pandas.pydata.org/) for data manipulation.

---

Feel free to reach out if you have any questions or suggestions! 🚀
