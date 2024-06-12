# Allowing Xero to Post Events to Zoho CRM

## Step 1: Register an App in Xero
First, register an application at the [Xero Developer's Portal](https://developer.xero.com/app/manage). Click the 'New App' button.

![Screenshot: Xero Developer's Portal - App Registration](https://previewengine-accl.zoho.com/image/WD/u3yseef1fa670c9964629b908ffe7a413cc15?version=1.0&amp;width=2046&amp;height=1536)

Select the 'Web' type for the application.

![Screenshot: Xero Application Type Choose](https://previewengine-accl.zohoexternal.com/image/WD/u3yse6626a8db670f45fd865e509e69ccba73)

## Step 2: Configure Redirect URI
Set the redirect URI to `https://dre.zoho.com.au/delugeauth/callback` to ensure that Zoho CRM can communicate with Xero.

![Screenshot: Set Redirect URI](https://previewengine-accl.zohoexternal.com/image/WD/u3ysea41d58e44002460aa02e12ee66847716?version=1.0&width=2046&height=1536)

Click the 'Create' button.

## Step 3: Generate Client Secret
After creating the app, navigate to the configuration page on Xero's portal to generate a client secret.

![Screenshot: Generate Client Secret](https://previewengine-accl.zohoexternal.com/image/WD/u3ysed6280e9d2cf144999667c68adb98807a?version=1.0&width=2046&height=1536)

## Step 4: Create Custom Connection in Zoho CRM
Navigate to **Setup > Automation > Connections** in Zoho CRM, select **Custom Services**, and click **Create Service**. Fill in the necessary details for the service.

Details to be entered:
- **Service Name**: Xero (The Service Link Name will be automatically populated.)
- **Authentication Type**: OAuth2
- **Parameter Type**: Header
- **Grant Type**: Authorization Code
- **Client ID, Client Secret**: Enter the details obtained in previous steps.
- **Authorize URL**: `https://login.xero.com/identity/connect/authorize`
- **Access Token URL**: `https://identity.xero.com/connect/token`
- **Refresh Token URL**: `https://identity.xero.com/connect/token`
- **Scopes**: Add 'offline_access' to ensure a stable connection.

![Screenshot: Create Custom Service in Zoho CRM](https://previewengine-accl.zohoexternal.com/image/WD/u3yseac8c4cd3a9db4a76aff449cf2314bf1b?version=1.0&width=2046&height=1536)

## Step 5: Create and Connect
In the 'Create Connection' page:
- Enter a Connection Name.
- Choose all the required scopes and click 'Create and Connect'.
- This action will redirect you to the Xero login page. Click 'Allow access' to authorize the connection.

![Screenshot: Create Custom Connection in Zoho CRM](https://previewengine-accl.zohoexternal.com/image/WD/u3yse387e4efb3f5d4f4b95a3a8995ac07c86)

## Step 6: Create an API Endpoint in Zoho CRM
Under the Developer Hub, create a standalone function with the following Deluge script to securely handle webhook payloads:

![Screenshot: API Creation in CRM](https://previewengine-accl.zohoexternal.com/image/WD/u3yse716c16e4b5a6472298759b4bd5dd5f54)

Configure the API parameters as follows:
- **Parameter Type**: String
- **Parameter Name**: crmAPIRequest

![Screenshot: API Parameter Configuration](https://previewengine-accl.zohoexternal.com/image/WD/u3yseb006888baf7442ba8f6b1dc8f833fee8)

Here is the Deluge script to validate the webhook signature:
```deluge
response = Map();
crmAPIRequestMap = crmAPIRequest.toMap();
payload = crmAPIRequestMap.get("body");
headers = crmAPIRequestMap.get("headers");
xero_signature = headers.get("x-xero-signature");
webhook_signing_key = {YOUR_WEBHOOK_SIGNING_KEY};
computed_hash = zoho.encryption.hmacsha256(webhook_signing_key,payload,"base64");

if(xero_signature == computed_hash) {
    info "Signature is valid";
    response.put("status", "success");
    response.put("status_code", 200);
    body = Map();
    response.put("body", body);
} else {
    info "Signature is invalid, respond with 401 Unauthorized";
    response.put("status", "unauthorized");
    response.put("status_code", 401);
    body = Map();
    response.put("body", body);
}
return {"crmAPIResponse":response};
```

After saving and exiting the function edit page, find your function listed under the function list. Click the '...' button next to your function and select the 'REST API' option.

![Screenshot: Select REST API Option](https://previewengine-accl.zohoexternal.com/image/WD/u3ysebb9ac05262ad4caa80665d352ebbbc1d?version=1.0&width=2046&height=1536)

Turn on API key authentication, copy the API link, and click 'Save'. This link will be used to connect your Zoho CRM with Xero's webhook.

![Screenshot: API Key and Link](https://previewengine-accl.zohoexternal.com/image/WD/u3yse0a8d716cae85494b8998fb338a4f8dcc?version=1.0&width=2046&height=1536)

## Step 7: Create a Webhook in Xero
Navigate to the Xero app you created on the Xero developer platform. Go to the 'Webhook' settings and select a module that you wish to notify the CRM about. Paste the API path from Zoho CRM as the Delivery URL.

![Screenshot: Webhook Configuration in Xero](https://previewengine-accl.zohoexternal.com/image/WD/u3yse6354947b79db4c03a47b920d763459e0?version=1.0&width=2046&height=1536)

![Screenshot: Webhook Configuration with Form Filled in Xero](https://previewengine-accl.zohoexternal.com/image/WD/u3yse05e67ffbe280472ca72a4ef4fbab3f8d?version=1.0&width=2046&height=1536)

Click 'Save' to confirm the settings and then copy the webhooks key provided by Xero.

![Screenshot: Webhook Configuration with Form Filled in Xero](https://previewengine-accl.zohoexternal.com/image/WD/u3ysebf42e3fca91443e6a0853e01401f0256?version=1.0&width=2046&height=1536)

## Step 8: Modify the API Function
Return to your Zoho CRM and navigate to the previously created standalone function. Replace the placeholder `{YOUR_WEBHOOK_SIGNING_KEY}` in your Deluge script with the actual signing key you copied from Xero. This key is used to securely validate incoming webhooks from Xero.

```deluge
webhook_signing_key = 'Your_Actual_Signing_Key_Here';
```

## Step 9: Send Intent to Receive
In Xero's webhook settings, click the 'Send Intent to Receive' button. This action sends a test webhook to your Zoho CRM endpoint.

![Screenshot: Send Intent to Receive](https://previewengine-accl.zohoexternal.com/image/WD/u3ysee3a4f59e2ad7460e96e7de7f986a5941?version=1.0&width=2046&height=1536)

After this showing OK, the connection is formally established.

## Step 10: Monitor and Maintain
Regularly monitor the webhook responses and maintain the API connection. Check for any errors in the logs and ensure the webhook's security by periodically updating the signing key and validating the payloads. This maintenance is crucial for the integrity and reliability of the automated data exchange between Xero and Zoho CRM.
