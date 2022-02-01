+++
title = "Node.js Connector to a MySQL Database"
date = 2018-07-26T18:33:19-04:00
keywords = [ "jpiedra", "mysql", "nodejs", "javascript" ]
tags = [ "mysql", "nodejs", "javascript" ]
draft = false
+++

A simple module that connects to MySQL databases. No need to hardcode your credentials. 

<!--more-->
Writing reusable code is a cornerstone of development: if you can solve a problem once, and solve it well, you don't have to solve it again.

Today we'll demonstrate this concept using a module I wrote to develop a NodeJS API backend. This module reads a configuration file, which has two configuration objects defined. You use an environment variable to indicate whether your app is running in production or development; consequently, the module will use the config object corresponding to the environment defined by said variable. 

<h2>The Code</h2>
First, the module: <i>database.js</i>:

<pre>
var mysql = require('mysql');

var config = require('../config');

var pool = mysql.createPool({
    connectionLimit: 1000,
    timeout: 40000,
    host: config.settings[config.env].db.host,
    user: config.settings[config.env].db.user,
    password: config.settings[config.env].db.password,
    database: config.settings[config.env].db.database
});

module.exports = pool;
</pre>

As stated, the connection is based on a config file we can keep elsewhere. Technically speaking, this is also a module! In this example, we use a file that lives in the root of our project, aptly named <i>config.js</i>:

<pre>
var config = {};

config.env = process.env.NODE_ENV === 'development' ? 'development' : 'production';
config.settings = [];

config.settings['development'] = {
	db: {
		host: 'DEV-DB-HOST',
		user: 'DEV-DB-USER',
		password: 'DEV-DB-PASSWORD',
		database: 'DEV-DB',
		columns: [
			'name', 'email', 'another-column'
		]
	}	
};

config.settings['production'] = {
	db: {
		host: 'PROD-DB-HOST',
		user: 'PROD-DB-USER',
		password: 'PROD-DB-PASSWORD',
		database: 'PROD-DB',
		columns: [
			'name', 'email', 'another-column'
		]
	}	
};

module.exports = config;
</pre>

While there's nothing stopping us from hardcoding the connection data in <i>database.js</i>, the advantages in defining this data in a separate file are:

<ul>
    <li><b>Added Security</b>: If you ever hope to store your code in a VCS service such as GitHub or GitLab (and can't keep the repository private), you need a way of keeping confidential information like database usernames and passwords out of version control. Now that the data lives in <i>config.js</i>, we can add that file to a <i>.gitignore</i> file and rest easy, ensuring Git never pushes that data upstream.</li>
    <li><b>Flexibility</b>: While it would require some minor refactoring of the configuration file, there's nothing preventing us from defining additional environments beyond dev and prod. We could add a config object for <i>staging</i> or any other environment, and adjust <i>config.js</i> to use it based on environment variables.</li>
    <li><b>Easier Deployment Process</b>: A Dockerfile that is set up for deploying this project can switch environments easily, simply by "toggling" the environment through the NODE_ENV variable. Compare that with having to go into <i>database.js</i> manually, changing the credentials yourself.</li>
</ul>

Let's go over setting up an example app to use this code. 

<h2>Generating an Express App</h2>

After having installed <a href="https://nodejs.org/en/download/">NodeJS</a> on your development environment, we can take the following steps to set up a quick example project:

<pre>
mkdir mysql-node-ex
cd mysql-node-ex
npm install express-generator -g # OR...
sudo npm install express-generator -g
express .
npm install mysql --save
</pre>

This installs the bare minimum of dependencies we would need to use our module. The package <i>express-generator</i> lets us scaffold an Express app very quickly, giving us an example route file we can modify to our liking. We run the command <i>express .</i> to scaffold that app, and then install the <i>node-mysql</i> dependency last as that's not included by default in the app we generated. The additional flag at the end saves that dependency to our <i>package.json</i> file.

At this point we should have the following directory layout:
<pre>
.
├── app.js
├── bin
│   └── www
├── node_modules
├── package-lock.json
├── package.json
├── public
│   ├── images
│   ├── javascripts
│   └── stylesheets
│       └── style.css
├── routes
│   ├── index.js
│   └── users.js
└── views
    ├── error.jade
    ├── index.jade
    └── layout.jade
</pre>

We still need to add the two aforementioned files. Let's do that now, adding <i>database.js</i> to a folder called <i>connectors</i> and <i>config.js</i> in the application root as previously decided. Add the above contents for each file:

<pre>
mkdir connectors
touch connectors/database.js
touch config.js

# should leave us with the following folder structure:

.
├── app.js
├── bin
│   └── www
<b>├── config.js
├── connectors
│   └── database.js</b>
├── node_modules
├── package-lock.json
├── package.json
├── public
│   ├── images
│   ├── javascripts
│   └── stylesheets
│       └── style.css
├── routes
│   ├── index.js
│   └── users.js
└── views
    ├── error.jade
    ├── index.jade
    └── layout.jade
</pre>

<h2>Getting The Data Out</h2>

The next step is updating the configuration file to have credentials set for a local development database.

<pre>
config.settings['development'] = {
	db: {
		host: 'localhost',
		user: 'root',
		password: 'mytestpassword',
		database: 'sql_ex',
		columns: [
			'code', 'name', 'budget'
		]
	}	
};
</pre>

Afterwards, we modify our <i>routes/users.js</i> to run a query and return the results as a response. Because the Node MySQL library handles queries asynchronously (as do most libraries in NodeJS), we pass a callback to the query() method, and the callback handles what to do with the results from the query once it's ready:

<pre>
var express = require('express');
var config = require('../config');
var pool = require('../connectors/database');
var router = express.Router();

<b>var dataCallback = function(err, response, results, next) {
  if (err) {
    next(err);
  } else {
    response.set('Content-Type', 'application/json');
    response.send(results);
  };
};</b>

/* GET users listing. */
router.get('/', function(req, res, next) {
  res.send('respond with a resource');
});

<b>router.get('/data', function(req, res, next) {
  // get data
  pool.getConnection(function(err, conn) {
    if (err) return dataCallback(err, res, rows, next);
    conn.query("SELECT * FROM departments", function(err, rows) {
      conn.release();
      dataCallback(err, res, rows, next);
    });
  });
});</b>

module.exports = router;
</pre>

We can test our app, and visit a the URL defined at <i>localhost:3000/users/data</i> to retrieve data from a local development database, in the page response:

<pre>
$ export NODE_ENV=development
$ npm start &
$ curl localhost:3000/users/data
[{"Code":14,"Name":"IT","Budget":65000},{"Code":37,"Name":"Accounting","Budget":15000},{"Code":59,"Name":"Human Resources","Budget":240000},{"Code":77,"Name":"Research","Budget":55000}]GET /users/data 200 14.996 ms - 185
</pre>