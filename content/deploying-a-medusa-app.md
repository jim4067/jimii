In the previous article [creating a medusa app](AVIYEL LINKE HERE), we created a Medusa app, from scratch. In this article we will deploy the medusa Back-End of our application to Azure.  For this we will use Digital Ocean.

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

## Digital Ocean

Sign into your Digital Ocean account and after the login you will be redirected to the default first-project page. 

In this page click the upper right green create drop-down button and select apps, the third option. 

![](/home/jimii/Documents/webcode/jimii/static/deploying-medusa-app/new-proj-digital-ocean-landing.jpg)

You will be taken to a new repository to configure the new application that you want to create. Make sure that you have your online repo service provider i.e GitHub for my case, and Digital Ocean account linked. If not clicking mange access should pull up the GitHub authorization page to link your two accounts together.

After authorizing your GitHub, select the repo where you pushed the medusa back-end code. Clicking next will take you to a resources page. This is where you can manage your serverless functions, databases... etc.  We want to add a dev database but if you are deploying for prod, the chose that. You also have the option to attach a previously created database. 

![](/home/jimii/Documents/webcode/jimii/static/deploying-medusa-app/DO-db-resource-attaching.jpg)

  

We now need to configure the environmental variables, that we created earlier. These are just the values in your `.env` file.

```shell
DB_USERNAME=${db.USERNAME}
DB_PASSWORD=${db.PASSWORD}
DB_HOST=${db.HOSTNAME}
DB_PORT=${db.PORT}
DB_DATABASE=${db.DATABASE}
REDIS_URL=${redis.DATABASE_URL}
JWT_SECRET=your-jwt-secret
COOKIE_SECRET=your-cookie-secret
```

After configuring your environmental variables, you now need to configure the name of your app and the region you want it to run from.

Make sure that all these variables are valid otherwise your build will fail.

Review your app and check the plan you want to be billed with and launch the app

We will now setup our Redis Database. 

Click on the drop-down create menu -> **Databases** -> select the data-center region you want to deploy it -> **select Redis** -> and finalize with the plan you want to be billed. 

Finally name your database and create the DB cluster. If you chose to name your database something else then update the env vars using accordingly i.e this line `REDIS_URL=${redis.DATABASE_URL}`.

On the left side navigation, go to apps and select the medusa app you deployed, to add the redis DB you created. Click the `create drop-down` and select `Create/Attach Database`, select previously created database and attach it to your app.

![](/home/jimii/Pictures/Screenshots/Screenshot_20220512233130.png)



Finally re-build and re-deploy your medusa backend. 



If for any reason your deploy should fail, in your app click on the deployment link to get the logs and see what went wrong. 

If everything goes according to plan, you should be able to visit your app 

@ `https://your-endpoint.ondigitalocean.app/health`


