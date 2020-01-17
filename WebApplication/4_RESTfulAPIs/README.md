# Module 4: RESTful APIs with AWS Lambda and Amazon API Gateway

In this module you'll use API Gateway to expose the Lambda function you built in the previous module as a RESTful API. This API will be accessible on the public Internet. It will be secured using the Amazon Cognito user pool you created in the previous module. Using this configuration you will then turn your statically hosted website into a dynamic web application by adding client-side JavaScript that makes AJAX calls to the exposed APIs.

![Dynamic web app architecture](../images/restful-api-architecture.png)

The diagram above shows how the API Gateway component you will build in this module integrates with the existing components you built previously. The grayed out items are pieces you have already implemented in previous steps.

The static website you deployed in the first module already has a page configured to interact with the API you'll build in this module. The page at /ride.html has a simple map-based interface for requesting a unicorn ride. After authenticating using the /signin.html page, your users will be able to select their pickup location by clicking a point on the map and then requesting a ride by choosing the "Request Unicorn" button in the upper right corner.

This module will focus on the steps required to build the cloud components of the API, but if you're interested in how the browser code works that calls this API, you can inspect the [ride.js](../1_StaticWebHosting/website/js/ride.js) file of the website. In this case the application uses jQuery's [ajax()](https://api.jquery.com/jQuery.ajax/) method to make the remote request.

## Implementation Instructions

Each of the following sections provides an implementation overview and detailed, step-by-step instructions. The overview should provide enough context for you to complete the implementation if you're already familiar with the AWS Management Console or you want to explore the services yourself without following a walkthrough.

### 1. Create a New REST API
Use the Amazon API Gateway console to create a new API.
    
**:white_check_mark:Step-by-step directions**
    
1. In the AWS Management Console, click **Services** then select **API Gateway** under Networking & Content Delivery.

1. Choose **Create API**.

1. Select **REST**.

1. Select **New API** and enter `WildRydes-firstname-lastname` for the **API Name**.

1. Select `Edge optimised` selected in the **Endpoint Type** dropdown.

1. Choose **Create API**

    ![Create API screenshot](../images/create-api.png)

### 2. Create a Cognito User Pools Authorizer

#### Background
Amazon API Gateway can use the JWT tokens returned by Cognito User Pools to authenticate API calls. In this step you'll configure an authorizer for your API to use the user pool you created in [module 2](../2_UserManagement).

#### High-Level Instructions
In the Amazon API Gateway console, create a new Cognito user pool authorizer for your API. Configure it with the details of the user pool that you created in the previous module. You can test the configuration in the console by copying and pasting the auth token presented to you after you log in via the /signin.html page of your current website.

**:white_check_mark:Step-by-step directions**
1. Under your newly created API, choose **Authorizers**.

1. Choose **Create New Authorizer**.

1. Enter `WildRydes` for the Authorizer name.

1. Select **Cognito** for the type.

1. In the Region drop-down under **Cognito User Pool**, select the Region where you created your Cognito user pool in module 2 (by default the current region should be selected).

1. Enter `WildRydes-firstname-lastname` (or the name you gave your user pool) in the **Cognito User Pool** input.

1. Enter `Authorization` for the **Token Source**.

1. Choose **Create**.

    ![Create user pool authorizer screenshot](../images/create-user-pool-authorizer.png)

#### Verify your authorizer configuration

1. Open a new browser tab and visit `/ride.html` under your website's domain.

1. If you are redirected to the sign-in page, sign in with the user you created in the last module. You will be redirected back to `/ride.html`.

1. Copy the auth token from the notification on the `/ride.html`,

1. Go back to previous tab where you have just finished creating the Authorizer

1. Click **Test** at the bottom of the card for the authorizer.

1. Paste the auth token into the **Authorization Token** field in the popup dialog.

    ![Test Authorizer screenshot](../images/apigateway-test-authorizer.png)

1. Click **Test** button and verify that the response code is 200 and that you see the claims for your user displayed.

### 3. Create a new resource and method
Create a new resource called /ride within your API. Then create a POST method for that resource and configure it to use a Lambda proxy integration backed by the RequestUnicorn function you created in the first step of this module.

**:white_check_mark:Step-by-step directions**
1. In the left nav, click on **Resources** under your WildRydes-firstname-lastname API.

1. From the **Actions** dropdown select **Create Resource**.

1. Enter `ride` as the **Resource Name**.

1. Ensure the **Resource Path** is set to `ride`.

1. Select **Enable API Gateway CORS** for the resource.

1. Click **Create Resource**.

    ![Create resource screenshot](../images/create-resource.png)

1. With the newly created `/ride` resource selected, from the **Action** dropdown select **Create Method**.

1. Select `POST` from the new dropdown that appears, then **click the checkmark**.

    ![Create method screenshot](../images/create-method.png)

1. Select **Lambda Function** for the integration type.

1. Check the box for **Use Lambda Proxy integration**.

1. Select the Region you are using for **Lambda Region**.

1. Enter the name of the function you created in the previous module, `RequestUnicorn-firstname-lastname`, for **Lambda Function**.

1. Choose **Save**. Please note, if you get an error that you function does not exist, check that the region you selected matches the one you used in the previous module.

    ![API method integration screenshot](../images/api-integration-setup.png)

1. When prompted to give Amazon API Gateway permission to invoke your function, choose **OK**.

1. Choose on the **Method Request** card.

1. Choose the pencil icon next to **Authorization**.

1. Select the WildRydes Cognito user pool authorizer from the drop-down list, and click the checkmark icon.

    ![API authorizer configuration screenshot](../images/api-authorizer.png)

### 4. Deploy Your API
From the Amazon API Gateway console, choose Actions, Deploy API. You'll be prompted to create a new stage. You can use `prod` for the stage name.

**:white_check_mark:Step-by-step directions**
1. In the **Actions** drop-down list select **Deploy API**.

1. Select **[New Stage]** in the **Deployment stage** drop-down list.

1. Enter `prod` for the **Stage Name**.

1. Choose **Deploy**.

1. Note the **Invoke URL**. You will use it in the next section.

### 5. Update the Website Config
Update the /js/config.js file in your website deployment to include the invoke URL of the stage you just created. You should copy the invoke URL directly from the top of the stage editor page on the Amazon API Gateway console and paste it into the \_config.api.invokeUrl key of your sites /js/config.js file. Make sure when you update the config file it still contains the updates you made in the previous module for your Cognito user pool.

**:white_check_mark:Step-by-step directions**
 Edit the `config.js` file you have saved locally from module 2.

1. Open the config.js file in a text editor.

1. Update the **invokeUrl** setting under the **api** key in the config.js file. Set the value to the **Invoke URL** for the deployment stage your created in the previous section.

    An example of a complete `config.js` file is included below. Note, the actual values in your file will be different.

    ```JavaScript
    window._config = {
        cognito: {
            userPoolId: 'us-west-2_uXboG5pAb', // e.g. us-east-2_uXboG5pAb
            userPoolClientId: '25ddkmj4v6hfsfvruhpfi7n4hv', // e.g. 25ddkmj4v6hfsfvruhpfi7n4hv
            region: 'us-west-2' // e.g. us-east-2
        },
        api: {
            invokeUrl: 'https://rc7nyt4tql.execute-api.us-west-2.amazonaws.com/prod' // e.g. https://rc7nyt4tql.execute-api.us-west-2.amazonaws.com/prod,
        }
    };
    ```

1. Save your changes locally.

1. In the AWS Management Console, choose **Services** then select **S3** under Storage.

1. Choose your website bucket and then browse to the `js` key prefix.

1. Choose **Upload**.

1. Choose **Add files**, select the local copy of `config.js` and then click **Next**.

1. Choose **Next** without changing any defaults through the `Set permissions` and `Set properties` sections.

1. Choose **Upload** on the `Review` section.

## Implementation Validation

**Note:** It's possible that you will see a delay between updating the config.js file in your S3 bucket and when the updated content is visible in your browser. You should also ensure that you clear your browser cache before executing the following steps.

1. Visit `/ride.html` under your website domain.

1. If you are redirected to the sign in page, sign in with the user you created in the previous module.

1. After the map has loaded, click anywhere on the map to set a pickup location.

1. Choose **Request Unicorn**. You should see a notification in the right sidebar that a unicorn is on its way and then see a unicorn icon fly to your pickup location.
2. If you have any errors, go back through each step *carefully*. The error message may help you.

:tada: Congratulations, you have completed the Wild Rydes Web Application Workshop!

See this workshop's [cleanup guide](../9_CleanUp) for instructions on how to delete the resources you've created.
