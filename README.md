# Lambda training app 2 - simple TODO app

Sample project just for the purpose of learning AWS Lambdas and DynamoDB

## Installing

### Requirements
* Linux/Mac
* node.js 8.10.*


### Install
```
npm install
serverless dynamodb install
```

## Tutorial
### Serverless Configuration
**Set new table name**
```
environment:
    TABLE_NAME: ${self:service}-${self:provider.stage}-todos
```
**Update DynamoDB to allow : GET, PUT, DELETE and SCAN**
```
iamRoleStatements:
    - Effect: Allow
      Action:
        - dynamodb:GetItem
        - dynamodb:PutItem
        - dynamodb:DeleteItem
        - dynamodb:Scan
      Resource: arn:aws:dynamodb:*:*:*
```
DynamoDB offline works even without the permissions set, but after deploying to AWS it will throw an error without the permissions set.

All permissions for DynamoDB :

[https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/api-permissions-reference.html](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/api-permissions-reference.html)

**Define DynamoDB Schema**
```
resources:
  Resources:
    ExampleDynamoDB:
      Type: AWS::DynamoDB::Table
      Properties:
        TableName: ${self:provider.environment.TABLE_NAME}
        AttributeDefinitions:
          - AttributeName: uuid
            AttributeType: S
        KeySchema:
          - AttributeName: uuid
            KeyType: HASH
        ProvisionedThroughput:
          ReadCapacityUnits: 5
          WriteCapacityUnits: 5
```
**Define New Handlers**
```
functions:
  createTodo:
      handler: handler.createTodo
      events:
        - http:
            path: todo
            method: post
            cors: true
            private: false
```

You can specify query parameters if needed:
```
- http:
      path: todo/{id} #named parameter ID
      method: get
      cors: true
      private: false
      request:
        parameters:
           paths:
             id: true #required
```

### Endpoint to create new TODO task
It will accept a POST request with a JSON body :
```
{"title":"First TODO task"}
```
**AWS.DynamoDB vs AWS.DynamoDB.DocumentClient**

AWS.DynamoDB.DocumentClient is a simplified version of AWS.DynamoDB.
It supports only document operations like put,get, delete etc. with a simplified syntax on a specific table.

It does NOT support any global operations like creating new databses, listing all databases, updating tables etc.

**AWS.DynamoDB.DocumentClient.put**

[https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/DynamoDB/DocumentClient.html#put-property](https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/DynamoDB/DocumentClient.html#put-property)

Example put data
```
let putConfig = {
    TableName: "example-table-name",
    Item : {uuid:"123456789", title:"TODO title"},
    ReturnValues: 'ALL_OLD' #Returns all of the attributes of the item, as they appeared before the UpdateItem operation
}
```
For all ReturnValues options see
[https://docs.aws.amazon.com/amazondynamodb/latest/APIReference/API_UpdateItem.html#DDB-UpdateItem-request-ReturnValues](https://docs.aws.amazon.com/amazondynamodb/latest/APIReference/API_UpdateItem.html#DDB-UpdateItem-request-ReturnValues)
```
module.exports.createTodo = async (event, context) => {
    //Create new DB connnection
    let dbConfiguration = process.env.IS_OFFLINE ? {region: 'localhost', endpoint: 'http://localhost:8000'} : {};
    let dynamo = new AWS.DynamoDB.DocumentClient(dbConfiguration);
    
    var uuid = require('uuid'); // requires npm install uuid
    let todoItem = JSON.parse(event.body);
    if(!todoItem.uuid) {
        todoItem.uuid = uuid.v4();
    }
    let putConfig = {
        TableName: process.env.TABLE_NAME,
        Item : todoItem,
        ReturnValues: 'ALL_OLD'
    }
    return new Promise((resolve,reject) => {
        dynamo.put(putConfig, (err, data) => {
            if(err) {
                resolve({body:JSON.stringify(err), statusCode:400});
            } else {
                resolve({body:JSON.stringify(todoItem), statusCode:201});
            }
        });
    });
};

```
## Run
```
serverless offline start
```
## Other Endpoints
Create :
1. get
2. getAll
3. delete

## Deployment

1. install AWS CLI - https://docs.aws.amazon.com/cli/latest/userguide/installing.html
2. set secret access env variable ``export AWS_SECRET_ACCESS_KEY=<your secret>``
3. set access env variable ``export AWS_ACCESS_KEY_ID=<your key>``

To deploy run :
```
serverless deploy
```

## Result
See folder ``result`` for the final result.

## Contributing
Pull requests are welcome. For major changes, please open an issue first to discuss what you would like to change.

Please make sure to update tests as appropriate.

## License
[MIT](https://choosealicense.com/licenses/mit/)
