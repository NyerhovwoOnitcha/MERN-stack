# Project 3: MERN STACK IMPLEMENTATION
## STEP 1: Backend Configuration
### Log into your server, proceed to upgrade and update your server
`sudo apt update`
`sudo apt upgrade`

### Get the location of of Node,js software
`curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -`

### Install Node.js on your server
`sudo apt-get install -y nodejs`

### The coomand above installs both node.js and npm which is a package manager for Node. Check that they were both installed.
`node -v `
`npm -v `
![node.js and npm installed](./images/installed.jpg)


### Create a new directory for your To-Do project and cd into the directory
`mkdir Todo`
`cd Todo`

### Use the npm init to initialize your project. A new file named package.json will be created, this file will contain information your application. Follow the prompts and accept deafualt values. 
`npm init`
![](./images/npm%20init.jpg)

### Run ls to confirm that the package.json file was installed.
`ls`
![package.json file installed](./images/npm%20init%20confirmed.jpg)

# Project 3: Step 2
## Installing Express JS

### Using the express js framework using npm
`npm install express`

### Create an index.js file
`touch index.js`

### Install the dotenv module
`npm install dotenv`

### Open the index.js file and insert the code below
`vim index.js`
```
const express = require('express');
require('dotenv').config();

const app = express();

const port = process.env.PORT || 5000;

app.use((req, res, next) => {
res.header("Access-Control-Allow-Origin", "\*");
res.header("Access-Control-Allow-Headers", "Origin, X-Requested-With, Content-Type, Accept");
next();
});

app.use((req, res, next) => {
res.send('Welcome to Express');
});

app.listen(port, () => {
console.log(`Server running on port ${port}`)
});
```

### Now start your server
`node index.js`
![run your server](./images/server%20running.jpg)

### First go to your instance and open port 5000 in it's security group.
![security grouo edited](./images/ec2.jpg)

### Open yourbrowser and try to access your server via port 5000. Use the follwoing commands to get ypur public ip-addess and DNS name.

```
Public IP address 
"curl -s http://169.254.169.254/latest/meta-data/public-ipv4"

Public DNS name
"curl -s http://169.254.169.254/latest/meta-data/public-hostname"
```
![server accessed from browser](./images/server%20running%20cofirmed.jpg)

### Your Todo application should be able to perfiorm 3 tasks:
```
1- Create a new task: POST METHOD
2- Display list of all tasks: GET METHOD
3- Delete a completed task: DELETE METHOD

Each task will be assocaited with a particular endpoint and will use a different HTTPD request method. Thus for each task you need to create routes. These routes will define various endpoints that your app will depend on.
```
### Create folder for routes and cd into the folder
`mkdir routes`
`cd routes`
### Create a file api.js, open the file and insert the code below.
`touch api.js`
```
const express = require ('express');
const router = express.Router();

router.get('/todos', (req, res, next) => {

});

router.post('/todos', (req, res, next) => {

});

router.delete('/todos/:id', (req, res, next) => {

})

module.exports = router;
```

# Project 3: STEP 3
## Create a model
### First install mongoose, it's a node.js package used that mkakes working with modgodn easier. It's useed to create a schema and a model
`npm install mongoose`
### Create a new folder models and cd into it and create a file todo.js inside.
`mkdir models && cd models && touch todo.js`

### Open the todo.js file and insert the code below
`vi todo.js`
```
const mongoose = require('mongoose');
const Schema = mongoose.Schema;

//create schema for todo
const TodoSchema = new Schema({
action: {
type: String,
required: [true, 'The todo text field is required']
}
})

//create model for todo
const Todo = mongoose.model('todo', TodoSchema);

module.exports = Todo;
```
### Now update the routes in the api.js file to use our model. cd to the route directory, open the api.js file, delete the codein it and replace it with the code below
```
const express = require ('express');
const router = express.Router();
const Todo = require('../models/todo');

router.get('/todos', (req, res, next) => {

//this will return all the data, exposing only the id and action field to the client
Todo.find({}, 'action')
.then(data => res.json(data))
.catch(next)
});

router.post('/todos', (req, res, next) => {
if(req.body.action){
Todo.create(req.body)
.then(data => res.json(data))
.catch(next)
}else {
res.json({
error: "The input field is empty"
})
}
});

router.delete('/todos/:id', (req, res, next) => {
Todo.findOneAndDelete({"_id": req.params.id})
.then(data => res.json(data))
.catch(next)
})

module.exports = router;
```
# Project 3: STEP 4
##  MongoDB database
### We need a database to store our data. We will be using mLab as mLab provides MongoDB as a service. Use the link to go create an account "https://www.mongodb.com/atlas-signup-from-mlab"

### Follow the process and sign in choosing the same options as below
![sign in](./images/atlas%201.jpg)

