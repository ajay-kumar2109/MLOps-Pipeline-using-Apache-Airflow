# MLOps-Pipeline-using-Apache-Airflow
MLOps Pipeline using Apache Airflow: Overview
The given dataset contains app usage behaviour with five key columns:

Date (usage day)
App (e.g., Instagram, WhatsApp)
Usage (minutes spent)
Notifications (alerts received)
and Times Opened (app launches).
Download the dataset from here.

The goal of this pipeline is to streamline the process of analyzing screentime data by automating its preprocessing and utilizing machine learning to predict app usage. To ensure seamless execution, we will design an Airflow DAG to schedule and automate daily data preprocessing tasks to support a robust and scalable workflow.

Building an MLOps Pipeline using Apache Airflow
Let’s start building an MLOps pipeline using Apache Airflow with the necessary data preprocessing steps:

import pandas as pd
from sklearn.preprocessing import MinMaxScaler
​
# load the dataset
data = pd.read_csv('screentime_analysis.csv')
​
# check for missing values and duplicates
print(data.isnull().sum())
print(data.duplicated().sum())
​
# convert Date column to datetime and extract features
data['Date'] = pd.to_datetime(data['Date'])
data['DayOfWeek'] = data['Date'].dt.dayofweek
data['Month'] = data['Date'].dt.month
​

# encode the categorical 'App' column using one-hot encoding
data = pd.get_dummies(data, columns=['App'], drop_first=True)
​
# scale numerical features using MinMaxScaler
scaler = MinMaxScaler()
data[['Notifications', 'Times Opened']] = scaler.fit_transform(data[['Notifications', 'Times Opened']])
​
# feature engineering
data['Previous_Day_Usage'] = data['Usage (minutes)'].shift(1)
data['Notifications_x_TimesOpened'] = data['Notifications'] * data['Times Opened']
​
# save the preprocessed data to a file
data.to_csv('preprocessed_screentime_analysis.csv', index=False)
The above code performs data preprocessing to prepare the screentime dataset for machine learning. It begins by loading the dataset and ensuring data quality through checks for missing values and duplicates. It then processes the Date column to extract useful temporal features like DayOfWeek and Month. The App column is transformed using one-hot encoding to convert it into a numeric format.

The process scales numerical columns, such as Notifications and Times Opened, using MinMaxScaler to ensure uniformity. Feature engineering creates lagged (Previous_Day_Usage) and interaction (Notifications_x_TimesOpened) features to enhance predictive power.

Training the Model
Next, after preprocessing, we will train a Random Forest model to predict app usage:

from sklearn.model_selection import train_test_split
from sklearn.ensemble import RandomForestRegressor
from sklearn.metrics import mean_absolute_error

# split data into features and target variable
X = data.drop(columns=['Usage (minutes)', 'Date'])
y = data['Usage (minutes)']

# train-test split
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# train the model
model = RandomForestRegressor(random_state=42)
model.fit(X_train, y_train)

# evaluate the model
predictions = model.predict(X_test)
mae = mean_absolute_error(y_test, predictions)
print(f'Mean Absolute Error: {mae}')
from sklearn.model_selection import train_test_split
from sklearn.ensemble import RandomForestRegressor
from sklearn.metrics import mean_absolute_error
​
# split data into features and target variable
X = data.drop(columns=['Usage (minutes)', 'Date'])
y = data['Usage (minutes)']
​
# train-test split
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)
​
# train the model
model = RandomForestRegressor(random_state=42)
model.fit(X_train, y_train)
​
# evaluate the model
predictions = model.predict(X_test)
mae = mean_absolute_error(y_test, predictions)
print(f'Mean Absolute Error: {mae}')
Mean Absolute Error: 15.398500000000002
In the above code, we are splitting the preprocessed data into training and testing sets, training a Random Forest Regressor model, and evaluating its performance.

First, the process separates the target variable (Usage (minutes)) from the features and performs an 80-20 train-test split. The training data is used to train the RandomForestRegressor model. After completing the training, the model generates predictions on the test set, and the Mean Absolute Error (MAE) metric quantifies the average difference between the predicted and actual values to assess performance.

