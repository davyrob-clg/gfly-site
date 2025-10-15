# UniGib AWS Project - Building a Flying Taxi Business for Gibraltar

  

This repo contains the code files for the web app and the Lambda function needed

  

## TL;DR

The project will demonstrate using AWS services as opposed to building EC2 resources individually.  The app is just a simple demonstrator creating a web application for a Gibraltar flying taxi service ride-sharing service called Gib Fly 

The original idea and code comes from  the original [Amazon workshop](https://aws.amazon.com/serverless-workshops)). 

The app uses IAM, Amplify, Cognito, Lambda, API Gateway and DynamoDB, with code stored in GitHub and incorporated into a CI/CD pipeline with Amplify.

  

The app will let you create an account and log in, then request a flying taxi by clicking on a map (powered by ArcGIS). The code can also be extended to build out more functionality.  **This is recommended for students !**

  

## Cost - Keep it Low!

All services used are eligible for the [AWS Free Tier](https://aws.amazon.com/free/). Outside of the Free Tier, there may be small charges associated with building the app (less than €1), but charges will continue to incur if you leave the app running.  So please ensure resources are shut down once completed.
 

## The Application Code

The application code is here in this repository.

  

## The Lambda Function Code

Here is the code for the Lambda function, originally taken from the [AWS workshop](https://aws.amazon.com/getting-started/hands-on/build-serverless-web-app-lambda-apigateway-s3-dynamodb-cognito/module-3/ ), and updated for Node 20.x:

  

```node

import { randomBytes } from 'crypto';

import { DynamoDBClient } from '@aws-sdk/client-dynamodb';

import { DynamoDBDocumentClient, PutCommand } from '@aws-sdk/lib-dynamodb';

  

const client = new DynamoDBClient({});

const ddb = DynamoDBDocumentClient.from(client);

  
// Taxi Fleet - need to change these values 
const fleet = [

{ Name: 'Taxi1', Color: 'White', Seats: '4' },

{ Name: 'Taxi2', Color: 'Blue', Seats: '2' },

{ Name: 'Taxi3', Color: 'Yellow', Seats: '2' },

];

  

export const handler = async (event, context) => {

if (!event.requestContext.authorizer) {

return errorResponse('Authorization not configured', context.awsRequestId);

}

  

const rideId = toUrlString(randomBytes(16));

console.log('Received event (', rideId, '): ', event);

  

const username = event.requestContext.authorizer.claims['cognito:username'];

const requestBody = JSON.parse(event.body);

const pickupLocation = requestBody.PickupLocation;

  

const unicorn = findUnicorn(pickupLocation);

  

try {

await recordRide(rideId, username, unicorn);

return {

statusCode: 201,

body: JSON.stringify({

RideId: rideId,

Unicorn: unicorn,

Eta: '30 seconds',

Rider: username,

}),

headers: {

'Access-Control-Allow-Origin': '*',

},

};

} catch (err) {

console.error(err);

return errorResponse(err.message, context.awsRequestId);

}

};

  

function findUnicorn(pickupLocation) {

console.log('Finding taxi for ', pickupLocation.Latitude, ', ', pickupLocation.Longitude);

return fleet[Math.floor(Math.random() * fleet.length)];

}

  

async function recordRide(rideId, username, unicorn) {

const params = {

TableName: 'Rides',

Item: {

RideId: rideId,

User: username,

Unicorn: unicorn,

RequestTime: new Date().toISOString(),

},

};

await ddb.send(new PutCommand(params));

}

  

function toUrlString(buffer) {

return buffer.toString('base64')

.replace(/\+/g, '-')

.replace(/\//g, '_')

.replace(/=/g, '');

}

  

function errorResponse(errorMessage, awsRequestId) {

return {

statusCode: 500,

body: JSON.stringify({

Error: errorMessage,

Reference: awsRequestId,

}),

headers: {

'Access-Control-Allow-Origin': '*',

},

};

}

```

  

## The Lambda Function Test Function

Here is the code used to test the Lambda function:

  

```json

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