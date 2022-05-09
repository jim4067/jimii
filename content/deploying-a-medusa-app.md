

In the previous article [creating a medusa app](AVIYEL LINKE HERE), we created a Medusa app, from scratch. In this article we will deploy the medusa Back-End of our application to Azure.  For this we will use Azure app services.



## setting up medusa

First we will make sure that our environmental variables are setup correctly. Open up the medusa app you created last time using your favorite editor. We will first check our `.env` file to make sure we have all our secret keys and variables correctly defined and named.(You have no idea how many prod builds have been brought down by typo errors). So first confirm whether everything is correctly named. After that we import these environmental secrets in out `medusa-config.js` file. 

After checking that everything is okay, your medusa config file should resemble this 

```shell
const DB_USERNAME = process.env.DB_USERNAME;
const DB_PASSWORD = process.env.DB_PASSWORD;
const DB_HOST = process.env.DB_HOST;
const DB_PORT = process.env.DB_PORT;
const DB_DATABASE = process.env.DB_DATABASE;

const DATABASE_URL = `postgres://${DB_USERNAME}:${DB_PASSWORD}@${DB_HOST}:${DB_PORT}/${DB_DATABASE}`;

module.exports = {
  projectConfig: {
    redis_url: REDIS_URL,
    database_url: DATABASE_URL,
    database_type: "postgres",
    store_cors: STORE_CORS,
    admin_cors: ADMIN_CORS,
    database_extra: { ssl: { rejectUnauthorized: false } }
  },
  plugins,
};
```

This is well and okay if you are planning to never make changes to the store. But that's usually not the case. It is better to separate and use different configurations for production, development and testing. When making changes you don't want to unnecessarily update your production database or deploy a feature that hasn't been thoroughly tested. So you could update your config file by adding the following lines. Next add [cross-env](https://www.npmjs.com/package/cross-env) package to set the environmental variables inline when running node commands. 

```shell
if (process.env.NODE_ENV === "development") {
   // your dev config goes here
  //...
}
if (process.env.NODE_ENV === "production") {
   // your prod config goes here
  //...
}
```

Update your npm scripts

```shell
...
"scripts": {
    "serve": "medusa start",
    "start": "medusa migrations run && medusa develop",
    "build": "babel src -d dist --extensions \".ts,.js\""
}
...
```

if you did the setup and are trying to separate your production config from dev config, you might have something that resembles

`"dev": "cross-env NODE_ENV=development medusa start",` depending on how you decide to configure your scripts.

Add the node engine field in the `package.json` file to include since azure app services only supports version 14 and 16 of node on Linux

```shell
...
"engines": {
    "node": "14.x"
}
...
```

After the changes stage, commit and push your changes to your remote repo. 

```shell
git add .
git commit -m "chore: deploy config setup"
git push origin master
```



## Azure

Sign into your Azure account and go to the [Azure portal]([Microsoft Azure](https://portal.azure.com/#home)) and go to the app services section which is Microsoft's offering of a PAAS

Once in the app services portal click, `create app service`. I am assuming that you have some familiarity with azure. If not, you may have something that looks like this in the basics tab. 



There are six-steps in creating an web application on Azure app services. In the first tab/step you fill in the detail for your app, like name, the subscription where you want to be billed etc... You should have something that resembles this

# INSERT IMAGE HERE

In the deployment tab select the repo that you hosted you medusa store code. 

In the tabs that follow e.g networking and monitoring, the defaults should be okay left as they are. Review, create and deploy your medusa store.



Click create. Once we have our web app created we need to create our Postgres database. In the azure portal search bar, type Postgres and select the first option. We will then create a Postgres database from here. 
















