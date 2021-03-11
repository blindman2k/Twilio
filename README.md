# Twilio 1.0.1 #

The Twilio library wraps [Twilio’s API](http://www.twilio.com/) for sending and receiving SMS message. To get started with this library, you will need a [Twilio developer account](http://developers.twilio.com/).

Once you’ve created your account, note your Twilio Phone Number, Account SID and Auth Token.

**To add this library to your project, add** `#require "Twilio.class.nut:1.0.1"` **to the top of your agent code**

## Class Usage

## Constructor: Twilio(*AccountSID, AuthToken, TwilioNumber*)

To create a new Twilio object, pass your Account SID, Auth token and Twilio Phone Number to the constructor:

```squirrel
twilio <- Twilio(TWILIO_SID, TWILIO_AUTH, TWILIO_NUM)
```

## Class Methods ##

## send(*numberToSendTo, message, [callback]*) ###

The *send()* method is used to send an SMS message. An optional callback can be passed to the function &ndash; if a callback is supplied, the message will be sent asynchronously, and your callback will fire when the HTTP request completes. The callback function must inlcude a single parameter into which will be passed an a Squirrel table containing three fields: *statuscode*, *headers* and *body*. If you don’t pass a callback, it will execute the request synchronously and return the table outlined above.

```squirrel
numberToSendTo <- "15555555555";
message <- "Hello World!";

// Synchronous send
local response = twilio.send(numberToSendTo, message);
server.log(response.statuscode + ": " + response.body);

// Asynchronous send
twilio.send(numberToSendTo, message, function(response) {
    server.log(response.statuscode + " - " + response.body);
})
```

## respond(*HTTPResponse, [message]*) ###

You can respond to Twilio text messages by using the *respond()* method, which will generate the necessary headers and XML for Twilio to understand your response. You must pass in an impOS [HTTPResponse](https://developer.electricimp.com/api/httpresponse) object, which will be used to respond to Twilio. In the following examples, we’re sending back a message of `"You just said '{original message}'"`, where `{original message}` is whatever the user sent to your Twilio phone number. The code uses the [HTTPResponse](https://developer.electricimp.com/api/httpresponse) object automatically generated by impOS on receiving an HTTP request. If you don’t pass a message, Twilio will send nothing back to the original sender.

```squirrel
// Processing messages
http.onrequest(function(request, response) {
    local path = request.path.tolower();
    if (path == "/twilio" || path == "/twilio/") {
        // Twilio request handler
        try {
            local data = http.urldecode(request.body);
            twilio.respond(response, "You just said '" + data.Body + "'");
        } catch(ex) {
            local message = "Uh oh, something went horribly wrong: " + ex;
            twilio.respond(response, message);
        }
    } else {
        // Default request handler
        response.send(200, "OK");
    }
})
```

**Note** Before the above example will work, you need to configure your Twilio phone number:

 - Click on the phone number you would like to configure on the [Manage Numbers](https://www.twilio.com/user/account/phone-numbers/incoming) dashboard.
 - Change the **Request URL** under **Messaging** to your agent’s URL, then click **Save**. In the example code, we’ve tacked a `/twilio` to the end of the path so we know when the message is coming from Twilio: ```https://agent.electricimp.com/{agentID}/twilio```.

## License ##

This library is licensed under the [MIT License](./LICENSE).
