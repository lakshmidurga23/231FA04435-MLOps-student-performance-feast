# Student Performance Prediction using Feast

## 1. Problem Statement

The objective of this project is to build a Machine Learning pipeline for predicting a student's final examination score using academic and lifestyle-related features.

Feast Feature Store is used to demonstrate:

- Feature engineering
- Historical feature retrieval
- Machine Learning training
- Feature materialization
- Online feature retrieval
- Final prediction

> **Note:** The current implementation focuses on student performance prediction. Curriculum-level and industry-level skill evidence is considered as future work.

---

## 2. Dataset

The dataset contains **1,000 student records** and **12 original columns**.

### Original Dataset Columns

| Column | Description |
|---|---|
| `student_id` | Unique student identifier |
| `gender` | Student gender |
| `study_time_hours` | Daily study time |
| `attendance_percent` | Attendance percentage |
| `sleep_hours` | Sleep duration |
| `parental_education` | Parent education level |
| `internet_access` | Internet availability |
| `extracurricular_activities` | Extracurricular activity participation |
| `part_time_job` | Part-time job status |
| `previous_grade` | Previous academic grade |
| `final_exam_score` | Final examination score |
| `final_grade` | Final letter grade |

### Target Variable

The Machine Learning target is:

final_exam_score
3. Feature Engineering

The dataset was cleaned and transformed before registering the features with Feast.

Data Preprocessing
Missing numerical values were filled using the median.
Missing parental_education values were replaced with Unknown.
Yes/No values were converted into numerical values:
Yes = 1
No = 0
Engineered Features
Study Attendance Ratio
study_attendance_ratio = study_time_hours / attendance_percent

For Student 1:

4.0 / 98.0 = 0.0408
Performance Change
performance_change = final_exam_score - previous_grade

For Student 1:

100.0 - 76.9 = 23.1
Study Efficiency
study_efficiency = final_exam_score / study_time_hours

An event_timestamp column was also created for Feast's time-aware feature retrieval.

4. Feast Architecture

The project uses the following Feast workflow:

Student Dataset
      |
      v
Feature Engineering
      |
      v
student_features.parquet
      |
      v
Feast FileSource
      |
      v
FeatureView
student_performance_features
      |
      +----------------------+
      |                      |
      v                      v
Historical Retrieval    Materialization
      |                      |
      v                      v
ML Training            SQLite Online Store
                             |
                             v
                     Online Feature Retrieval
                             |
                             v
                       ML Prediction
Entity

The Feast entity is:

student_id

Each student is uniquely identified using this ID.

Data Source

The Feast data source is:

student_features.parquet

The timestamp field is:

event_timestamp
FeatureView

The FeatureView is:

student_performance_features
5. Features Stored in the FeatureView

The student_performance_features FeatureView contains:

study_time_hours
attendance_percent
sleep_hours
previous_grade
internet_access
extracurricular_activities
part_time_job
study_attendance_ratio
performance_change
study_efficiency

The target final_exam_score is stored separately in student_targets.csv and is used for model training.

6. Implementation
Feast Apply

The Feast definitions were registered using:

feast apply

This created the required Feast entity, FeatureView, and online-store tables.

Historical Feature Retrieval

Historical features were retrieved using:

store.get_historical_features()

The resulting features were combined with the target variable and used for Machine Learning.

Machine Learning Model

A Random Forest Regressor was trained using the Feast historical features.

The dataset was divided into:

Training records: 800
Testing records: 200
Materialization

Features were materialized into the SQLite online store.

Online Feature Retrieval

Online features were retrieved using:

store.get_online_features()

For example, features for:

student_id = 1

were successfully retrieved from the online store.

7. Results
Model Performance
Metric	Result
Training Records	800
Testing Records	200
MAE	1.44
RMSE	2.24
R²	0.9535

The Random Forest model achieved an R² score of 0.9535 on the test dataset.

Online Feature Retrieval

For Student 1, Feast successfully returned:

study_time_hours            = 4.0
attendance_percent          = 98.0
sleep_hours                 = 6.5
previous_grade              = 76.9
internet_access             = 1
extracurricular_activities  = 1
part_time_job               = 0
study_attendance_ratio      = 0.0408
performance_change          = 23.1
study_efficiency            = 25.0
Final Prediction
Student ID: 1
Predicted Final Exam Score: 98.85
8. Required Analysis
1. What is the entity in your Feast implementation?

