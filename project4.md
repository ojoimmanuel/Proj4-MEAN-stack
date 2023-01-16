# MEAN Stack Implementation

MEAN Stack is a combination of following components:  

1. MongoDB (Document database) – Stores and allows to retrieve data.  

2.  Express (Back-end application framework) – Makes requests to Database for Reads and Writes.  

3.  Angular (Front-end application framework) – Handles Client and Server Requests  

4.  Node.js (JavaScript runtime environment) – Accepts requests and displays results to end user  

## STEP 0  

- Create an EC2 instance  

![EC2 running](./images/Screenshot%20from%202023-01-16%2008-18-48.png) 

## STEP 1 - Install NodeJs  

-   update ubuntu  
```
sudo apt update
```  
![update](./images/Screenshot%20from%202023-01-16%2008-21-21.png)

-   upgrade ubuntu  
```
sudo apt upgrade
```  
![upgrade](./images/Screenshot%20from%202023-01-16%2008-22-02.png)
-   add certificates  
```
sudo apt -y install curl dirmngr apt-transport-https lsb-release ca-certificates

curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
```
![cert](./images/Screenshot%20from%202023-01-16%2008-23-17.png)  

-   Install NodeJS  
```
sudo apt install -y nodejs
```
![nodejs](./images/Screenshot%20from%202023-01-16%2008-24-44.png)  


## STEP 2: Install MongoDB  

-   MongoDB stores data in flexible, JSON-like documents. Fields in a database can vary from document to document and data structure can be changed over time. For our example application, we are adding book records to MongoDB that contain book name, isbn number, author, and number of pages.
mages/WebConsole.gif  
```
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 0C49F3730359A14518585931BC711F9BA15703C6
```
![mongo1](./images/Screenshot%20from%202023-01-16%2008-28-09.png)
```
echo "deb [ arch=amd64 ] https://repo.mongodb.org/apt/ubuntu trusty/mongodb-org/3.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.4.list
```
![mongo2](./images/Screenshot%20from%202023-01-16%2008-29-56.png)


-   Install MongoDB  
```
sudo apt install -y mongodb
```

![mondodb](./images/Screenshot%20from%202023-01-16%2008-31-23.png)  


-   Start the server
```
sudo service mongodb start
```
![start server](./images/Screenshot%20from%202023-01-16%2008-36-00.png)

-   verify service running  
```
sudo systemctl status mongodb
```

![service running](./images/Screenshot%20from%202023-01-16%2008-36-36.png)

-   Install npm  
```
sudo apt install -y npm
```
![install npm](./images/Screenshot%20from%202023-01-16%2008-37-42.png)

-   Install body-parser package  
```
sudo npm install body-parser
```
![body-parser](./images/Screenshot%20from%202023-01-16%2008-39-17.png)  

- create a folder named 'Books'  
```
mkdir Books && cd Books
```
![books](./images/Screenshot%20from%202023-01-16%2008-39-49.png)  

- In the Books dir, initialize npm project  
```
npm init
```
![npm init](./images/Screenshot%20from%202023-01-16%2008-40-12.png)

- Add a file named server.js  
```
vi server,js
```
![add file](./images/Screenshot%20from%202023-01-16%2008-42-17.png)


-   copy and paste the web server code below into the server.js  
```
var express = require('express');
var bodyParser = require('body-parser');
var app = express();
app.use(express.static(__dirname + '/public'));
app.use(bodyParser.json());
require('./apps/routes')(app);
app.set('port', 3300);
app.listen(app.get('port'), function() {
    console.log('Server up: http://localhost:' + app.get('port'));
});
```
![code](./images/Screenshot%20from%202023-01-16%2008-41-42.png)  

## STEP 3-Install Express and set up routes to the server

Express is a minimal and flexible Node.js web application framework that provides features for web and mobile applications. We will use Express in to pass book information to and from our MongoDB database.

We also will use Mongoose package which provides a straight-forward, schema-based solution to model your application data. We will use Mongoose to establish a schema for the database to store data of our book register.

```
sudo npm install express mongoose
```
![express](./images/Screenshot%20from%202023-01-16%2008-43-40.png)

- In 'Books' folder, ceate a folder named apps  
```
mkdir apps && cd apps
```
![apps folder](./images/Screenshot%20from%202023-01-16%2008-44-12.png)

- Create a routes.js file  
```
vi routes.js
```
![routes](./images/Screenshot%20from%202023-01-16%2008-44-40.png)

-   Copy and paste the below code into routes.js  
```
var Book = require('./models/book');
module.exports = function(app) {
  app.get('/book', function(req, res) {
    Book.find({}, function(err, result) {
      if ( err ) throw err;
      res.json(result);
    });
  }); 
  app.post('/book', function(req, res) {
    var book = new Book( {
      name:req.body.name,
      isbn:req.body.isbn,
      author:req.body.author,
      pages:req.body.pages
    });
    book.save(function(err, result) {
      if ( err ) throw err;
      res.json( {
        message:"Successfully added book",
        book:result
      });
    });
  });
  app.delete("/book/:isbn", function(req, res) {
    Book.findOneAndRemove(req.query, function(err, result) {
      if ( err ) throw err;
      res.json( {
        message: "Successfully deleted the book",
        book: result
      });
    });
  });
  var path = require('path');
  app.get('*', function(req, res) {
    res.sendfile(path.join(__dirname + '/public', 'index.html'));
  });
};
```  
![code](./images/Screenshot%20from%202023-01-16%2008-45-30.png)

