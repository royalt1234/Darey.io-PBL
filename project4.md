## MEAN STACK IMPLEMENTATION

### Mean stack deployment to Ubuntu in AWS

1. Install NodeJs:
- Update and upgrade Ubuntu
```
sudo apt update
sudo apt upgrade
```
- On this project, i used the CLI straight from Web Console in AWS as i did not want to install or launch anything outside of AWS.

![](./images/p4/ScreenShot_4_14_2022_3_48_14_AMe.png)

- Add certificates
```
sudo apt -y install curl dirmngr apt-transport-https lsb-release ca-certificates

curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
```

![](./images/p4/ScreenShot_4_14_2022_3_51_46_AM.png)

- Install NodeJs `sudo apt install -y nodejs`

2. Install MongoDB:
- Run the following command to install MongoDB:
```
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 0C49F3730359A14518585931BC711F9BA15703C6

echo "deb [ arch=amd64 ] https://repo.mongodb.org/apt/ubuntu trusty/mongodb-org/3.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.4.list

sudo apt install -y mongodb
```

![](./images/p4/ScreenShot_4_14_2022_3_51_27_AM.png)

- Start the mongodb server and verify that it is running:
```
sudo service mongodb start

sudo systemctl status mongodb
```

![](./images/p4/ScreenShot_4_14_2022_3_53_26_AM.png)

- Install npm – Node package manager and body-parser package:
```
sudo apt install -y npm

sudo npm install body-parser
```

![](./images/p4/ScreenShot_4_14_2022_3_55_19_AM.png)


- Create a Books folder, cd into it `mkdir Books && cd Books` and run `npm init`

![](./images/p4/ScreenShot_4_14_2022_3_56_28_AM.png)


- Create a server.js file to run the server. Add the following code into the file:
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

![](./images/p4/ScreenShot_4_14_2022_3_57_16_AM.png)


3. Install Express and set up routes to the server
- Run the following command to install Express and mongoose:
`sudo npm install express mongoose`

![](./images/p4/ScreenShot_4_14_2022_3_58_44_AM.png)


- Make an apps directory and create a routes.js file in the folder to route requests. Add the following code into the file:
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

- Create a models directory and create a book.js file in the folder. Add the following code into the file:
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

4. Access the routes with AngularJS

- Cd to the root directory of the project and create a "public" directory. Create a "script.js" file in the "public" directory to load the AngularJS application. Add the following code into the file:
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

- Create an index.html file in the "public" directory and add the following code into the file:
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
- cd back to books folder and start the server with the following command:
`node server.js`

- Add port 3300 in the EC2 instance inbound rules.

![](./images/p4/ScreenShot_4_14_2022_4_04_24_AM.png)


- visit http://ip-address:3300/ in your browser.