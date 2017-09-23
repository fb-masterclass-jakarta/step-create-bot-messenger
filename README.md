# How to build Facebook BOT Messenger

## What is needed ?

- Facebook Page
- Facebook App
- Webhook to Service
- heroku-cli (*optional*)


## **Step 1**

### we will build Webhook Service and deployment using Heroku as cloud service  

- make sure you have 
[Toolbet Heroku](https://devcenter.heroku.com/articles/heroku-cli) and account 
[Heroku](https://www.heroku.com)

```
npm install -g heroku-cli
heroku --version
heroku login
```
## **STEP 2**

### build webhook service with express

```
mkdir buildBot
cd buildBot
npm init
npm install express request body-parser --save

```

## **Step 3**

### create file **index.js**

```javascript
const express = require('express')
const bodyParser = require('body-parser')
const request = require('request')
const app = express()

app.set('port', (process.env.PORT || 5000))

app.use(bodyParser.urlencoded({extended: false}))

app.use(bodyParser.json())

app.get('/', function (req, res) {
	res.send('Masterclass Facebook Jakarta - Example Webhook')
})

app.get('/webhook/', function (req, res) {
	if (req.query['hub.verify_token'] === 'make_indonesian_great_again') {
		res.send(req.query['hub.challenge'])
	}
	res.send('Error, wrong token')
})

app.listen(app.get('port'), function() {
	console.log('running on port', app.get('port'))
})

```

## **Step 4**

### create file **Procfile**

```
web: node index.js
```

## **Step 5**

### lets deploy to heroku

```
git init
git add .
git commit --message "first upload"
heroku create
git push heroku master

```
*and try access heroku server*

## **Step 6**

Create Facebook Page = https://www.facebook.com/pages/create

## **Step 7**

Create Facebook App =  https://developers.facebook.com/apps/

Add Product **Facebook Messenger**

## **Step 8**

Click **Setup Webhook** and put URL of your Heroku Server

## **Step 9**

### test connection

```
curl -X POST "https://graph.facebook.com/v2.10/me/subscribed_apps?access_token=<PAGE_ACCESS_TOKEN>"
```

## **Step 10**

### add AccessToken 
```javascript
const token = "<PAGE_ACCESS_TOKEN>"
```


### bot sending messege
```javascript
app.post('/webhook/', function (req, res) {
    let messaging_events = req.body.entry[0].messaging
    for (let i = 0; i < messaging_events.length; i++) {
	    let event = req.body.entry[0].messaging[i]
	    let sender = event.sender.id
	    if (event.message && event.message.text) {
		    let text = event.message.text
		    sendTextMessage(sender, "Text received, echo: " + text.substring(0, 200))
	    }
    }
    res.sendStatus(200)
})

function sendTextMessage(sender, text) {
    let messageData = { text:text }
    request({
	    url: 'https://graph.facebook.com/v2.6/me/messages',
	    qs: {access_token:token},
	    method: 'POST',
		json: {
		    recipient: {id:sender},
			message: messageData,
		}
	}, function(error, response, body) {
		if (error) {
		    console.log('Error sending messages: ', error)
		} else if (response.body.error) {
		    console.log('Error: ', response.body.error)
	    }
    })
}

```

## **Step 11**

### deploy heroku again

```
git add .
git commit -m 'updated the bot to speak'
git push heroku master	
```

## **Step 12**

### Improve Authentication

add crypto library
```javascript
const crypto = require('crypto')
```

type app secret
```javascript
const AppSecret = 'APP_YOUR_SECRET';
```

check first
```javascript
app.use(bodyParser.json({verify: verifyRequestSignature}))
```

add function to verivy
```javascript
function verifyRequestSignature(req, res, buf){
  let signature = req.headers["x-hub-signature"];
  
  if(!signature){
    console.error('You dont have signature')
  } else {
    let element = signature.split('=')
    let method = element[0]
    let signatureHash = element[1]
    let expectedHash = crypto.createHmac('sha1', AppSecret).update(buf).digest('hex')

    console.log('signatureHash = ', signatureHash)
    console.log('expectedHash = ', expectedHash)
    if(signatureHash != expectedHash){
      console.error('signature invalid, send message to email or save as log')
    }
  }
}

```

## **Step 13**

Lets Improve!

```javascript
app.post('/webhook/', function (req, res) {
  let data = req.body
  if(data.object == 'page'){
    data.entry.forEach(function(pageEntry) {
      pageEntry.messaging.forEach(function(messagingEvent) {
        if(messagingEvent.message.text){
          sendTextMessage(messagingEvent.sender.id,messagingEvent.message.text);
        } else {
          sendTextMessage(messagingEvent.sender.id,'Service Belum Support Untuk Mendeteksi Hal ini');
        }
      }); 
    });
    res.sendStatus(200)
  }
})
```

## **Step 14**

Improve Again !

```javascript
function sendTextMessage(sender, text) {
  let url = `https://graph.facebook.com/v2.6/${sender}?fields=first_name,last_name,profile_pic&access_token=${token}`;
  
  request(url, function (error, response, body) {
    if (!error && response.statusCode == 200) {
      let parseData = JSON.parse(body);
      let messageData = {
        text: `Hi ${parseData.first_name} ${parseData.last_name}, you send message : ${text}`
      }
      request({
        url: 'https://graph.facebook.com/v2.10/me/messages',
        qs: {
          access_token: token
        },
        method: 'POST',
        json: {
          recipient: {
            id: sender
          },
          message: messageData,
        }
      }, function (error, response, body) {
        if (error) {
          console.log('Error sending messages: ', error)
        } else if (response.body.error) {
          console.log('Error: ', response.body.error)
        }
      })
    }
  })
}
```

# GraphAPI Testing

```
curl -X POST -H "Content-Type: application/json" -d '{
  "recipient": {
    "id": "<SENDER_ID>"
  },
  "message": {
    "text": "hello, world!"
  }
}' "https://graph.facebook.com/v2.6/me/messages?access_token=<ACCESS_TOKEN>"

