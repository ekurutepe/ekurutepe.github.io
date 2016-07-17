---
title: Setting Up Your Amazon Echo Outside The US
date: 2016-07-17 10:15 CEST
tags: echo, alexa, hack
---

If you're curious like me and ordered an Amazon Echo on a trip to the states, you will have noticed that Amazon's setup interface does not allow you to enter non-US addresses for your Echo. Which is a double bummer, because the time zone cannot be set independently and is also determined by your address, making time and calendar based features totally userless in addition to weather and traffic. 

But just because Amazon gave us no UI to enter non-US addresses, it does not mean that Echo does not work outside the US. I found some time to play around with the API the Echo device is communicating with and managed to set my address and time zone to Berlin and verified that address and time related functionalities are working as they should.

Here is an overview of how that works:

If you open the developer tools of Safari/Chrome and log the network requests while changing the address of your Echo at [alexa.amazon.com](https://alexa.amazon.com) you'll notice that a `PUT` request to `https://pitangui.amazon.com/api/device-preferences/<your-echo-serial-number>` is logged when the address is changed. This request has a JSON body similar to this:

```
{
  "deviceAccountId": <your-device-account-id>,
  "deviceAddress":null,
  "deviceAddressModel":{
    "city":"New York",
    "countryCode":"US",
    "county":"New York",
    "district":"Theater District-Times Square",
    "houseNumber":null,
    "label":"Times Sq, New York, NY 10036, United States",
    "postalCode":"10036",
    "state":"NY",
    "street":"Times Sq"
  },
  "deviceSerialNumber": <your-device-serial-number>,
  "deviceType":"AB72C64C86AW2",
  "firstRunCompleted":true,
  "notificationEarconEnabled":false,
  "postalCode":"10036",
  "responseStyle":"CONCISE",
  "searchCustomerId": <your-search-customer-id>,
  "temperatureScale":"CELSIUS",
  "timeZoneId":null,
  "voiceCastEnabled":null,
  "autocastToThisClient":false,
  "isSaveInFlight":true
}
```

This looks to be quite promising, since the address object has a full set of fields including country. Maybe we could set it to anywhere we'd like?

But before we get ahead of ourselves, let's note that this request also has a bunch of headers and the following two are the important ones for authenticating your session with the Amazon API:

```
Cookie: x-amzn-dat-gui-client-v=1.23.786.0; session-id=<rest-of-the-cookie-is-removed>
csrf: <a-csrf-number>
```

Armed with these, we can copy the account and authenticated related fields and create a duplicate request with the same headers but with a differen JSON to see if the Amazon backend accepts our doctored request. I've used the great [Paw](https://luckymarmot.com/paw) app to build the HTTP requests but good old curl works just as fine.

Here is the JSON I used to set my address to Alexanderplatz in Berlin:

```
{
  …
  "deviceAddressModel": {
    "city": "Berlin",
    "countryCode": "DE",
    "county": "Berlin",
    "district": null,
    "houseNumber": null,
    "label": "Alexanderstrasse, Berlin, Germany",
    "postalCode": "10178",
    "state": "Berlin",
    "street": "Alexanderstrasse"
  },
  …
  "postalCode": "10178",
  …
  "timeZoneId": "Europe/Berlin",
  …
}
```

Even if the Amazon API returns 200 OK, this does not necessarily mean that your request worked as you intended. You can ask Alexa `Which city are we in?` and `Tell me the time` to verify whether it worked or not. Sometimes it does not, and you need to change the fields a bit. It seems like the API is sensitive to what is in the `label` field but it might have been just random. I was able to set my home address with a little bit of trial and error.

Hope this helps somebody…