The entity is:

student_id

It uniquely identifies each student and is used by Feast to retrieve the student's features.

2. List the features stored in your FeatureView.

The student_performance_features FeatureView contains:

study_time_hours
attendance_percent
sleep_hours
previous_grade
internet_access
extracurricular_activities
part_time_job
study_attendance_ratio
performance_change
study_efficiency
3. Explain how one feature was calculated.

The performance_change feature was calculated as:

performance_change = final_exam_score - previous_grade

For Student 1:

100.0 - 76.9 = 23.1

This represents the change between the student's previous grade and final examination score.

4. What is the difference between your original dataset and the feature dataset?

The original dataset contains 1,000 records and 12 columns, including student information and target variables.

The feature dataset contains the features required by Feast and the Machine Learning model.

The feature dataset:

Converts Yes/No values to numerical values.
Handles missing values.
Adds engineered features.
Adds an event_timestamp.
Removes unnecessary columns such as gender and parental_education from the FeatureView.

The target values are stored separately in:

student_targets.csv
5. What is the purpose of the offline store?

The offline store contains historical feature data.

In this project, the feature data is stored in:

student_features.parquet

It is used for:

Historical feature retrieval
Creating Machine Learning training data
Reproducing historical features
6. What is the purpose of the online store?

The online store provides feature values for fast online retrieval during prediction.

This project uses a SQLite online store.

After materialization, features can be retrieved using:

store.get_online_features()
7. What is the purpose of feast apply?

feast apply registers the Feast definitions with the Feature Store.

It creates and updates objects such as:

Entities
Data Sources
FeatureViews
Online-store tables

In this project:

feast apply

was used to register:

student_id
student_performance_features
8. What does materialization do?

Materialization transfers feature values from the offline data source to the online store.

In this project:

student_features.parquet
        |
        v
Materialization
        |
        v
SQLite Online Store

After materialization, the features could be retrieved online for Student 1.

9. What is the advantage of retrieving features through Feast instead of manually calculating them separately during training and prediction?

Feast provides a centralized way to manage and retrieve features.

Without Feast, feature calculations may need to be duplicated for training and prediction, which can cause inconsistencies.

With Feast:

Features are defined centrally.
Historical features can be retrieved for training.
Online features can be retrieved for prediction.
Feature definitions remain consistent.
Feature management becomes easier to reproduce and maintain.
10. State two limitations of your current dataset.
Limitation 1: Limited Academic Information

The dataset contains general student performance information but does not contain detailed subject-level or topic-level information.

For example, it does not include:

Subject-wise scores
Topic-level performance
Learning outcomes
Detailed assessment history
Limitation 2: Limited Curriculum and Industry Evidence

The current dataset does not contain curriculum or industry information such as:

Industry-required skills
Job-role requirements
Current technology trends
Certification requirements

Therefore, the current implementation mainly demonstrates student performance prediction.

11. State two ways your feature store could be improved when more curriculum and industry evidence becomes available.
Improvement 1: Add Curriculum-Level Features

The Feature Store could be extended with:

Subject-level performance
Topic-level performance
Assignment scores
Quiz scores
Skill mastery
Course completion
Learning outcomes
Improvement 2: Add Industry-Level Features

The Feature Store could also include:

Job-role skill requirements
Industry-demanded technologies
Programming language requirements
Certification requirements
Internship requirements
Industry skill-demand trends

This would allow the system to combine student performance with curriculum and industry evidence for better skill-gap analysis.

9. Conclusion

This project demonstrates an end-to-end Machine Learning workflow using Feast Feature Store.

The project successfully implements:

Data Preparation
      ↓
Feature Engineering
      ↓
Feast FeatureView
      ↓
Historical Feature Retrieval
      ↓
Machine Learning
      ↓
Model Evaluation
      ↓
Feature Materialization
      ↓
Online Feature Retrieval
      ↓
Final Prediction
Final Results
Training Records : 800
Testing Records  : 200

MAE              : 1.44
RMSE             : 2.24
R²               : 0.9535

Student ID       : 1
Predicted Score  : 98.85
