# Hackathon SmartServices
We are happy you joined the SmartServices Universe and want to participate in the future of urban mobility. This document will provide you some short introduction on how to launch your own smart service. If you have any questions feel free to ask our guides!

## tl;dr;
* [SmartServices API-Doc (Vega)](https://www.prod.smartservices.car2go.com/vega/swagger-ui.html)
* [Vehicle Mock App (MockA)](https://www.prod.smartservices.car2go.com/mocka/)
* Open vehicle in MockA *before* accessing it via the api.
* `curl https://api.prod.smartservices.car2go.com/vega/service/check --cert pixelcamp.pem`
* `curl -X PUT 'https://api.prod.smartservices.car2go.com/vega/vehicles/MOCKPIXEL017/doors/unlock' --cert pixelcamp.pem`
* `curl  -X GET 'https://api.prod.smartservices.car2go.com/vega/events?vin=MOCKPIXEL017&since=2017-09-01T08:15:42.000Z&cursor=42&limit=5' --cert pixelcamp.pem`
* `curl  -X GET 'https://api.prod.smartservices.car2go.com/vega/vehicles/MOCKPIXEL017?fields=batteryLevel,connection.connected,connection.since,doors.allClosed,doors.leftOpen,doors.locked,doors.rightOpen,doors.trunkOpen,engineOn,fuelLevel,geo.latitude,geo.longitude,ignitionOn,immobilizerEngaged,mileage,powerState,vin' --cert pixelcamp.pem`

#### Names
| Word    | Meaning                               |
|---------|---------------------------------------|
| VIN     | Vehicle Identification Number         |
| Command | Message sent **to** the vehicle       |
| Event   | Message sent **from** the vehicle     |
| Vega    | Name of the SmartServices API Service |
| MockA   | Name of the Vehicle Mock App          |

## Inventory
Things included in the SmartServices Package you received:
* This document (obviously)
* Combined PEM containing client certificate and private key (pixelcamp.pem)
* Client Key as PKCS12 for use in browser (pixelcamp.pfx) (without password)
* Private key (password smartcode)
* Credentials for MockA (credentials.txt)

## Useful links
* http://dillinger.io
  view this document in the online editor it was made with.

* https://jsonlint.com  
  check and show ugly json in a beautiful way.

* https://curl.haxx.se/docs/manpage.html  
  manpage for cURL. Mostly you will need `-v`, `--cert`, `-X PUT` and `-d"body to send"`.  

* https://www.getpostman.com/ 
  if you still need to learn that terminal is love, terminal is life and want to send HTTP request with a GUI.  

## API
To implement extraordinary services you need a powerful API which enables you to perform various and detailed requests. A detailed overview of what (and how) you can do with the SmartServices Platform is written down in our [API-Doc](https://www.prod.smartservices.car2go.com/vega/swagger-ui.html). This will possibly be the most important document you will have during developing a smart service.
> **Hint:** Import the .pfx certificate in your browser (leave password blank) to be able to execute REST calls with out API Doc. After importing, make sure your access our api here: [https://api.prod.smartservices.car2go.com/vega/swagger-ui.html](https://api.prod.smartservices.car2go.com/vega/swagger-ui.html)

## Client-Certificate
All REST endpoints are protected by client certificate authentication. Admittedly, this will be some obstacle while bootstrapping your service - sorry for that - but it ensures a great level of access security and once setup correctly, it will easily and transparently grant access for your service.  
To check if your client cert setup works you can just call `https://api.prod.smartservices.car2go.com/vega/service/check`. Either you get a response or the connection gets declined by a client handshake failure.  
You can check your connection by calling the check endpoint with following curl request:
```sh
$ curl 'https://api.prod.smartservices.car2go.com/vega/service/check' --cert pixelcamp.pem
```
In the best case, you will get a response looking like this:
```json
{
	"certProperties": {
		"Subject CN": "pixelcamp"
	},
	"result": "Certificate is valid and therefore you are able to communicate with the server."
}
```

> You will find your client certificate and private key in a combined pem-file in your SmartServices package.

## Sending Commands
Commands are messages you can send to the vehicle. They will trigger actions like blinking with the direction indicators, (un)locking the doors or (dis)engaging the immobilizer. Some of them will be answered by the car sending its own event back to the server.

## Receiving Events
Events are messages sent from the vehicle to the server. Whenever something happens the car wants to tell you, it will send an event. You can view the events by requesting the event endpoint in Vega. The event queue in Vega is cursor based. This means that the events won't get deleted when you fetch them. You can fetch them as often as you want (for the next 30 days), just append the right query parameters like `cursor`, `since`, and `limit` and pick the events you are interested in. See below for an example or the API-Doc for further information.

## Vehicle Mock App (MockA)
For easy testing without real vehicles we built [*MockA*](https://www.prod.smartservices.car2go.com/mocka/) for you. With MockA, on the one hand, you can create some noise on the event queue by simulating vehicle interaction (e.g. open/close doors). On the other hand, you can send commands to a mocked (simulated) vehicle you attached to MockA and see how it reacts.  

> You will find your MockA credentials in the file credentials.txt in your SmartServices package.

## Full Example:
Log in to MockA with your team credentials and attach the provided vehicle MOCKPIXEL017 (with xxx being your team number). Now, you can try to send an unlock command to the vehicle mock:
```sh
$ curl -X PUT 'https://api.prod.smartservices.car2go.com/vega/vehicles/MOCKPIXEL017/doors/unlock' --cert pixelcamp.pem
```
Like a real vehicle we expect the mocked vehicle to unlock (represented by a status change and log entry in MockA) and send a `statusChange` event to the event queue. Let's fetch it:
```sh
$ curl  -X GET 'https://api.prod.smartservices.car2go.com/vega/events?vin=MOCKPIXEL017&since=2017-09-26T08:15:42.000Z' --cert pixelcamp.pem
```
> **Hint:** Do not forget to adept the `vin` and `since` parameters or just omit them if you don't expect (m)any events in the queue.

If everything worked fine you will get a response looking similar to this:
```json
{
	"nextCursor": 0,
	"events": [{
		"vin": "MOCKPIXEL017",
		"type": "statusChange",
		"created": "2017-09-01T13:37:54.321Z",
		"event": {
			"doors": {
				"locked": false
			}
		},
		"cursor": 17
	}]
}
```
*Congratulations!*  
You just managed to perform a full command-event-roundtrip and are ready to implement some kick-a** service!
