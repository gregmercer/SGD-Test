# SGD-Test
Example of a FB Messenger Bot using Nodejs

## Steps for creating the sgd-test bot

### Create a directory for your new bot

```
mkdir sgd
cd sgd
mkdir node_modules
```

### Create a package.json file

```
touch package.json
create package.json file
{
  "name": "sgd-test",
  "version": "0.0.1",
  "engines": {
    "node": "0.10.x",
    "npm": "3.8.x"
  },
  "private": true,
  "scripts": {
    "start": "node app.js"
  },
  "dependencies": {
  }
}
```

### Use the Node Package Manager to add the required packages

```
sudo npm install --save express
sudo npm install --save request
sudo npm install --save body-parser
sudo npm update --save
```

### Create your bot app on Heroku

```
heroku create --app sgd-test
  (should return this -> https://sgd-test.herokuapp.com/ | https://git.heroku.com/sgd-test.git)
```

### Push these files up to Heroku

```
git init
git remote add heroku git@heroku.com:sgd-test.git
git add .
git commit -m 'test'
git push heroku master
```

### Create the app.js file

```
touch app.js
```

### Edit the app.js file to have the following:

```
var express = require("express");
var bodyParser = require("body-parser");
var request = require("request");
var app = express();

app.use(bodyParser.urlencoded({ extended: false }));
app.use(bodyParser.json());

var port = process.env.PORT || 3000;

// Configuration
var pub = __dirname + '/public';

// Routing paths

app.get('/', function(req, res){
  res.send('Hello World from sgd-test Server');
});

// Only listen on $ node app.js

if (!module.parent) {
  app.listen(port);
  console.log("Express server listening on port %d", port);
}
```

### Push these changes up to Heroku

```
git add .
git commit -m 'test'
git push heroku master
```

### Test your new app

```
open the following in your browser
https://sgd-test.herokuapp.com/
```

### Create a new FB page

```
go to FB and create a new page
https://www.facebook.com/pages/create
```

### Create a new FB Messenger Bot App 

```
go to FB and add your new messenger app
https://developers.facebook.com/apps
pick www (website)

click on 'Messenger'
then click on 'Get Started'
select the Page you created in the previous step
copy the generated Page Access Token 
```

### Add config variables to Heroku

```
go to heroku
https://dashboard.heroku.com/apps

find your new app and click on the icon next to the app's name
click on the 'Settings' link
next click on the 'Reveal Config Vars'
add a key and value for your Page Access Token (FB_PAGE_ACCESS_TOKEN)
click 'Add'

also add a FB Verify Token (FB_VERIFY_TOKEN) 
this can be any string you want.
```
### Add a new route for GET /webhook

```
add the following to app.js
app.get('/webhook/', function (req, res) {
  if (req.query['hub.verify_token'] === process.env.FB_VERIFY_TOKEN) {
    res.send(req.query['hub.challenge']);
  }
  res.send('Error, wrong validation token');
});

save app.js
```

### Push these changes up to Heroku

```
git add .
git commit -m 'test'
git push heroku master
```

### Setup Webhooks on FB

```
go back to FB app dashboard

open this page
https://developers.facebook.com/apps

click on your app
click on 'Messenger'

next click on 'Setup Webhooks'

copy your heroku app url
https://sgd-test.herokuapp.com/

copy your FB Verify Token

check off all the Subscription Fields

click 'Verify and Save'

you should get back a 'Completed' status
```

### Subscribe to your FB page

```
on the same FB app 'Messenger' page
in the 'Webhooks' section
Select your page to subscribe
click on 'Subscribe'
```

### Add a POST /webhook to your Bot

```
edit app.js
add Post handling for /webhook

app.post('/webhook/', function (req, res) {
  messaging_events = req.body.entry[0].messaging;
  for (i = 0; i < messaging_events.length; i++) {
    event = req.body.entry[0].messaging[i];
    console.log(event.sender);
    sender = event.sender.id;
    if (event.message && event.message.text) {
      text = event.message.text;
      console.log('and the text is: '+text);
      sendTextMessage(sender, "Text received, echo: "+ text.substring(0, 200));
    }
  }
  res.sendStatus(200);
});

var page_access_token = process.env.FB_PAGE_ACCESS_TOKEN;

function sendTextMessage(sender, text) {
  messageData = {
    text:text
  }
  request({
    url: 'https://graph.facebook.com/v2.6/me/messages',
    qs: {access_token:page_access_token},
    method: 'POST',
    json: {
      recipient: {id:sender},
      message: messageData,
    }
  }, function(error, response, body) {
    if (error) {
      console.log('Error sending message: ', error);
    } else if (response.body.error) {
      console.log('Error: ', response.body.error);
    }
  });
}
```

### Push these changes up to Heroku

```
git add .
git commit -m 'test'
git push heroku master
```

### Test your Post message handling

```
go to your FB page
click on the 'Message' button

send a test message

if all is working you should see your test message returned by your bot
Text received, echo: test 123
```

### Viewing your Heroku Bot logs

```
you can view Heroku logs with either of the following command-line commands:

heroku logs
or
heroku logs -t
```

### Use your FB m.me link

```
go to Messenger link for you app

m.me/<your app name>
m.me/sgd-test
```

### Edit app.js a final time

```
as a final step edit app.js and paste in the text from app-final.js

this covers using a Strutured Messages and Postbacks

the details for this step are covered on this page:

Getting Started
https://developers.facebook.com/docs/messenger-platform/quickstart
```

### Test your Structure Messages and Postbacks

```
go to your FB page
click on the 'Message' button

send a test message with the text 'Generic'

this should reply with a Structured Message

now click on the Postback button in the returned message

you should see a reply from the Postback
```

