# Pern App Cloud Deployment

### Objectives

- Deploy server to heroku
- Deploy react app to Heroku

## **Disclaimer**

This repo is set up assuming that your apps are stuctured the same way we've done them in class.

## Preparing Your Database Connection

Open the `config/config.json` file for sequelize. Rename the file to `config.js`. And refactor it to a module exports like below:

```js
require('dotenv').config()
module.exports = {
  development: {
    database: '<Your Dev Database>',
    dialect: 'postgres'
  },
  test: {
    database: '<Your Database Test Name>',
    dialect: 'postgres'
  },
  production: {
    use_env_variable: 'DATABASE_URL',
    dialect: 'postgres',
    dialectOptions: {
      ssl: {
        rejectUnauthorized: false,
        require: true
      }
    }
  }
}
```

In `models/index.js` we need to fix the require for the config, change the config require statement to the one below:

```js
const config = require(__dirname + '/../config/config.js')[env]
```

## Creating A Heroku Project

In your project directory, run:

```sh
heroku create
```

Once your app is created, we need to add postgres as an addon:

```sh
heroku addons:create heroku-postgresql:hobby-dev
```

Next we'll add any `environment variables` for your project:

```sh
heroku config:set VARIABLE_NAME=<variable>
```

### Wiring Up Our Code

We now need to prep our code for deployment.

In your `server.js`:

- > Require path from `path`:
  >
  > ```js
  > const path = require('path')
  > ```

- > Add the following **BELOW** your current middleware,and routes, **but before the listen method**:
  >
  > ```js
  > if (process.env.NODE_ENV === 'production') {
  >   app.use(express.static(path.join(__dirname, 'client/build')))
  >   app.get('*', (req, res) => {
  >     res.sendFile(path.join(`${__dirname}/client/build/index.html`))
  >   })
  > }
  > ```

Finally in your `package.json` for your **SERVER**, add a new script in your `scripts` section:

```json
    "build": "npm install && npx sequelize-cli db:migrate && cd client && rm -rf build && npm install && npm run build"
```

Make sure you have a `start` and `dev` script:

```json
{
  "dev": "nodemon <entry>.js",
  "start": "node <entry>.js"
}
```

## Pointing Client To Our Api

In `client/src/globals/index.js`, modify your base url`:

```js
process.env.NODE_ENV === 'production'
  ? `${window.location.origin}/api`
  : '<your local backend server>/api'
```

## Deploying Our Project

In your `Heroku` account select the deploy tab and select `connect to github`. If prompted to sign in go ahead and do so. You should now have a field to search repos:

![Heroku-Github](images/heroku-github.png)

Search For your project repo and click on `connect`.

You can now set up automatic deploys:

![Deploy](images/deploy.png)

Select `Enable Automatic Deploy` and make sure it's pointing to your `main` branch.

Now in your project folder, `add`, `commit` and `push` your changes and a build should kick off on heroku!

You can monitor progress in heroku's activity tab!

Once the build is finished you can open your app by using the `Open App` button or `heroku open` in your terminal.

You can also publish a build by adding and committing your changes and running `git push heroku main`

## Monitoring Your Server

You can run `heroku logs --tail` to monitor what's happening with your server.

## Seeding Files

If you have seeds that you want to run, you can ssh into your heroku server with `heroku run bash` and then running `npx sequelize-cli db:seed:all`.