The Mean Absolute Error (MAE) of 15.3985 indicates that, on average, the model’s predicted screentime differs from the actual screentime by approximately 15.4 minutes. This gives a measure of the model’s predictive accuracy, showing that while the model performs reasonably well, there is still room for improvement in reducing this error to make predictions more precise.

Automating Preprocessing with a Pipeline using Apache Airflow
Apache Airflow enables the automation of tasks using Directed Acyclic Graphs (DAGs). Here, we will use a DAG to build a pipeline to preprocess data daily. First, install Apache Airflow:

pip install apache-airflow
Now, we will define the DAG and task to build the pipeline:

from airflow import DAG
from airflow.operators.python import PythonOperator
from datetime import datetime

# define the data preprocessing function
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

# define the DAG
dag = DAG(
    dag_id='data_preprocessing',
    schedule_interval='@daily',
    start_date=datetime(2025, 1, 1),
    catchup=False,
)

# define the task
preprocess_task = PythonOperator(
    task_id='preprocess',
    python_callable=preprocess_data,
    dag=dag,
)
from airflow import DAG
from airflow.operators.python import PythonOperator
from datetime import datetime
​
# define the data preprocessing function
def preprocess_data():
    file_path = 'screentime_analysis.csv'
    data = pd.read_csv(file_path)
​
    data['Date'] = pd.to_datetime(data['Date'])
    data['DayOfWeek'] = data['Date'].dt.dayofweek
    data['Month'] = data['Date'].dt.month
​
    data = data.drop(columns=['Date'])

    data = pd.get_dummies(data, columns=['App'], drop_first=True)
    scaler = MinMaxScaler()
    data[['Notifications', 'Times Opened']] = scaler.fit_transform(data[['Notifications', 'Times Opened']])

    preprocessed_path = 'preprocessed_screentime_analysis.csv'
    data.to_csv(preprocessed_path, index=False)
    print(f"Preprocessed data saved to {preprocessed_path}")
# define the DAG
dag = DAG(
    dag_id='data_preprocessing',
    schedule_interval='@daily',
    start_date=datetime(2025, 1, 1),
    catchup=False,
)
​
# define the task
preprocess_task = PythonOperator(
    task_id='preprocess',
    python_callable=preprocess_data,
    dag=dag,
)
The above code defines a Directed Acyclic Graph with a single task to preprocess screentime data. The preprocess_data function loads the dataset, extracts temporal features (DayOfWeek and Month) from the Date column, encodes the App column using one-hot encoding, and scales numerical features (Notifications and Times Opened) using MinMaxScaler.

Next, the system saves the processed data to a new CSV file. The Airflow DAG schedules this task daily, which ensures automation and reproducibility in the data preparation process.

Testing and Running the Pipeline
Testing and Running the Pipeline are meant to be executed in the terminal. These commands should be run in separate terminal windows (or tabs) while your Python environment is active. First, initialize the database:

airflow db init
This command initializes the metadata database used by Airflow to store details about tasks, DAGs, and schedules.

Next, start the Airflow webserver:

airflow webserver --port 8080
This starts the Airflow webserver, which hosts the user interface for managing DAGs and monitoring task execution. The default port is 8080, but you can specify a different port if needed.

Finally, start the Airflow scheduler:

airflow scheduler
The scheduler is responsible for executing tasks as per the DAG schedule. It monitors the tasks and ensures they are executed in the correct order.

To access the Airflow UI, navigate to http://localhost:8080 in your browser. Once there, enable the data_preprocessing DAG and manually trigger it to execute the defined tasks. After the DAG has run successfully, validate the output by checking the preprocessed file to ensure it contains the updated and preprocessed data.

Summary
So, building an MLOps pipeline using Apache Airflow simplifies the end-to-end process of data preprocessing, model training, and deployment. Automating tasks through DAGs ensures efficiency, scalability, and reproducibility in managing machine learning workflows.
