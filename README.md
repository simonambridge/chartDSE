#chartDSE
A simple demonstration of charting data from DSE/Cassandra tables generated by a Spark streaming data source. Visualised using a d3 dashboard using a ReST json API, collecting data from DSE using the DataStax Nodejs driver.

##Source Data
This demo queries two Cassandra tables and displays the data visually:
- a sysem table (compaction history)
- a custom table containing time series data generated by this Spark Streaming example here https://github.com/simonambridge/SparkSensorData

You can change the simplechart.html file to query and display data from a different table if you wish. If you do this you will also need to edit app.js to change the ReST API query for your table.

#Pre-Requisites
##Install node.js
<pre>
$ sudo curl -sL https://deb.nodesource.com/setup_7.x | sudo bash -
$ sudo apt-get install nodejs
$ npm install pug
$ npm install cassandra-driver
</pre>

##Create test project to test Node server
<pre>
$ mkdir test
$ cd test
$ npm init
$ npm install express --save
$ npm install connect serve-static
$ sudo npm install express-generator -g
</pre>

The default server definition file is app.js - this displays a simple Hello message:
<pre>
$ vi app.js

var express = require('express');
var app = express();

app.get('/', function (req, res) {
  res.send('Hello World!');
});

app.listen(3000, function () {
  console.log('Example app listening on port 3000!');
});
</pre>

Start the simple node server app:
<pre>
$ node app.js
</pre>
Example app listening on port 3000!

In your browser go to http://127.0.0.1:3000/

The browser displays:
<pre>
HelloWorld!
</pre>

Kill the server (press ^C).

##Simple Static Content Server Example
If you simply want to serve static content (e.g. HTML files) you could try this:
<pre>
$ node server - runs server.js file
</pre>
This provides a simple server on port 8000 e.g. http://localhost:8000/chartdse/public/index.html

The server.js file is very simple:
<pre>
var connect = require('connect');
var serveStatic = require('serve-static');
connect().use(serveStatic(__dirname)).listen(8000);
</pre>

if no start script is in package .json it will default to this server.js

See also http://expressjs.com/en/starter/basic-routing.html

#Build The chartDSE Application
Now let's create a proper node application.

From a parent directory create a new project and directory:
<pre>
$ express chartdse

   create : chartdse
   create : chartdse/package.json
   create : chartdse/app.js
   create : chartdse/public
   create : chartdse/routes
   create : chartdse/routes/index.js
   create : chartdse/routes/users.js
   create : chartdse/views
   create : chartdse/views/index.jade
   create : chartdse/views/layout.jade
   create : chartdse/views/error.jade
   create : chartdse/bin
   create : chartdse/bin/www
   create : chartdse/public/javascripts
   create : chartdse/public/images
   create : chartdse/public/stylesheets
   create : chartdse/public/stylesheets/style.css

   install dependencies:
     $ cd chartdse && npm install

   run the app:
     $ DEBUG=chartdse:* npm start
</pre>

Go into the directory that was created.
<pre>
$ cd ../chartdse
</pre>

Install local package dependencies:
<pre>
$ npm install
</pre>

Run this command to start the node server:
<pre>
DEBUG=chartdse:* npm start           
</pre>
Server process output:
<pre>
> chartdse@0.0.0 start /root/chartdse
> node ./bin/www
  chartdse:server Listening on port 3000 +0ms

GET / 200 364.402 ms - 170
GET /stylesheets/style.css 200 5.804 ms - 111
</pre>

Go to http://127.0.0.1:3000/

The browser displays:

<pre>
Express

Welcome to Express
</pre>
Our Express app is now up and running!

Now from the parent directory clone the repo contents into the project directory:
<pre>
$ git clone http://gitchub.com/simonambridge/chartDSE
</pre>
This will overlay the repo files into the directory structure created by Express.

##Important Files
- app.js - defines the environment - server, port, paths etc. maps the /public directory for static content, defines the paths to the ReST interfaces and the Cassandra query code.
-- package.json contains the 'start script' element pointing to "./bin/www"
- ./bin/www" - starts the http server
- If an index.html file is in ./public then it will display that as the entry point e.g. http://localhost:3000 
- If not then it will use index.jade  in /views and display the Express welcome message
- ./public/chart.html - a simple d3 example chart and buttons to test the ReST interfaces for the two table queries
- ./public/simplechart.html - a simple dashboard that displays time-series data from the custom table - from ideas at https://github.com/ESeufert/d3.js-dashboard-examples

##Example Cassandra Nodejs code:
A simple example of using the Nodejs Cassandra connector:
<pre>
var cassandra = require('cassandra-driver');
var client = new cassandra.Client({ contactPoints: ['localhost']});

client.execute('select key from system.local', function(err, result) {
  if (err) throw err;
  console.log(result.rows[0]);
});
</pre>

##Data we will pull from Cassandra

