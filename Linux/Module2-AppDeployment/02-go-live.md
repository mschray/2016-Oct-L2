# Make your app work

Now that we have installed the MEAN stack lets take it for a spin. We will get a fully functional app running in the VM.

1. Configure git
1. Deploy the app
1. MigrateDB

## Download and install sample app

We will be using the Northwind sample app created by [Bradley Braithwaite](https://github.com/bbraithwaite).

1. First of, lets fork his [Northwind repository](https://github.com/bbraithwaite/NorthwindNode).

    ![alt text][fork]

1. From your SSH session in the server lets create a new ssh key, just like we did in section 1 but this time make sure to keep the default location and and an empty passphrase. 

    ```bash
     ssh-keygen -t rsa -b 4096
     cat ~/.ssh/id_rsa.pub
     ```
1. Name your server

    ```bash
    git config --global user.name "Server name"
    git config --global user.email "my@server.rocks"
    ```

1. Add your server's key to the repo. Go to `'Settings' > 'Deploy Keys' > 'Add deploy key'`, and add the key from your server.

    ![alt text][deploy-key]

1. Clone your repo to your VM:

    ```bash
    cd ~
    git clone git@github.com:yourGithubUsername/NorthwindNode.git northwind
    ```

1. Lets create a git webhook in our server

    ```bash
    cd northwind
    mkdir webhook
    cd webhook
    npm init
    sudo npm install -g express
    ```

     Create `'index.js'` file an add the following lines to it:

     ```js
     var express = require('express'),
    http = require('http'),
    app = express();

    app.set('port', process.env.PORT || 3030);

    app.post('/push/', function (req, res) {
        var spawn = require('child_process').spawn,
            deploy = spawn('sh', [ './pull.sh' ]);

        deploy.stdout.on('data', function (data) {
            console.log(''+data);
        });

        deploy.on('close', function (code) {
            console.log('Child process exit code: ' + code);
        });
        res.json(200, {message: 'Received'})
    });

    http.createServer(app).listen(app.get('port'), function(){
    console.log('Listening on: ' + app.get('port'));
    });
    ```

    And create the file `'pull.sh'`:

    ```bash
    #!/bin/sh
    #First we remove all the changes in the local master, then we pull the changes
    cd ~/northwind
    git reset --hard origin/master
    git clean -f
    git pull
    ```

    Replace the contents of the `'package.json'` file with the following lines:

    ```json
    {
        "name": "webhook",
        "version": "1.0.0",
        "description": "",
        "main": "index.js",
        "scripts": {
            "test": "echo \"Error: no test specified\" && exit 1",
            "start": "node index.js"
        },
        "author": "",
        "license": "ISC",
        "dependencies": {
            "express": "^4.14.0"
        }
    }
    ```

    Start the service:

    ```bash
    npm update
    npm start
    ------------------------
    > webhook@1.0.0 start /home/bruno/webhook
    > node index.js

    Listening on: 3030
    ```

1. In github we go to `'Settings' > 'Webhook' > 'Add webhook'`. And we provide the url of our webhook which is `'http://<vm-url>:3030/push/'`.

    ![alt text][webhook]

    We will immediately see that our webhook will respond to that:

    ```Shell
    HEAD is now at 718026e move filter expression to controller

    Already up-to-date.

    Child process exit code: 0
     ```
1. Open a new SSH session to your VM and go to the `'~/northwind'` folder and correct the dependecies in the package json file. Change `'"grunt-node-inspector": "~0.1.3",'` to `'"grunt-node-inspector": ">=0.1.3",'`

1. Install the dependencies (one by one)

    ```bash
    sudo npm install -g node-inspector
    sudo npm install -g node-gyp
    sudo npm install -g mocha
    sudo npm install -g migrate
    npm install grunt --save-dev
    npm install mocha --save-dev
    ```

    After you are done with the dependencies run:

    ```bash
    npm install
    bower install
    npm start
    ```

1. Test the app in the browser, then commit a file on the fly and profit!

## Create user account and populate database

1. In the Northwind website click on the `'Sign up'` button on the top right and create the user `'admin'` with the password `'password'`.

1. Back in your SSH session inside the `'northwind'` folder use the npm package `'migrate'` to populate the database and review the new elements in the website.

    ```bash
    cd ~/northwind
    migrate up
    ```

1. Open MongoDB cli

    ```bash
    mongo
    help
    show dbs
    use northwindnode-dev
    show collections
    db.categories.find()
    db.categories.findOne()
    db.categories.find({"name":{$regex : ".*s"}})
    ```

1. Update one category.

    ```bash
    db.categories.findAndModify({
    query: { name: "Beverages" },
    update: { $set: { "description" : "not beer" } },
    })
    ```

1. Verify the result by refreshing the list of categories in the webpage.

## Following step

1. [Things to consider](../Module3-ThingsToConsider/readme.md)

[fork]: img/fork.jpg "Fork it!"
[deploy-key]: img/deploy-key.jpg "Add the whole key"
[webhook]: img/webhook.jpg "3030 is the port"