### Create a free shared account
![choose plan](./images/server%20running.jpg)
### choose cloud provider, select region and create cluster
![create cluster](./images/cloud%20provider.jpg)
### Click on database, on your cluster click on browse collections and add your data. 
![](./images/database.jpg)
![](./images/add%20data.jpg)
![create database](./images/create%20database.jpg)

### Now your database has been created. Click on database access and add a new database user
![database access](./images/database%20access.jpg)

### Choose authentication method as password, set a username and password and create a  user. Toggle on the temporary user duration and set itfor 1 week
![add user](./images/add%20user.jpg)

### Click on network access, add ip address and choose allow from anywhere.
![](./images/network%20access.jpg)
![](./images/allow%20anywhere.jpg)

### Get your connection string and write it down. Get it by clicking on database -> Connect -> connect your application -> choose your driver as node.js and copy the connection string
![connect](./images/connect.jpg)
![connect your app](./images/connect%20your%20app.jpg)
![your connection string](./images/string.jpg)

### On your connection string replace the <password> with your user password.

### Copy the new string, Go to your Todo direcory and create a file, name it .env and insert the string in the file.

`touch .env`
`vi .env`

### Go to your shell, cd into your Todo directory and replace the codes in your index.js file with the codes below
`vi index.js`
```
const express = require('express');
const bodyParser = require('body-parser');
const mongoose = require('mongoose');
const routes = require('./routes/api');
const path = require('path');
require('dotenv').config();

const app = express();

const port = process.env.PORT || 5000;

//connect to the database
mongoose.connect(process.env.DB, { useNewUrlParser: true, useUnifiedTopology: true })
.then(() => console.log(`Database connected successfully`))
.catch(err => console.log(err));

//since mongoose promise is depreciated, we overide it with node's promise
mongoose.Promise = global.Promise;

app.use((req, res, next) => {
res.header("Access-Control-Allow-Origin", "\*");
res.header("Access-Control-Allow-Headers", "Origin, X-Requested-With, Content-Type, Accept");
next();
});

app.use(bodyParser.json());

app.use('/api', routes);

app.use((err, req, res, next) => {
console.log(err);
next();
});

app.listen(port, () => {
console.log(`Server running on port ${port}`)
});
```
### start your server, you should see a message ‘Database connected successfully’.
`node index.js`

```
If you are unable to connect to your database and you get an error saying "MongooseError: The `uri` parameter to `openUri()` must be a string, got "undefined". Make sure the first parameter to `mongoose.connect()` or `mongoose.createConnection()` is a string"

You can solve this by editing your index.js file. 
Look for this line of code : 
```
`mongoose.connect(process.env.DB, { useNewUrlParser: true, useUnifiedTopology: true })`

```
Replace the "process.env.DB" with your connection string you pasted in your .env file.
```

`mongoose.connect('mongodb+srv://usermike:cnl123456@cluster0.0xksaxh.mongodb.net/?retryWrites=true&w=majority', { useNew`

### You should be able to connect to your database

### Your backend is now configured. Test your backend code i.e all the api endpoints using an API dev client to make sure it gives the right response. Use POSTMAN to achieve this.

### Open your Postman and create a post request. Before making your request set your header as "key: Content-Type" and "value:application/json" 
![](./images/content-type.jpg)

`http://<PublicIP-or-PublicDNS>:5000/api/todos`

![post success](./images/post%20success.jpg)

### Create a get request

![get success](./images/get%20success.jpg)

### Delete request
![Delete request](./images/delete%20success.jpg)

### You have now successfully created the backend and have confirmed it accepts requests from your API.


# Project 3: STEP 4
## Frontend Creation

### To start the frontend, use the create-react-app command. This will create a directory called "client" in your project directory where all your react code will be stored.
` npx create-react-app client`

### Install the following dependencies.
### Concurrently
`npm install concurrently --save-dev`

### Nodemon
`npm install nodemon --save-dev`

### In your project directory folder, open the package.json file, replace the line of code that starts with "scripts" with the code below
```
"scripts": {
"start": "node index.js",
"start-watch": "nodemon index.js",
"dev": "concurrently \"npm run start-watch\" \"cd client && npm start\""
},
```
### Change to the clinet directory, open the package .json file and add the following key value pair in the file

`cd client`

`vi package.json`

`"proxy": "http://localhost:5000"`

### The reason we are doing this is so we can access the application directly from our browser by calling the url "http://localhost:5000" rather than the whole part of the application "http://localhost:5000/api/todos"

### Go to the project directory and run the command below
`npm run dev`

![server started](./images/npm%20run%20devv.jpg)

### Open port 3000 in your instance security group. Go to your browser, open:

`<instance public ip address:3000 >`

![server started](./images/frontend.jpg)

### Create your react components. navigate to your client directory, proceed to move to the src folder and inside the src folder create a folder called components
`cd client`