```

```
curl -X POST -H "Content-Type: application/json" -d '{
  "recipient": {
    "phone_number": "+6281********"
  },
  "message": {
    "text": "hello saya nyobain ya"
  }
}' "https://graph.facebook.com/v2.6/me/messages?access_token=<ACCESS_TOKEN>"
```

```
curl -X POST -H "Content-Type: application/json" -d '{
  "recipient": {
    "id": "<SENDER_ID>"
  },
  "message": {
  "attachment":{
    "type": "audio",
    "payload":{
      "url":"https://www.soundhelix.com/examples/mp3/SoundHelix-Song-2.mp3"
      }
    }
  }
}' "https://graph.facebook.com/v2.6/me/messages?access_token=<ACCESS_TOKEN>"
```

```
curl -X POST -H "Content-Type: application/json" -d '{
  "recipient": {
    "id": "<SENDER_ID>"
  },
  "message": {
  "attachment":{
    "type": "image",
    "payload":{
      "url":"https://scontent-sit4-1.xx.fbcdn.net/v/t1.0-9/19113629_1549630381745758_3709583968208622335_n.jpg?oh=c8dca7028d3be6a8515d6753b0adae9f&oe=5A42A45A"
    }
  }
  }
}' "https://graph.facebook.com/v2.6/me/messages?access_token=<ACCESS_TOKEN>"
```

```
curl -X POST -H "Content-Type: application/json" -d '{
  "recipient": {
    "id": "<SENDER_ID>"
  },
  "message": {
attachment: {
        type: "template",
        payload: {
          template_type: "generic",
          elements: [{
            title: "touch",
            subtitle: "Your Hands, Now in VR",
            item_url: "https://www.oculus.com/en-us/touch/",
            buttons: [{
              type: "web_url",
              url: "https://www.oculus.com/en-us/touch/",
              title: "Open Web URL"
            }, {
              type: "postback",
              title: "Call Postback",
              payload: "Payload for second bubble",
            }]
          }]
        }
      }  
}
}' "https://graph.facebook.com/v2.6/me/messages?access_token=<ACCESS_TOKEN>"

```