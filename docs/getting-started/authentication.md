# Authentication

The v4 google sheets API requires some level of authentication to make any requests.<br>
_(v3 used to allow totally anonymous requests for read-only access to public sheets)_

You have several options for how you want to connect, but most projects should use a service account.

- [Service Account](#service-account) - connects as a specific "bot" user generated by google for your application
- [API-key](#api-key) - only identifies your application, provides read-only access
- [OAuth](#oauth) - connect on behalf of a specific user (coming soon!)

**👉 BUT FIRST -- Set up your google project & enable the sheets API 👈**
1. Go to the [Google Developers Console](https://console.developers.google.com/)
2. Select your project or create a new one (and then select it)
3. Enable the Sheets API for your project
  - In the sidebar on the left, select **APIs & Services > Library**
  - Search for "sheets"
  - Click on "Google Sheets API"
  - click the blue "Enable" button

## 🤖 Service Account (recommended) :id=service-account
**Connect as a bot user that belongs to your app**

This is a 2-legged oauth method and designed to be "an account that belongs to your application instead of to an individual end user".
Use this for an app that needs to access a set of documents that you have full access to, or can at least be shared with your service account.
([read more](https://developers.google.com/identity/protocols/OAuth2ServiceAccount))

You may also grant your service account "domain-wide delegation" which enables it to impersonate any user within your org. This can be helpful if you need to connect on behalf of users only within your organization. (support + example coming soon)

__Setup Instructions__

1. Follow steps above to set up project and enable sheets API
2. Create a service account for your project
  - In the sidebar on the left, select **APIs & Services > Credentials**
  - Click blue "+ CREATE CREDENITALS" and select "Service account" option
  - Enter name, description, click "CREATE"
  - You can skip permissions, click "CONTINUE"
  - Click "+ CREATE KEY" button
  - Select the "JSON" key type option
  - Click "Create" button
  - your JSON key file is generated and downloaded to your machine (__it is the only copy!__)
  - click "DONE"
  - note your service account's email address (also available in the JSON key file)
3. Share the doc (or docs) with your service account using the email noted above

!> Be careful - never check your API keys / secrets into version control (git)

You can now use this file in your project to authenticate as your service account. If you have a config setup involving environment variables, you only need to worry about the `client_email` and `private_key` from the JSON file. For example:

```javascript
const creds = require('./config/myapp-1dd646d7c2af.json'); // the file saved above
const doc = new GoogleSpreadsheet('<YOUR-DOC-ID>');
await doc.useServiceAccountAuth(creds);

// or preferably, loading that info from env vars / config instead of the file
await doc.useServiceAccountAuth({
  client_email: process.env.GOOGLE_SERVICE_ACCOUNT_EMAIL,
  private_key: process.env.GOOGLE_PRIVATE_KEY,
});
```

**SPECIAL NOTE FOR HEROKU USERS**
Getting the key saved/retrieved properly can be a little tricky. Here's one way to get it to work
1. Save the private key to a new text file
2. Replace `\n` with actual line breaks
3. Replace `\u003d` with `=`
4. run `heroku config:add GOOGLE_PRIVATE_KEY="$(cat yourfile.txt)"` in your terminal


## 🔑 API Key :id=api-key
**read-only access of public docs**

Google requires this so they can at least meter your usage of their API.

__Setup Instructions__
1. Follow steps above to set up project and enable sheets API
2. Create an API key for your project
  - In the sidebar on the left, select **Credentials**
  - Click blue "+ CREATE CREDENITALS" and select "API key" option
  - Copy the API key
3. OPTIONAL - click "Restrict key" on popup to set up restrictions
  - Click "API restrictions" > Restrict Key"
  - Check the "Google Sheets API" checkbox
  - Click "Save"

!> Be careful - never check your API keys / secrets into version control (git)

```javascript
const doc = new GoogleSpreadsheet('<YOUR-DOC-ID>');
doc.useApiKey(process.env.GOOGLE_API_KEY);
```

## 👨‍💻 OAuth 2.0 :id=oauth
**connect on behalf of a user with an Oauth token**

!> Support and instructions coming soon