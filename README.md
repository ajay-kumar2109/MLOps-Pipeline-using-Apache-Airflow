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

1
import pandas as pd
2
from sklearn.preprocessing import MinMaxScaler
3
​
4
# load the dataset
5
data = pd.read_csv('screentime_analysis.csv')
6
​
7
# check for missing values and duplicates
8
print(data.isnull().sum())
9
print(data.duplicated().sum())
10
​
11
# convert Date column to datetime and extract features
12
data['Date'] = pd.to_datetime(data['Date'])
13
data['DayOfWeek'] = data['Date'].dt.dayofweek
14
data['Month'] = data['Date'].dt.month
15
​
16
# encode the categorical 'App' column using one-hot encoding
17
data = pd.get_dummies(data, columns=['App'], drop_first=True)
18
​
19
# scale numerical features using MinMaxScaler
20
scaler = MinMaxScaler()
21
data[['Notifications', 'Times Opened']] = scaler.fit_transform(data[['Notifications', 'Times Opened']])
22
​
23
# feature engineering
24
data['Previous_Day_Usage'] = data['Usage (minutes)'].shift(1)
25
data['Notifications_x_TimesOpened'] = data['Notifications'] * data['Times Opened']
26
​
27
# save the preprocessed data to a file
28
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
1
from sklearn.model_selection import train_test_split
2
from sklearn.ensemble import RandomForestRegressor
3
from sklearn.metrics import mean_absolute_error
4
​
5
# split data into features and target variable
6
X = data.drop(columns=['Usage (minutes)', 'Date'])
7
y = data['Usage (minutes)']
8
​
9
# train-test split
10
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)
11
​
12
# train the model
13
model = RandomForestRegressor(random_state=42)
14
model.fit(X_train, y_train)
15
​
16
# evaluate the model
17
predictions = model.predict(X_test)
18
mae = mean_absolute_error(y_test, predictions)
19
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
1
from airflow import DAG
2
from airflow.operators.python import PythonOperator
3
from datetime import datetime
4
​
5
# define the data preprocessing function
6
def preprocess_data():
7
    file_path = 'screentime_analysis.csv'
8
    data = pd.read_csv(file_path)
9
​
10
    data['Date'] = pd.to_datetime(data['Date'])
11
    data['DayOfWeek'] = data['Date'].dt.dayofweek
12
    data['Month'] = data['Date'].dt.month
13
​
14
    data = data.drop(columns=['Date'])
15
​
16
    data = pd.get_dummies(data, columns=['App'], drop_first=True)
17
​
18
    scaler = MinMaxScaler()
19
    data[['Notifications', 'Times Opened']] = scaler.fit_transform(data[['Notifications', 'Times Opened']])
20
​
21
    preprocessed_path = 'preprocessed_screentime_analysis.csv'
22
    data.to_csv(preprocessed_path, index=False)
23
    print(f"Preprocessed data saved to {preprocessed_path}")
24
​
25
# define the DAG
26
dag = DAG(
27
    dag_id='data_preprocessing',
28
    schedule_interval='@daily',
29
    start_date=datetime(2025, 1, 1),
30
    catchup=False,
31
)
32
​
33
# define the task
34
preprocess_task = PythonOperator(
35
    task_id='preprocess',
36
    python_callable=preprocess_data,
37
    dag=dag,
38
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
