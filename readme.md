# zModel Feature Pack
zModel feature pack provides features, which extend basic [zModel framework](https://github.com/elinsoftware/zmodel):
 1. [Workflow activities](#workflow-activities) for 3 models: Linear, Logistic and Softmax regression.
 2. [REST API](#rest-api), which allows to:
    - query zModel framework data (models and scores)
    - run the model on particular input data

>zModel feature pack requires Orchestration plugin and zModel framework to be installed in ServiceNow instance.

![img](assets/img1.png)

### Contacts
Questions, requests for customization or development services - support@dev-labs.io

## Workflow activities
Workflow activities provide the ability to use predictive models in ServiceNow workflows for any business/ITIL application. zModel activities can be found on *Packs* tab in Workflow editor:
<p align="center">
<img src="assets/11.png" style="width:40%;">
</p>

#### How to use zModel workflow activities
1. Drag and drop the corresponding model to the workflow
2. Connect activity (input and output)
3. Specify model parameters.

<p align="center">
<img src="assets/2.png" style="width:90%;">
</p>

>Model parameters can be specified manually when creating a workflow or during runtime using scratchpad variables or workflow input parameters.

The following parameters should be specified:
1. **Model** - sys_id of the corresponding model in zModel framework.
2. **Input Data** - input data, which should be scored by the model. This data should be passed to the model in runtime using workflow input variables or scratchpad variables.
3. **Transaction Reference** - optional parameter, which may be used as a unique transaction ID or as a specific score reference. Can be defined during runtime of when designing a workflow.


#### How to pass data to the model activity
Input data can be passed to the model activity using workflow input variables (most common use case):
1. Define a workflow input variable.
2. Populate the variable in a script.
3. Add a reference to the workflow variable in the model parameters. You can reference the variable as`${workflow.inputs.u_input_data}`, where `u_input_data` is a name of the workflow input variable.

<p align="center">
<img src="assets/3.png" style="width:90%;">
</p>

Using scratchpad variable (legacy option):
1. Define a scratchpad variable prior to the model activity, populate that variable with the corresponding data you need to score (this can be done in *Run Script* activity):
```javascript
workflow.scratchpad.input_data = "[0,25,86,12,73,72,0,1]";
```
2. Use that variable in the model (activity) parameters. You can reference scratchpad variable as `${workflow.scratchpad.input_data}`

#### How to read the model output data
<p align="center">
<img src="assets/701.png" style="width:90%;">
</p>

Model output stored as an object in *Score* activity variable. If you have multiple models in the workflow then multiple *Score* objects will be available.

<p align="center">
<img src="assets/7.png" style="width:30%;">
</p>

You can read the output *Score* object using the following options:
1. If you need to read model's output and put it as *an input parameter* in to some other activities then you can reference it as `${data.get(x).Score}`, where `x` is a model reference number from *Data* tab in a workflow editor.
2. If you want to access model's output in a script then you can use the following reference - `data.get(x).Score`.
```javascript
// write model calculation Status to info log in ServiceNow
// "9" - is a reference number of the model on a Data tab

gs.info(data.get(9).Score.Status);
```

## REST API
zModel framework REST API based on the out-of-the-box ServiceNow Table API (#1-#7) and a custom scripted web service (#8).

The following requests supported by zModel REST API:
 1. [GET the list of Models](#1-get-the-list-of-models)
 2. [GET the list of Scores](#2-get-the-list-of-scores)
 3. [GET the specific model by sys_id](#3-get-the-specific-model-by-sys_id)
 4. [GET the list of scores by Model sys_id](#4-get-the-list-of-scores-by-model-sys_id)
 5. [GET the specific score by sys_id](#5-get-the-specific-score-by-sys_id)
 6. [CREATE model](#6-create-model)
 7. [UPDATE model](#7-update-model)
 8. [RUN score](#8-run-score)

>ServiceNow Table API reference: https://docs.servicenow.com/bundle/geneva-servicenow-platform/page/integrate/inbound_rest/concept/c_TableAPI.html

 ##### Security

 For requests #1-7 `rest_service` role should be assigned to a user account in order to performs REST calls.

 For request #8 `rest.score.calculation` role should be assigned to the corresponding account.

##### Response
ServiceNow responds with a JSON object, which contains information about the transaction itself and the corresponding record or list of records.

### 1. GET the list of Models
>All GET methods based on ServiceNow Table API

This method retrieves multiple records for **Model** table with proper pagination information. Additional request parameters can be added, please refer to [ServiceNow Table API documentation](https://docs.servicenow.com/bundle/geneva-servicenow-platform/page/integrate/inbound_rest/reference/r_TableAPI-GET.html).

- Method: **GET**
- Endpoint: https://<your_instance>.service-now.com/api/now/v1/table/x_elsr_z_model_model
- Authentication: Basic

Javascript jQuery example:
```javascript
var settings = {
  "async": true,
  "crossDomain": true,
  "url": "https://<your_instance>.service-now.com/api/now/v1/table/x_elsr_z_model_model",
  "method": "GET",
  "headers": {
    "authorization": "Basic UkVTVC56TW9kZWw6YW5kcmV3NzA3",
    "cache-control": "no-cache",
    "postman-token": "717d0879-ffba-2440-76b4-a6a731a45504"
  }
}

$.ajax(settings).done(function (response) {
  console.log(response);
});
```


### 2. GET the list of Scores
This method retrieves multiple records for **Score** table.
- Method: **GET**
- Endpoint: https://<your_instance>.service-now.com/api/now/v1/table/x_elsr_z_model_score
- Authentication: Basic

Javascript jQuery example:
```Javascript
var settings = {
  "async": true,
  "crossDomain": true,
  "url": "https://<your_instance>.service-now.com/api/now/v1/table/x_elsr_z_model_score",
  "method": "GET",
  "headers": {
    "authorization": "Basic UkVTVC56TW9kZWw6YW5kcmV3NzA3",
    "cache-control": "no-cache",
    "postman-token": "ce02e558-4477-9dec-258e-4192ea8d0d5f"
  }
}

$.ajax(settings).done(function (response) {
  console.log(response);
});
```
### 3. GET the specific model by sys_id
This method retrieves a specific model by model sys_id.

 - Method: **GET**
 - Endpoint: https://<your_instance>.service-now.com/api/now/v1/table/x_elsr_z_model_model
 - Authentication: Basic
 - Parameters: key - `sysparm_query`, value - `sys_id=XXX`, where XXX is a sys_id of the corresponding model.

Javascript jQuery example:
```Javascript
var settings = {
  "async": true,
  "crossDomain": true,
  "url": "https://<your_instance>.service-now.com/api/now/v1/table/x_elsr_z_model_model?sysparm_query=sys_id%3Dd491ea19db77ba80cc38f7fdbf9619f4",
  "method": "GET",
  "headers": {
    "authorization": "Basic UkVTVC56TW9kZWw6YW5kcmV3NzA3",
    "cache-control": "no-cache",
    "postman-token": "f8e0557f-d89d-f1a2-725d-e6797c2a3b16"
  }
}

$.ajax(settings).done(function (response) {
  console.log(response);
});
```
#### 4. GET the list of scores by Model sys_id
This method retrieves a list of all scores for the particular model.
- Method: **GET**
- Endpoint: https://<your_instance>.service-now.com/api/now/v1/table/x_elsr_z_model_score
- Authentication: Basic
- Parameters: key - `sysparm_query`, value - `model=XXX`, where XXX is a sys_id of the corresponding model.

Javascript jQuery example:
```javascript
var settings = {
  "async": true,
  "crossDomain": true,
  "url": "https://<your_instance>.service-now.com/api/now/v1/table/x_elsr_z_model_score?sysparm_query=model%3Dd491ea19db77ba80cc38f7fdbf9619f4",
  "method": "GET",
  "headers": {
    "authorization": "Basic UkVTVC56TW9kZWw6YW5kcmV3NzA3",
    "cache-control": "no-cache",
    "postman-token": "3565a564-211d-7645-0b01-f33ce481847a"
  }
}

$.ajax(settings).done(function (response) {
  console.log(response);
});
```
#### 5. GET the specific score by sys_id
This method retrieves a specific score record from Score table by sys_id.
- Method: **GET**
- Endpoint: https://<your_instance>.service-now.com/api/now/v1/table/x_elsr_z_model_score
- Authentication: Basic
- Parameters: key - `sysparm_query`, value - `sys_id=XXX`, where XXX is a sys_id of the corresponding score record.

Javascript jQuery example:
```javascript
var settings = {
  "async": true,
  "crossDomain": true,
  "url": "https://<your_instance>.service-now.com/api/now/v1/table/x_elsr_z_model_score?sysparm_query=sys_id%3D4868c671db77fa80cc38f7fdbf961907",
  "method": "GET",
  "headers": {
    "authorization": "Basic UkVTVC56TW9kZWw6YW5kcmV3NzA3",
    "cache-control": "no-cache",
    "postman-token": "f8b3f25d-93bb-bd82-04b5-dd0f3705c3a1"
  }
}

$.ajax(settings).done(function (response) {
  console.log(response);
});
```

#### 6. CREATE model
This method creates a model record in **Model** table.
- Method: **POST**
- Endpoint: https://<your_instance>.service-now.com/api/now/v1/table/x_elsr_z_model_model
- Authentication: Basic
- Content-type: application/json
- Body: JSON object

Request body example:
```JSON
{
	"type": "Logistic Regression",
	"name": "test REST model",
	"description": "test model deployed by REST API",
	"work_notes": "first comment on model deployment",
	"logic": "[[0.036239],[0.07511084],[-0.05622768],[-0.0388718],[-0.00453927],[0.16277024],[0.26297148],[-0.00086712],[-0.00365082],[0.01359326],[-0.00066987],[-0.02952172],[0.8052103]]"
}
```
Javascript jQuery example:
```javascript
var settings = {
  "async": true,
  "crossDomain": true,
  "url": "https://<your_instance>.service-now.com/api/now/v1/table/x_elsr_z_model_model",
  "method": "POST",
  "headers": {
    "content-type": "application/json",
    "authorization": "Basic UkVTVC56TW9kZWw6YW5kcmV3NzA3",
    "cache-control": "no-cache",
    "postman-token": "ca3e4d8c-d94e-79cd-39e0-4838292a7baf"
  },
  "processData": false,
  "data": "{\n\t\"type\": \"Logistic Regression\",\n\t\"name\": \"test REST model\",\n\t\"description\": \"test model deployed by REST API\",\n\t\"work_notes\": \"first comment on model deployment\",\n\t\"logic\": \"[[0.036239],[0.07511084],[-0.05622768],[-0.0388718],[-0.00453927],[0.16277024],[0.26297148],[-0.00086712],[-0.00365082],[0.01359326],[-0.00066987],[-0.02952172],[0.8052103]]\"\n}"
}

$.ajax(settings).done(function (response) {
  console.log(response);
});
```

#### 7. UPDATE model
This method updates the existing model.

- Method: **PUT**
- Endpoint: https://<your_instance>.service-now.com/api/now/v1/table/x_elsr_z_model_model/{sys_id}, where {sys_id} is model's sys_id
- Authentication: Basic
- Content-type: application/json
- Body: JSON object

Request body example:
```json
{
	"work_notes": "second comment on the existing model"
}
```
Javascript jQuery example:
```Javascript
var settings = {
  "async": true,
  "crossDomain": true,
  "url": "https://<your_instance>.service-now.com/api/now/v1/table/x_elsr_z_model_model/c4497e9edb8407c0cc38f7fdbf9619f0",
  "method": "PUT",
  "headers": {
    "content-type": "application/json",
    "authorization": "Basic UkVTVC56TW9kZWw6YW5kcmV3NzA3",
    "cache-control": "no-cache",
    "postman-token": "47ec108f-f17a-4553-9069-0672a71b019d"
  },
  "processData": false,
  "data": "{\n\t\"type\": \"Logistic Regression\",\n\t\"name\": \"updated name\",\n\t\"description\": \"updated description\",\n\t\"work_notes\": \"second comment on the existing model\",\n\t\"logic\": \"[]\"\n}"
}

$.ajax(settings).done(function (response) {
  console.log(response);
});
```

#### 8. RUN score
This method runs model calculation for the corresponding input data.
- Method: **POST**
-  Endpoint: https://<your_instance>.service-now.com/api/x_elsr_zmodel_fp/zmodel
- Authentication: Basic
- Content-type: application/json
- Body: JSON object. The object should contain model type, model sys_id, input data and (optional) external reference id.

Request body example:
```json
{
  "modelType":"Linear Regression",
  "modelSys_id":"98340bf1dbc7f240cc38f7fdbf961957",
  "inputData":"[96,93,95,1]",
  "externalReference":"REST API score run"
}
```
> Three model types supported - "Linear Regression", "Logistic Regression" and "Softmax Regression".

Javascript jQuery example:
```Javascript
var settings = {
  "async": true,
  "crossDomain": true,
  "url": "https://<your_instance>.service-now.com/api/x_elsr_zmodel_fp/zmodel",
  "method": "POST",
  "headers": {
    "content-type": "application/json",
    "authorization": "Basic UkVTVC56TW9kZWw6YW5kcmV3NzA3",
    "cache-control": "no-cache",
    "postman-token": "4b78ffb5-93f8-1627-65f1-099fbe07eac9"
  },
  "processData": false,
  "data": "{\n  \"modelType\":\"Linear Regression\",\n  \"modelSys_id\":\"98340bf1dbc7f240cc38f7fdbf961957\",\n  \"inputData\":\"[96,93,95,1]\",\n  \"externalReference\":\"REST API score run\"\n}"
}

$.ajax(settings).done(function (response) {
  console.log(response);
});
```
Response example:
```json
{
    "result": {
        "status": 77,
        "status details": "Success",
        "score": "[191.19541163700285]",
        "score sys_id": "55bbe697db480300cc38f7fdbf96194b"
    }
}
```