- Create models folder  
```
mkdir models && cd models
```
![models](./images/Screenshot%20from%202023-01-16%2008-46-23.png)

-   create a book.js file  
```
vi book.js
```
![book](./images/Screenshot%20from%202023-01-16%2008-46-45.png)

-   Copy and paste below code into 'book.js'  
```
var mongoose = require('mongoose');
var dbHost = 'mongodb://localhost:27017/test';
mongoose.connect(dbHost);
mongoose.connection;
mongoose.set('debug', true);
var bookSchema = mongoose.Schema( {
  name: String,
  isbn: {type: String, index: true},
  author: String,
  pages: Number
});
var Book = mongoose.model('Book', bookSchema);
module.exports = mongoose.model('Book', bookSchema);
```
![code](./images/Screenshot%20from%202023-01-16%2008-47-19.png)

## STEP 4-Access the routes with AngularJS
AngularJS provides a web framework for creating dynamic views in your web applications. In this tutorial, we use AngularJS to connect our web page with Express and perform actions on our book register.

-   Change the directory back to ‘Books’  
```
cd ../..
```
![dir](./images/Screenshot%20from%202023-01-16%2008-48-26.png)
-   create a folder named public  
```
mkdir public && cd public
```
![public](./images/Screenshot%20from%202023-01-16%2008-49-01.png)

-   Add a file named script.js
```
vi script.js
```
![file](./images/Screenshot%20from%202023-01-16%2008-49-24.png)

-   copy and paste below code (controller config defined) into the script.js file
```
var app = angular.module('myApp', []);
app.controller('myCtrl', function($scope, $http) {
  $http( {
    method: 'GET',
    url: '/book'
  }).then(function successCallback(response) {
    $scope.books = response.data;
  }, function errorCallback(response) {
    console.log('Error: ' + response);
  });
  $scope.del_book = function(book) {
    $http( {
      method: 'DELETE',
      url: '/book/:isbn',
      params: {'isbn': book.isbn}
    }).then(function successCallback(response) {
      console.log(response);
    }, function errorCallback(response) {
      console.log('Error: ' + response);
    });
  };
  $scope.add_book = function() {
    var body = '{ "name": "' + $scope.Name + 
    '", "isbn": "' + $scope.Isbn +
    '", "author": "' + $scope.Author + 
    '", "pages": "' + $scope.Pages + '" }';
    $http({
      method: 'POST',
      url: '/book',
      data: body
    }).then(function successCallback(response) {
      console.log(response);
    }, function errorCallback(response) {
      console.log('Error: ' + response);
    });
  };
});
```
![code](./images/Screenshot%20from%202023-01-16%2008-49-59.png)


-   In public folder, create a file named index.html;
```
vi index.html
```
![create file](./images/Screenshot%20from%202023-01-16%2008-50-42.png)


-   Copy and paste the code below into index.html file.
```
<!doctype html>
<html ng-app="myApp" ng-controller="myCtrl">
  <head>
    <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.6.4/angular.min.js"></script>
    <script src="script.js"></script>
  </head>
  <body>
    <div>
      <table>
        <tr>
          <td>Name:</td>
          <td><input type="text" ng-model="Name"></td>
        </tr>
        <tr>
          <td>Isbn:</td>
          <td><input type="text" ng-model="Isbn"></td>
        </tr>
        <tr>
          <td>Author:</td>
          <td><input type="text" ng-model="Author"></td>
        </tr>
        <tr>
          <td>Pages:</td>
          <td><input type="number" ng-model="Pages"></td>
        </tr>
      </table>
      <button ng-click="add_book()">Add</button>
    </div>
    <hr>
    <div>
      <table>
        <tr>
          <th>Name</th>
          <th>Isbn</th>
          <th>Author</th>
          <th>Pages</th>

        </tr>
        <tr ng-repeat="book in books">
          <td>{{book.name}}</td>
          <td>{{book.isbn}}</td>
          <td>{{book.author}}</td>
          <td>{{book.pages}}</td>

          <td><input type="button" value="Delete" data-ng-click="del_book(book)"></td>
        </tr>
      </table>
    </div>
  </body>
</html>
```

![code](./images/Screenshot%20from%202023-01-16%2008-51-27.png)

-   change dir to Books
```
cd ..
```
![dir](./images/Screenshot%20from%202023-01-16%2008-52-39.png)

-   Start the server
```
node server.js
```
![server](./images/Screenshot%20from%202023-01-16%2008-53-23.png)

-   The server is now up and running, we can connect it via port 3300. You can launch a separate Putty or SSH console to test what curl command returns locally.  
```
curl -s http://localhost:3300
```
-   open TCP port 3300 in AWS Web Console. Access the Book Register web app from the internet with a browser using Public IP address or Public DNS name  

![result](./images/Screenshot%20from%202023-01-16%2008-56-14.png)

## DONE!