For the first ReST API we will use the system.compaction_history table: 
<pre>
system.compaction_history (
    id uuid PRIMARY KEY,
    bytes_in bigint,
    bytes_out bigint,
    columnfamily_name text,
    compacted_at timestamp,
    keyspace_name text,
    rows_merged map<int, bigint>
</pre>
This is the query:
<pre>
select keyspace_name, columnfamily_name, compacted_at, bytes_in, bytes_out from system.compaction_history
</pre>

The second table looks like this:
<pre>
CREATE TABLE sparksensordata.sensordata (
    name text,
    time timestamp,
    value double,
    PRIMARY KEY ((name, time))
)
</pre>
And we use this query:
<pre>
select time, value from sparksensordata.sensordata
</pre>
<BR>

#Test The ReST Interfaces
The rest interfaces are served from routes defined in app.js e.g.
<pre>
app.get('/compaction', function(req, res) {
  var client = new cassandra.Client({ contactPoints: ['localhost'] , keyspace: 'system'});
  var queryString = 'select keyspace_name, columnfamily_name, compacted_at, bytes_in, bytes_out from system.compaction_history';
  client.execute(queryString, function(err, result) 
  {
    if (err) throw err;
    for (var item in result.rows) {
      console.log(result.rows[item]);
    }
    res.setHeader('Content-Type', 'application/json');
    jsonString=JSON.stringify(result.rows);
    console.log('JSON = ',jsonString);
    res.send(JSON.stringify(result.rows));
  });
});
</pre>

Now we know where everything is and what we're looking for, let's test it.
If the server isn't currently running, go to your project directory and run this command to start the node server:
<pre>
DEBUG=chartdse:* npm start           
</pre>

##Compaction History
Test the system.compaction_history interface - json data is returned:
<BR>
<BR>
![alt text] (https://raw.githubusercontent.com/simonambridge/chartDSE/master/compaction_history.png)

##Sensor Data
Test the custom sparksensordata.sensordata interface - json data is returned:
<BR>
<BR>
![alt text] (https://raw.githubusercontent.com/simonambridge/chartDSE/master/sensordata.png)
<BR>

#Test The Chart Pages
These are HTML pages that call the ReST interfaces and contain scripts to build d3 charts to display the data. These are in ./public

##Simple d3 Chart
This contains a simple demo to build a bar chart, plus links to the ReST interfaces for the two tables. There is also a button showing how to link to another static HTML page.
<BR>
<BR>
![alt text] (https://raw.githubusercontent.com/simonambridge/chartDSE/master/chart_html.png)

##Time Series Data
This takes the sensor data in the sparksensordata.sensordata table and displays it in a time series line graph.
<BR>
<BR>
![alt text] (https://raw.githubusercontent.com/simonambridge/chartDSE/master/simplechart_html.png)

#Installing d3js Locally
By default the d3 scripts are downloaded at runtime using a link at the top of the html page that will run your javascript (for example in simplechart.html) e.g.:

<pre>
<script src="http://d3js.org/d3.v2.js"></script>
</pre>

If you do not want to be dependent on an internet connection in order to run your code you can download d3 and use it locally.

Follow these steps:

Go to the d3 site at http://d3js.org 
Create a directory in your project directory (or centrally if you want to share between projects).

<pre>
$ cd <my project directory>
$ mkdir d3
</pre>

Download the d3js.zip zip file to the d3 dir created above.

In your chart HTML file
Replace <script src="http://d3js.org/d3.v2.js"></script>
with    <script src="./d3/d3.js"></script>


##jquery

You can also install jquery locally to remove the dependency on an internet connection.

$ npm install jqueryui
chartdse@0.0.0 /u02/dev/dse_dev/chartdse
└── jqueryui@1.11.1 

$ npm install jquery
chartdse@0.0.0 /u02/dev/dse_dev/chartdse
└── jquery@3.1.1 

In your chart HTML file:
Replace <script type="text/javascript" src="https://ajax.googleapis.com/ajax/libs/jquery/1.7.2/jquery.min.js"></script>
with  <script type="text/javascript" src="./node_modules/jquery/dist/jquery.min.js"></script>

Replace <script type="text/javascript" src="https://ajax.googleapis.com/ajax/libs/jqueryui/1.8.21/jquery-ui.min.js"></script>
with    <script type="text/javascript" src="./node_modules/jqueryui/jquery-ui.min.js"></script>


##Copy files to static directory

$ cp node_modules/jquery/dist/jquery.min.js public
$ cp node_modules/jqueryui/jquery-ui.min.js public
$ cp d3/d3.js public

Your HTML file header should now look like this:

//<script src="http://d3js.org/d3.v2.js"></script>
//<script type="text/javascript" src="https://ajax.googleapis.com/ajax/libs/jquery/1.7.2/jquery.min.js"></script>
//<script type="text/javascript" src="https://ajax.googleapis.com/ajax/libs/jqueryui/1.8.21/jquery-ui.min.js"></script>
<script src="d3.js"></script>
<script type="text/javascript" src="jquery.min.js"></script>
<script type="text/javascript" src="jquery-ui.min.js"></script>






