# Lesson 3: Offer Your Service's Insights via an API

Goal: Create a publicly accessible, unauthenticated RESTful API to query for contest winners.

### Step 1: Review Your `serverless.yml`

Declared in your `serverless.yml` is an API Gateway (as HTTP events sources of your functions), two API functions (scores and contributions), and individual IAM Roles for each function.

Deploying this YAML creates your public RESTful endpoint that allows you to GET the winners from the web service.  If desired, you could require authentication via IAM or an arbitrary authentication Lambda attached to your API Gateway but these options are considered beyond the scope of the workshop.  Note that API Gateway allows you to export an auto-generated SDK for your endpoint in a variety of different languages as well as to export a swagger definition for your API.  Caching can be enabled, as well as API-wide of API-key specific RPS thresholds and a variety of other features.

More details can be found here: https://serverless.com/framework/docs/providers/aws/events/apigateway/

### Step 2: Deploy Your API

#### OS X

```sh
pushd Lesson3_PublicEndpointToAccessView/winner-api
serverless deploy -s $STAGE
popd
```

#### Windows

```bat
pushd Lesson3_PublicEndpointToAccessView\winner-api
serverless deploy -s %STAGE%
popd
```

### Step 3: Confirm Your Deployment

* Look in the AWS console under AWS API Gateway.
* Copy the URL you find there under Stages.  That is the root of your endpoint.
* Paste that URL into your browser and add `/scores?role=creator&limit=2` to the end of it.

Here's an example of what the URL looks like for an API Gateway endpoint:

```
https://zzzzzzzzzz.execute-api.us-west-2.amazonaws.com/XXXX/scores?role=creator&limit=2
```

Where `zzzzzzzzzzzz` is an ID generated by AWS for your API and `XXXX` will be the name of your stage.

The `role` query parameter can have a value of either `creator` or `photographer` depending on which role you want to list a winner for.

The optional `limit` query parameter sets how many results to query for and return.  The default value is one.

### Extra Credit

```sh
https://zzzzzzzzzz.execute-api.us-west-2.amazonaws.com/XXXX/contributions
```

The contributions path above returns product IDs that have contributions.  As an exercise, alter the winnerApi.js code to also return the lastEventId as well as the productId, then re-deploy.  (If you did Lesson 6, rather than Lesson 2, skip this exercise.)

Any change to the codebase or resource settings requires a re-deployment of your service.  Since we've changed code, let's redeploy:

#### OS X

```sh
pushd Lesson3_PublicEndpointToAccessView/winner-api
serverless deploy function -f contributions -s $STAGE
popd
```

#### Windows

```bat
pushd Lesson3_PublicEndpointToAccessView\winner-api
serverless deploy function -f contributions -s %STAGE%
popd
```

The additional `-f` flag specifies to only deploy the code for the specified Lambda, in order to update it and is unnecessary in practice.  A `serverless deploy -s [$STAGE/%STAGE%]` would have been sufficient.

### Extra, Extra Credit

Another question you could answer is: how many purchases were made of each product?  Examine the Contributions table.  You can see that it records a creatorScore for each product.  This number tracks all the purchases made of the product since its creation (by the creator).

How would you provide this information through the API?

You deployed lambdas to handle requests to the /contributions and /scores paths.  How would you adapt the winnerApi.js code for a third lambda that would service a /bestseller path?

Note that this gateway can scale massively.  To handle a lot more traffic (thousands of TPS) you'd want to ensure that your available AWS account limits could handle the traffic and appropriately increase the read IOPs on your DynamoDB table.

You should expect about a 50ms P50 and 130ms P99 from this endpoint under any scale.  The results here will be accurate within about a second of the purchase activity from a customer on the core stream.  We will discuss the distinction between latency and staleness if we have not already done so.

### Extra, Extra, Extra Credit

In the previous extra credit, you took advantage of exist resources and modified existing code.  For this exercise, try implementing the above from scratch.  Feel free to do some copying and pasting but create a separate project (i.e. in a new folder, run `sls create --template aws-nodejs`) and start building it out to accomplish the same result.  You'll need a DynamoDB table, a Lambda, and a subscription of that Lambda to the Stream from Lesson 1 (to trim horizon).  Make the accompanying API if you like or verify that the database is correct.  Either way, being able to build a project that can accumulate/interpret the right answer is good evidence that you have absorbed the technology well enough to start implementing interesting serverless systems.  Congratulations and thank you for taking the time and energy to engage with this exercise!

