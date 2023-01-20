# Module 4: RESTful APIs with AWS Lambda and Amazon API Gateway

In this module you'll use API Gateway to expose the Lambda function you built in the previous module as a RESTful API. This API will be accessible on the public Internet. It will be secured using the Amazon Cognito user pool you created in the User Management module. Using this configuration you will then turn your statically hosted website into a dynamic web application by adding client-side JavaScript that makes AJAX calls to the exposed APIs.

![image](https://user-images.githubusercontent.com/115881685/209072995-6b207d92-c626-49c4-89b0-6256d7458bf7.png)

The diagram above shows how the API Gateway component you will build in this module integrates with the existing components you built previously. The grayed out items are pieces you have already implemented in previous steps.

The static website you deployed in the first module already has a page configured to interact with the API you'll build in this module. The page at /ride.html has a simple map-based interface for requesting a unicorn ride. After authenticating using the /signin.html page, your users will be able to select their pickup location by clicking a point on the map and then requesting a ride by choosing the "Request Unicorn" button in the upper right corner.

This module will focus on the steps required to build the cloud components of the API, but if you're interested in how the browser code works that calls this API, you can inspect the ride.js file of the website. In this case the application uses jQuery's ajax() method to make the remote request.

## Implementation Instructions

❗ Ensure you've completed the Serverless Backend step before beginning the workshop.

Each of the following sections provides an implementation overview and detailed, step-by-step instructions. The overview should provide enough context for you to complete the implementation if you're already familiar with the AWS Management Console or you want to explore the services yourself without following a walkthrough.

## 1. Create a New REST API

Use the Amazon API Gateway console to create a new API named WildRydes.

## ✅ Step-by-step directions

1. Go to the Amazon API Gateway Console

2. Choose Create API.

3. Select REST, New API and enter WildRydes for the API Name.

4. Select Edge optimized from the Endpoint Type dropdown. Note: Edge optimized are best for public services being accessed from the Internet. Regional endpoints are typically used for APIs that are accessed primarily from within the same AWS Region. Private APIs are for internal services inside of an Amazon VPC.

5. Choose Create API

![image](https://user-images.githubusercontent.com/115881685/209073568-f242ba49-41ce-4bd9-9aff-e9c7ed3bb36e.png)

##### 2. Create a Cognito User Pools Authorizer

##### Background

Amazon API Gateway can use the JWT tokens returned by Cognito User Pools to authenticate API calls. In this step you'll configure an authorizer for your API to use the user pool you created in User Management.

##### High-Level Instructions

##### ✅ Step-by-step directions

1. Under your newly created API, choose Authorizers.

2. Choose Create New Authorizer.

3. Enter WildRydes for the Authorizer name.

4. Select Cognito for the type.

5. In the Region drop-down under Cognito User Pool, select the Region where you created your Cognito user pool in the User Management module (by default the current region should be selected).

6. Enter WildRydes (or the name you gave your user pool) in the Cognito User Pool input.

7. Enter Authorization for the Token Source.

8. Choose Create.

![image](https://user-images.githubusercontent.com/115881685/209074089-2d808aee-4af0-421f-bb4a-f5b10e07107e.png)

##### Verify your authorizer configuration

##### ✅ Step-by-step directions

1. Open a new browser tab and visit /ride.html under your website's domain.

2. If you are redirected to the sign-in page, sign in with the user you created in the last module. You will be redirected back to /ride.html.

3. Copy the auth token from the notification on the /ride.html,

4. Go back to previous tab where you have just finished creating the Authorizer

5. Click Test at the bottom of the card for the authorizer.

6. Paste the auth token into the Authorization Token field in the popup 
dialog.

![image](https://user-images.githubusercontent.com/115881685/209074404-a30beea2-16b9-4e6a-97f1-8d8d34fac7ef.png)

7. Click Test button and verify that the response code is 200 and that you see the claims for your user displayed.

##### 3. Create a new resource and method

Create a new resource called /ride within your API. Then create a POST method for that resource and configure it to use a Lambda proxy integration backed by the RequestUnicorn function you created in the first step of this module.

##### ✅ Step-by-step directions

1. In the left nav, click on Resources under your WildRydes API.

2. From the Actions dropdown select Create Resource.

3. Enter ride as the Resource Name.

4. Ensure the Resource Path is set to ride.

5. Select Enable API Gateway CORS for the resource.

6. Click Create Resource.

![image](https://user-images.githubusercontent.com/115881685/209074791-258c2869-d332-410b-a295-31532bfb1483.png)

7. With the newly created /ride resource selected, from the Action dropdown select Create Method.

8. Select POST from the new dropdown that appears, then click the checkmark.

![image](https://user-images.githubusercontent.com/115881685/209074994-27482178-174d-40f8-8292-f70ec164baf7.png)

9. Select Lambda Function for the integration type.

10. Check the box for Use Lambda Proxy integration.

11. Select the Region you are using for Lambda Region.

12. Enter the name of the function you created in the previous module, RequestUnicorn, for Lambda Function.

13. Choose Save. Please note, if you get an error that you function does not exist, check that the region you selected matches the one you used in the previous module.

![image](https://user-images.githubusercontent.com/115881685/209075196-4bbe406e-1df5-4ac4-a8b9-fb7aa2cff075.png)

14. When prompted to give Amazon API Gateway permission to invoke your function, choose OK.

15. Choose on the Method Request card.

16. Choose the pencil icon next to Authorization.

17. Select the WildRydes Cognito user pool authorizer from the drop-down list, and click the checkmark icon.

![image](https://user-images.githubusercontent.com/115881685/209075343-5f9ca067-0ee0-4ae2-ae6b-89df67bd70bf.png)

## 4. Deploy Your API

From the Amazon API Gateway console, choose Actions, Deploy API. You'll be prompted to create a new stage. You can use prod for the stage name.

## ✅ Step-by-step directions

1. In the Actions drop-down list select Deploy API.

2. Select [New Stage] in the Deployment stage drop-down list.

3. Enter prod for the Stage Name.

4. Choose Deploy.

5. Note the Invoke URL. You will use it in the next section.

## 5. Update the Website Config

Update the /js/config.js file in your website deployment to include the invoke URL of the stage you just created. You should copy the invoke URL directly from the top of the stage editor page on the Amazon API Gateway console and paste it into the _config.api.invokeUrl key of your sites /js/config.js file. Make sure when you update the config file it still contains the updates you made in the previous module for your Cognito user pool.

## ✅ Step-by-step directions

1. On your Cloud9 development environment open js/config.js

2. Update the invokeUrl setting under the api key in the config.js file. Set the value to the Invoke URL for the deployment stage your created in the previous section. An example of a complete config.js file is included below. Note, the actual values in your file will be different.

```
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

3. Save the modified file making sure the filename is still config.js.

4. Commit the changes to your git repository:

```
$ git add js/config.js   
$ git commit -m "configure api invokeURL"
$ git push
...
Counting objects: 4, done.
Compressing objects: 100% (4/4), done.
Writing objects: 100% (4/4), 422 bytes | 422.00 KiB/s, done.
Total 4 (delta 3), reused 0 (delta 0)
To https://git-codecommit.us-east-1.amazonaws.com/v1/repos/wildrydes-site
c15d5d5..09f1c9a  master -> master
```
  Amplify Console should pick up the changes and begin building and deploying your web application. Watch it to verify the completion of the deployment.
  
## Implementation Validation

## ✅ Step-by-step directions

1. Visit /ride.html under your website domain.

2. If you are redirected to the sign in page, sign in with the user you created in the previous module.

3. After the map has loaded, click anywhere on the map to set a pickup location.

4. Choose Request Unicorn. You should see a notification in the right sidebar
that a unicorn is on its way and then see a unicorn icon fly to your pickup
location.

## ⭐ Recap

🔑 Amazon API Gateway is a fully managed service that makes it easy for developers to create, publish, maintain, monitor, and secure APIs at any scale. You can easily plug in Authorization via Amazon Cognito and backends such as AWS Lambda to create completely serverless APIs.

🔧 In this module you've used API Gateway to provide a REST API to the Lambda function created in the previous module. From there you've updated the website to use the API endpoint so that you can request rides and the information about the ride is saved in the DynamoDB table created earlier.

⭐ Congratulations, you have completed the Wild Rydes Web Application Workshop! Check out our other workshops covering additional serverless use cases.

## Next

Delete all recoueses in this workshop in order to avoid charges.
