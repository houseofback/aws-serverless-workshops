# Module 3: Serverless Service Backend

In this module you'll use AWS Lambda and Amazon DynamoDB to build a backend process for handling requests from your web application. The browser application that you deployed in the first module allows users to request that a unicorn be sent to a location of their choice. In order to fulfill those requests, the JavaScript running in the browser will need to invoke a service running in the cloud.

You'll implement a Lambda function that will be invoked each time a user requests a unicorn. The function will select a unicorn from the fleet, record the request in a DynamoDB table and then respond to the front-end application with details about the unicorn being dispatched.

![Serverless backend architecture](../images/serverless-backend-architecture.png)

The function is invoked from the browser using Amazon API Gateway. You'll implement that connection in the next module. For this module you'll just test your function in isolation.

## Implementation Instructions

Each of the following sections provide an implementation overview and detailed, step-by-step instructions. The overview should provide enough context for you to complete the implementation if you're already familiar with the AWS Management Console or you want to explore the services yourself without following a walkthrough.

### 1. Create an Amazon DynamoDB Table

Use the Amazon DynamoDB console to create a new DynamoDB table. Call your table `Rides-firstname-lastname` and give it a partition key called `RideId` with type String. The table name and partition key are case sensitive. Make sure you use the exact IDs provided. Use the defaults for all other settings.

**:white_check_mark: Step-by-step directions**

1. Go to the Amazon DynamoDB Console

1. Choose **Create table**.

1. Enter `Rides-firstname-lastname` for the **Table name**. This field is case sensitive.

1. Enter `RideId` for the **Partition key** and select **String** for the key type. This field is case sensitive.

1. Check the **Use default settings** box and choose **Create**.


### 2. Create an IAM Role for Your Lambda function

#### Background

Every Lambda function has an IAM role associated with it. This role defines what other AWS services the function is allowed to interact with. For the purposes of this workshop, you'll need to create an IAM role that grants your Lambda function permission to write logs to Amazon CloudWatch Logs and access to write items to your DynamoDB table.

**:white_check_mark: Step-by-step directions**
1. Go to the AWS IAM Console
1. Select **Roles** in the left navigation bar and then choose **Create role**.
1. Select **Lambda** for the role type from the **AWS service** group, then click **Next: Permissions**

    **Note:** Selecting a role type automatically creates a trust policy for your role that allows AWS services to assume this role on your behalf. If you were creating this role using the CLI, AWS CloudFormation or another mechanism, you would specify a trust policy directly.

1. Begin typing `AWSLambdaBasicExecutionRole` in the **Filter** text box and check the box next to that role.
1. Also, add `AmazonDynamoDBFullAccess` as a policy just like you did above.

1. Click **Next: Tags**. Add any tags that you wish.
1. Click **Next: Review**.

1. Enter `WildRydesLambda-firstname-lastname` for the **Role name**.
1. Choose **Create role**. 

### 3. Create a Lambda Function for Handling Requests

#### Background

AWS Lambda will run your code in response to events such as an HTTP request. In this step you'll build the core function that will process API requests from the web application to dispatch a unicorn. In the next module you'll use Amazon API Gateway to create a RESTful API that will expose an HTTP endpoint that can be invoked from your users' browsers. You'll then connect the Lambda function you create in this step to that API in order to create a fully functional backend for your web application.

#### High-Level Instructions

Use the AWS Lambda console to create a new Lambda function called `RequestUnicorn-firstname-lastname` that will process the API requests. Use the provided [requestUnicorn.js](requestUnicorn.js) example implementation for your function code. Just copy and paste from that file into the AWS Lambda console's editor.

Make sure to configure your function to use the `WildRydesLambda` IAM role you created in the previous section.

**:white_check_mark: Step-by-step directions**
1. Choose on **Services** then select **Lambda** in the Compute section.

1. Click **Create function**.

1. Keep the default **Author from scratch** card selected.

1. Enter `RequestUnicorn-firstname-lastname` in the **Name** field.

1. Select **Node.js 10.x** for the **Runtime**.
2. Expand *Choose or create an execution role* under **Permissions**.
1. Ensure `Choose an existing role` is selected from the **Role** dropdown.

1. Select `WildRydesLambda-firstname-lastname` from the **Existing Role** dropdown.
    ![Create Lambda function screenshot](../images/create-lambda-function.png)

1. Click on **Create function**.

1. Scroll down to the **Function code** section and replace the existing code in the **index.js** code editor with the contents of [requestUnicorn.js](requestUnicorn.js).
    ![Create Lambda function screenshot](../images/create-lambda-function-code.png)
    
1. Edit line 90 of the function code `TableName: 'Rides',` to `TableName: 'Rides-firstname-lastname',`

1. Click **"Save"** in the upper right corner of the page.


## Implementation Validation

For this module you will test the function that you built using the AWS Lambda console. In the next module you will add a REST API with API Gateway so you can invoke your function from the browser-based application that you deployed in the first module.

**:white_check_mark: Step-by-step directions**
1. From the main edit screen for your function, select **Configure test event** from the the **Select a test event...** dropdown.
    ![Configure test event](../images/configure-test-event.png)

1. Keep **Create new test event** selected.

1. Enter `TestRequestEvent` in the **Event name** field

1. Copy and paste the following test event into the editor:

    ```JSON
    {
        "path": "/ride",
        "httpMethod": "POST",
        "headers": {
            "Accept": "*/*",
            "Authorization": "eyJraWQiOiJLTzRVMWZs",
            "content-type": "application/json; charset=UTF-8"
        },
        "queryStringParameters": null,
        "pathParameters": null,
        "requestContext": {
            "authorizer": {
                "claims": {
                    "cognito:username": "the_username"
                }
            }
        },
        "body": "{\"PickupLocation\":{\"Latitude\":47.6174755835663,\"Longitude\":-122.28837066650185}}"
    }
    ```

    ![Configure test event](../images/configure-test-event-2.png)

1. Click **Create**.

1. On the main function edit screen click **Test** with `TestRequestEvent` selected in the dropdown.   

1. Scroll to the top of the page and expand the **Details** section of the **Execution result** section.

1. Verify that the execution succeeded and that the function result looks like the following:
```JSON
{
    "statusCode": 201,
    "body": "{\"RideId\":\"1h0zDZ-6KLZaEQCPyqTxeQ",\"Unicorn\":{\"Name\":\"Shadowfax\",\"Color\":\"White\",\"Gender\":\"Male\"},\"UnicornName\":\"Shadowfax\",\"Eta\":\"30 seconds\",\"Rider\":\"the_username\"}",
    "headers": {
        "Access-Control-Allow-Origin": "*"
    }
}
```

After you have successfully tested your new function using the Lambda console, you can move on to the next module, [RESTful APIs](../4_RESTfulAPIs).
