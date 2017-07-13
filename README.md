## Parse Serverless

This is a template for running parse server using API Gateway and Lambda on AWS.

### Steps for running Parse Serverless on AWS
This guide assumes you have already [set up an AWS account](http://docs.aws.amazon.com/AmazonSimpleDB/latest/DeveloperGuide/AboutAWSAccounts.html) and have the latest version of the [AWS CLI](https://aws.amazon.com/cli/) installed.

1. run npm install

2. Run `npm run config -- --account-id="<accountId>" --bucket-name="<bucketName>" [--region="<region>" --function-name="<functionName>"]` to configure the example, eg. `npm run config -- --account-id="123456789012" --bucket-name="my-unique-bucket"`. This modifies `package.json`, `simple-proxy-api.yaml` and `cloudformation.yaml` with your account ID, bucket, region and function name (region defaults to `us-east-1` and function name defaults to `AwsServerlessExpressFunction`). If the bucket you specify does not yet exist, the next step will create it for you. This step modifies the existing files in-place; if you wish to make changes to these settings, you will need to modify `package.json`, `simple-proxy-api.yaml` and `cloudformation.yaml` manually.
3. Run `npm run setup` (Windows users: `npm run win-setup`) - this installs the node dependencies, creates an S3 bucket (if it does not already exist), packages and deploys your serverless Express application to AWS Lambda, and creates an API Gateway proxy API.

4. After the setup command completes, open the AWS CloudFormation console https://console.aws.amazon.com/cloudformation/home and switch to the region you specified. Select the `AwsServerlessExpressStack` stack, then click the `ApiUrl` value under the __Outputs__ section - this will open a new page with your running API. The API index lists the resources available in the example Express server (`app.js`), along with example `curl` commands.

5. Modify `lambda.js` to import your own server configuration (eg. change `require('./app')` to `require('./server')`). You will need to ensure you export your app configuration from the necessary file (eg. `module.exports = app`). This library takes your app configuration and listens on a Unix Domain Socket for you, so you can remove your call to `app.listen()` (if you have a `server.listen` callback, you can provide it as the second parameter in the `awsServerlessExpress.createServer` method).

6. Run `npm run package-deploy` (Windows users: `npm run win-package-deploy`) to package and deploy your application.

To perform a basic, local simulation of API Gateway and Lambda with your Node server, update `api-gateway-event.json` with some values that are valid for your server (`httpMethod`, `path`, `body` etc.) and run `npm run local`. AWS Lambda uses NodeJS 4.3 LTS, and it is recommended to use the same version for testing purposes.

If you need to make modifications to your API Gateway API, modify `simple-proxy-api.yaml` and run `npm run package-deploy`. If your API requires CORS, be sure to modify the two `options` methods defined in the Swagger file, otherwise you can safely remove them. To modify your other AWS assets, make your changes to `cloudformation.yaml` and run `npm run package-deploy`. Alternatively, you can manage these assets via the AWS console.

### Saving your first object

Now that you're running Parse Server, it is time to save your first object. We'll use the [REST API](http://docs.parseplatform.org/rest/guide), but you can easily do the same using any of the [Parse SDKs](http://parseplatform.org/#sdks). Run the following:

```bash
curl -X POST \
-H "X-Parse-Application-Id: APPLICATION_ID" \
-H "Content-Type: application/json" \
-d '{"score":1337,"playerName":"Sean Plott","cheatMode":false}' \
http://localhost:1337/parse/classes/GameScore
```

You should get a response similar to this:

```js
{
  "objectId": "2ntvSpRGIK",
  "createdAt": "2016-03-11T23:51:48.050Z"
}
```

You can now retrieve this object directly (make sure to replace `2ntvSpRGIK` with the actual `objectId` you received when the object was created):

```bash
$ curl -X GET \
  -H "X-Parse-Application-Id: APPLICATION_ID" \
  http://localhost:1337/parse/classes/GameScore/2ntvSpRGIK
```
```json
// Response
{
  "objectId": "2ntvSpRGIK",
  "score": 1337,
  "playerName": "Sean Plott",
  "cheatMode": false,
  "updatedAt": "2016-03-11T23:51:48.050Z",
  "createdAt": "2016-03-11T23:51:48.050Z"
}
```

Keeping tracks of individual object ids is not ideal, however. In most cases you will want to run a query over the collection, like so:

```
$ curl -X GET \
  -H "X-Parse-Application-Id: APPLICATION_ID" \
  http://localhost:1337/parse/classes/GameScore
```
```json
// The response will provide all the matching objects within the `results` array:
{
  "results": [
    {
      "objectId": "2ntvSpRGIK",
      "score": 1337,
      "playerName": "Sean Plott",
      "cheatMode": false,
      "updatedAt": "2016-03-11T23:51:48.050Z",
      "createdAt": "2016-03-11T23:51:48.050Z"
    }
  ]
}

```

To learn more about using saving and querying objects on Parse Server, check out the [Parse documentation](http://docs.parseplatform.org).

### Connect your app to Parse Server

Parse provides SDKs for all the major platforms. Refer to the Parse Server guide to [learn how to connect your app to Parse Server](https://github.com/parse-community/parse-server/wiki/Parse-Server-Guide#using-parse-sdks-with-parse-server).

## Running Parse Server elsewhere

Once you have a better understanding of how the project works, please refer to the [Parse Server wiki](https://github.com/parse-community/parse-server/wiki) for in-depth guides to deploy Parse Server to major infrastructure providers. Read on to learn more about additional ways of running Parse Server.
# Documentation

The full documentation for Parse Server is available in the [wiki](https://github.com/parse-community/parse-server/wiki). The [Parse Server guide](http://docs.parseplatform.org/parse-server/guide/) is a good place to get started. If you're interested in developing for Parse Server, the [Development guide](http://docs.parseplatform.org/parse-server/guide/#development-guide) will help you get set up.

## Configuration

Parse Server can be configured using the following options. You may pass these as parameters when running a standalone `parse-server`, or by loading a configuration file in JSON format using `parse-server path/to/configuration.json`. If you're using Parse Server on Express, you may also pass these to the `ParseServer` object as options.

For the full list of available options, run `parse-server --help`.

#### Basic options

* `appId` **(required)** - The application id to host with this server instance. You can use any arbitrary string. For migrated apps, this should match your hosted Parse app.
* `masterKey` **(required)** - The master key to use for overriding ACL security.  You can use any arbitrary string. Keep it secret! For migrated apps, this should match your hosted Parse app.
* `databaseURI` **(required)** - The connection string for your database, i.e. `mongodb://user:pass@host.com/dbname`. Be sure to [URL encode your password](https://app.zencoder.com/docs/guides/getting-started/special-characters-in-usernames-and-passwords) if your password has special characters.
* `port` - The default port is 1337, specify this parameter to use a different port.
* `serverURL` - URL to your Parse Server (don't forget to specify http:// or https://). This URL will be used when making requests to Parse Server from Cloud Code.
* `cloud` - The absolute path to your cloud code `main.js` file.
* `push` - Configuration options for APNS and GCM push. See the [Push Notifications quick start](http://docs.parseplatform.org/parse-server/guide/#push-notifications_push-notifications-quick-start).

## Node.js version

This example was written against Node.js version 6.10