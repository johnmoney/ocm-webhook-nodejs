# OCM Webhook Node.js

*OCM Webhook Node.js* is a demo of [Oracle Content Management webhook](https://docs.oracle.com/en/cloud/paas/content-cloud/solutions/use-webhooks.html) integrations built on Node.js, Express and Heroku. From the product documentation:

> Use webhooks to receive push notifications about content lifecycle events and content publishing events. The Webhooks application let you automatically receive information from Oracle Content and Experience to send to external applications through REST APIs.

This application persistently stores those notification events in a Postgres database in _channel_ (content publishing webhook) and _repository_ (content workflow webhook) tables. It can be used as a skeleton for creating your own application that integrates with OCM, for example:

- You might want OCM content to show up in the Shared Content library in Oracle Eloqua or as a content variation in Oracle Maxymiser. Use the content publishing webhook and call the external API when content is published or unpublished to the associated channel. 

- You might want to comment on a Jira ticket or send a Slack notification when content is approved or published. Use the content lifecycle webhook and call the external API when content moves through the lifecycle.

### Quick Start

In this example, we will store asset publish and unpublish events for an OCM channel.

#### 1. Clone the repository.

```console
$ git clone https://github.com/johnmoney/ocm-webhook-nodejs.git
Cloning into 'ocm-webhook-nodejs'...
Unpacking objects: 100% (94/94), done.

$ cd ocm-webhook-nodejs
```

#### 2. Install Heroku CLI and login.

Refer to the [Getting Started](https://devcenter.heroku.com/articles/getting-started-with-nodejs?singlepage=true) documentation to install the Heroku Command Line Interface.

```console
$ heroku login
heroku: Press any key to open up the browser to login or q to exit: 
Logging in... done
Logged in as your-email
```

#### 3. Deploy the app to Heroku.

The OCM webhook will call the URL generated from this step.

```console
$ heroku create your-app-name
Creating ⬢ your-app-name... done
https://your-app-name.herokuapp.com/ | https://git.heroku.com/your-app-name.git

$ git push heroku master
remote: Verifying deploy... done.
To https://git.heroku.com/ocm-apps-daily-summary.git
 * [new branch]      master -> master
```

#### 4. Provision a Postgres database.

```console
$ heroku addons:create heroku-postgresql:hobby-dev
Creating heroku-postgresql:hobby-dev on ⬢ your-app-name... free
Database has been created and is available
 ! This database is empty. If upgrading, you can transfer
 ! data from another database with pg:copy
Created postgresql as DATABASE_URL
Use heroku addons:docs heroku-postgresql to view documentation
```

#### 5. Create the tables in database.

```console
$ heroku pg:psql
--> Connecting to postgresql
psql (12.1, server 11.6 (Ubuntu 11.6-1.pgdg16.04+1))
SSL connection (protocol: TLSv1.2, cipher: ECDHE-RSA-AES256-GCM-SHA384, bits: 256, compression: off)
Type "help" for help.

your-app-name::DATABASE=> CREATE TABLE channel_event (event_id VARCHAR(36) PRIMARY KEY, event_type VARCHAR(32) NOT NULL, channel_id VARCHAR(50) NOT NULL, user_id VARCHAR(255) NOT NULL, event_time timestamp DEFAULT current_timestamp, assets jsonb);
CREATE TABLE

your-app-name::DATABASE=> CREATE TABLE repository_event (event_id VARCHAR(36) PRIMARY KEY, event_type VARCHAR(32) NOT NULL, repository_id VARCHAR(50) NOT NULL, user_id VARCHAR(255) NOT NULL, event_time timestamp DEFAULT current_timestamp, channel_id jsonb, asset jsonb);
CREATE TABLE
```

After the tables are created, exit the database prompt.

```console
your-app-name::DATABASE=> \q
```


#### 6. Setup the webhook in OCM.

From Admin > Integrations, select _Webhooks_ and click the Create button.

Select _Asset Publishing Webhook_ and fill in the following:

| Field | Value |
| --- | --- |
| **Name** | ocm-webhook-nodejs channel |
| **Publishing Channel** | _select one for testing_ |
| **Events** | _select individual events and Channel Asset Published and Channel Asset Unpublished_ |
| **Payload** | Detailed |
| **Target URL** | https://your-app-name.herokuapp.com/channel |

Save the Webhook.

### 7. Test the webhook.

Interactively view the Heroku logs to see events streaming to your console.

```console
$ heroku logs --source app --tail
```

In OCM, publish an asset to the channel previously selected. You should see the app log a successful event insert.

When you are done, exit the log.

```console
$ ^C
```

__That's it.__ You now have a persistent store of publish and unpublish events for the channel. You can view the database within Heroku by creating a [dataclip](https://data.heroku.com/dataclips) or extend the application for your own purposes.