`cd src`

`mkdir components`

### Move to the components folder and in create create 3 files  Input.js, ListTodo.js and Todo.js.

`cd components`

`touch Input.js ListTodo.js Todo.js`

### Open the input.js file and insert the code below
`vi Input.js`

```
import React, { Component } from 'react';
import axios from 'axios';

class Input extends Component {

state = {
action: ""
}

addTodo = () => {
const task = {action: this.state.action}

    if(task.action && task.action.length > 0){
      axios.post('/api/todos', task)
        .then(res => {
          if(res.data){
            this.props.getTodos();
            this.setState({action: ""})
          }
        })
        .catch(err => console.log(err))
    }else {
      console.log('input field required')
    }

}

handleChange = (e) => {
this.setState({
action: e.target.value
})
}

render() {
let { action } = this.state;
return (
<div>
<input type="text" onChange={this.handleChange} value={action} />
<button onClick={this.addTodo}>add todo</button>
</div>
)
}
}

export default Input
```
### navigate back to the client folder and install Axios

`npm install axios`

### Navigate nback to the components directory
`cd src/components`

### Open the ListTodo.js file and insert the code below

`cd src/components`

```
import React from 'react';

const ListTodo = ({ todos, deleteTodo }) => {

return (
<ul>
{
todos &&
todos.length > 0 ?
(
todos.map(todo => {
return (
<li key={todo._id} onClick={() => deleteTodo(todo._id)}>{todo.action}</li>
)
})
)
:
(
<li>No todo(s) left</li>
)
}
</ul>
)
}

export default ListTodo
```

### In your Todo.js file insert the following code
```
import React, {Component} from 'react';
import axios from 'axios';

import Input from './Input';
import ListTodo from './ListTodo';

class Todo extends Component {

state = {
todos: []
}

componentDidMount(){
this.getTodos();
}

getTodos = () => {
axios.get('/api/todos')
.then(res => {
if(res.data){
this.setState({
todos: res.data
})
}
})
.catch(err => console.log(err))
}

deleteTodo = (id) => {

    axios.delete(`/api/todos/${id}`)
      .then(res => {
        if(res.data){
          this.getTodos()
        }
      })
      .catch(err => console.log(err))

}

render() {
let { todos } = this.state;

    return(
      <div>
        <h1>My Todo(s)</h1>
        <Input getTodos={this.getTodos}/>
        <ListTodo todos={todos} deleteTodo={this.deleteTodo}/>
      </div>
    )

}
}

export default Todo;
```

### Navigate to the src folder, edit the app.js file and paste the code in it

`cd ..`
`vi app.js`
```
import React from 'react';

import Todo from './components/Todo';
import './App.css';

const App = () => {
return (
<div className="App">
<Todo />
</div>
);
}

export default App;
```
### In the same directory, edit the app.css file and insert the code below
`vi app.css`
```
.App {
text-align: center;
font-size: calc(10px + 2vmin);
width: 60%;
margin-left: auto;
margin-right: auto;
}

input {
height: 40px;
width: 50%;
border: none;
border-bottom: 2px #101113 solid;
background: none;
font-size: 1.5rem;
color: #787a80;
}

input:focus {
outline: none;
}

button {
width: 25%;
height: 45px;
border: none;
margin-left: 10px;
font-size: 25px;
background: #101113;
border-radius: 5px;
color: #787a80;
cursor: pointer;
}

button:focus {
outline: none;
}

ul {
list-style: none;
text-align: left;
padding: 15px;
background: #171a1f;
border-radius: 5px;
}

li {
padding: 15px;
font-size: 1.5rem;
margin-bottom: 15px;
background: #282c34;
border-radius: 5px;
overflow-wrap: break-word;
cursor: pointer;
}

@media only screen and (min-width: 300px) {
.App {
width: 80%;
}

input {
width: 100%
}

button {
width: 100%;
margin-top: 15px;
margin-left: 0;
}
}

@media only screen and (min-width: 640px) {
.App {
width: 60%;
}

input {
width: 50%;
}

button {
width: 30%;
margin-left: 10px;
margin-top: 0;
}
}
```

### Also edit the index.css file in the src directory

`vi index.css`
```
body {
margin: 0;
padding: 0;
font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", "Roboto", "Oxygen",
"Ubuntu", "Cantarell", "Fira Sans", "Droid Sans", "Helvetica Neue",
sans-serif;
-webkit-font-smoothing: antialiased;
-moz-osx-font-smoothing: grayscale;
box-sizing: border-box;
background-color: #282c34;
color: #787a80;
}

code {
font-family: source-code-pro, Menlo, Monaco, Consolas, "Courier New",
monospace;
}
```
### Go to your project directory i.e Todo and run:
`npm run dev`
![Todo app](./images/todo%20app.jpg)




