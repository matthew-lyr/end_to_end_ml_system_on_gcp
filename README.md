# End-to-end Machine Learning Framework on Google Cloud Platform

### Overview
This project provides a simply way to implement a scalable machine learning system on Google Cloud Platform that handles on-going data cleaning, model training, model updating, and prediction for large-scale structured data.

### Special Features
1. Able to make large-scale batch predictions for more than hundreds of millions of records. 
2. Data cleaning and preparation is simple and straight-forward: use SQL to describe how data needs to be prepared.
3. Trained models can be easily re-trained and updated periodically.

### How it works
![alt text](https://github.com/matthew-lyr/end_to_end_machine_learning_framework/blob/main/End_to_end_ml_high_level_flowchart.PNG
)
There are three main pieces in the framework:
1. Trainer grabs data from the data source, trains a model, and exports the trained model.
2. Model_manager deploys the trained model or use it to replace a deployed model.
3. Auto_predictor then uses the deployed model to make batch predictions on the input data specified by users in SQL.

In essence, what the framework is really doing is to utilize various services provided by Google Cloud, such as BigQuery, Cloud Storage, and ML Engine/AI Platform. The framework utilizes their strengths and orchestrates them into a connected system. 

![alt text](https://github.com/matthew-lyr/end_to_end_machine_learning_framework/blob/main/End_to_end_ml_detailed_flowchart.PNG
)


### Code Examples

Train and deploy the trained model:
```
project_id          = 'yoyoyo-27106'                                               
service_account_dir = '/srv/secrets/yoyoyo-27106-934d57ded.json'
sql                 = "SELECT * FROM `yoyoyo-27106.finance.daily_stock_price` "
steps               = 1000
dropout_num         = 0.1
batch_size          = 10
hidden_units        = [100,100,100,100]
target_name         = "next_open"
modelName           = "finance_model"
versionName         = "v1"

trainer = Trainer(project_id, service_account_dir)
trainer.train(sql, steps, dropout_num, batch_size, hidden_units, target_name, is_classification = False)
completed_model_dir = trainer.export_model()
model_manager       = Model_manager(modelName, versionName, project_id, completed_model_dir)
model_manager.remove_model()  # Remove existing deployed model
model_manager.create_model()
```

Make batch prediction with the deployed model:

```
credentials   = '/srv/secrets/yoyoyo-27106-934d57ded.json'
project_id    = 'yoyoyo-27106' 
dataset_id    = 'finance'
model_name    = "finance_model"
bucket        = "gs://yiren-data/test"
sql           = "SELECT * FROM (SELECT *except(next_open) FROM `yoyoyo-27106.finance.daily_stock_price` order by date)"

predictor = Auto_predictor(credentials, project_id, dataset_id, model_name, bucket, sql)
predictor.prepare_input_data()
predictor.batch_predict()
```

