# AWS Serverless Framework Tutorial notes

1. ## Setting up the serverless framework with AWS

	User: ServerlessAccount <br/>
	Access Key: AKIAYTG7K4F7CNVUBGUO <br/>
	Secrete Access Key: 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 <br/>

	Ensure permissions to install node packages globally. Ref "Mac Os X Answer" [here](https://stackoverflow.com/questions/33725639/npm-install-g-less-does-not-work-eacces-permission-denied). <br/>
	Install aws serverless: ` $ npm install -g serverless`.<br/>
	Set credentials: `$ serverless config credentials --provider aws --key {enterKey} --secret {enterSecret} --profile serverlessUser`.


2. ## Creating a new serverless project and deploying a lambda

	Set up serverless folder  `$ serverless create --template aws-nodejs --path myServerlessProject`. <br/>
	This should have generated a file called 'myServerlessProject' with handle.js and serverless.yml in. <br/>
	In serverless.yml, make sure to add the user profile to the provider. <br/>
	Then you can run `$ sls deploy` (sls is short for serverless) This creates a cloudformation template. <br/>
	NOTE: make sure your region is set correctly in serverless.yml file. It defaults to us-east-1, and should be eu-west-2. <br/>


3. ## How to deploy an s3 bucket and upload data

	Set up the file in `serverless.yml`. <br/>
    `BucketName` must be unique <br/>
    ```
		resources:
		  Resources:
		    DemoBucketUpload:
		      Type: AWS::S3::Bucket
		      Properties:
		        BucketName: myserverlessprojectuploadbucket-310316
    ```       
	run `$ sls deploy` to upload data to s3 bucket we can use a plugin called `serverless-s3-sync` <br/>
    ```
		plugins:
		  - serverless-s3-sync

		custom:
		  s3Sync:
		    - bucketName: myserverlessprojectuploadbucket-310316
		      localDir: UploadData
    ```
	run `$npm init` <br/>
	and add to dependencies run `$ npm install --save serverless-s3-sync` <br/>


4. ## Creating an API with serverless

	Create lambdas folder. <br/>
	Create a lambda, in this instance, getUser.js. <br/>
	Your porogotive how to, but create output or responses module to be reused. In this instance, inside the lambda folder, we created apiResponses.js. <br/>
	Once you've completed your API, you need to update `serverless.yml` to configure on upload. <br/>
    ```
		functions:
		  getUser:
		    handler: lambdas/getUser.handler
		    events:
		      - http:
		          path: get-user/{ID}
		          method: GET
		          cors: true
    ```
	Finally, deploy $ sls deploye. <br/>


5. ## Adding serverless webpack to your project

	When adding lambdas and deploying, by default it adds everything from the serverless repo. We can use webpack, to only include the files needed within the lambda. <br/>
    To add this, in the `serverless.yml` file, under `plugins` we add `serverless-webpack` <br/>
    A bit further down the same file, we add `package` with `individually: true` <br/>

    ```
        plugins:
          - serverless-s3-sync
          - serverless-webpack

        package:
          individually: true
    ```

    Install plugin: `npm install --save serverless-webpack` <br/>
    After this, install webpack itself: `npm install --save webpack`
    Next we create a new config file, same level as lambdas folder: `webpack.config.js`. <br/>
    Once this is done, run `sls deploy` <br/>

6. ## Create a serverless database - DynamoDB database

    Adding a table via the resources in our `serverless.yml` file. <br/>
    ``` 
        MyDynamoDbTable:
          Type: AWS::DynamoDB::Table
          Properties:
            TableName: ${self:custom.tableName}
            AttributeDefinitions:
              - AttributeName: ID
                AttributeType: S
            KeySchema:
              - AttributeName: ID
                KeyType: HASH
            BillingMode: PAY_PER_REQUEST
    ```
    A little further up the file, in `custom` we have created a variable we can reference elsewhere called `tableName`. Notice we used this in `Properties.TableName` above. <br/>

    ```
        custom:
          tableName: player-points
          s3Sync:
              - bucketName: myserverlessprojectuploadbucket-310316
              localDir: UploadData
    ```

    once finished, you can run `sls deploy` <br/>
    You can make sure its worked by checking dynamoDB on AWS.

7. ## Create an API to get data from your DynamoDB database

    First thing to do is reorganise lambda file structure into `endpoints` and `common`. <br/>
    We move the `apiResponses.js` module into common and `getUser.js` to endpoints. <br/>
    Creating the `getPlayerScore` endpoint in endpoints file. We'll need a common `get()` method from dynamoDB. So next we create `Dynamo.js` in common. <br/>
    Next we update our `serverless.yml` file.
    
    ```
        environment:
          tableName: ${self:custom.tableName}
        iamRoleStatements:
          - Effect: Allow
          Action:
          - dynamodb:*
          - lambda:*
          Resource: '*'

        ...

        custom:
          tableName: player-points
          s3Sync:
              - bucketName: myserverlessprojectuploadbucket-310316
              localDir: UploadData

        functions:
          getUser:
            handler: lambdas/endpoints/getUser.handler
            events:
              - http:
                  path: get-user/{ID}
                  method: GET
                  cors: true

          getPlayerScore:
            handler: lambdas/endpoints/getPlayerScore.handler
            events:
              - http:
                  path: get-player-score/{ID}
                  method: GET
                  cors: true
    ```

8. ## Adding new data to dynamoDB 


