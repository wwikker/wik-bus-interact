# bhbus-interact

Simple sinatra-based app that talks to Brighton & Hove [live bus times](http://www.buses.co.uk/travel/live-bus-times.aspx) service exposed by [Bus CMS](http://bh.buscms.com)

## API Analysis

Original set of request-response (via burp):

```
GET /api/rest/ent/stop.aspx?callback=jsonp1463771285418&clientid=BrightonBuses&method=search&format=jsonp&q=boundary+road HTTP/1.1
Host: bh.buscms.com
Accept: */*
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_5) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/50.0.2661.102 Safari/537.36
DNT: 1
Referer: http://www.buses.co.uk/travel/live-bus-times.aspx
Accept-Encoding: gzip, deflate, sdch
Accept-Language: en-US,en;q=0.8,pl;q=0.6
Connection: close
```

```
jsonp1463771285418({"result":[{"stopId": "6830", "stopName": "Boundary Road (stop K)", "lat": "50.8329788888889", "lng": "-0.206588055555556", "NaptanCode": "brimajw", "OperatorsCode1": "06532", "OperatorsCode2": "6532", "OperatorsCode3": "30761", "OperatorsCode4": "brimajw"},{"stopId": "7470", "stopName": "Boundary Road (stop A)", "lat": "50.8328584394913", "lng": "-0.207314655184745", "NaptanCode": "brijdaj", "OperatorsCode1": "07532", "OperatorsCode2": "7532", "OperatorsCode3": "21141", "OperatorsCode4": "brijdaj"},{"stopId": "7471", "stopName": "Boundary Road (stop B)", "lat": "50.8335624239899", "lng": "-0.207225619049081", "NaptanCode": "brijtam", "OperatorsCode1": "07533", "OperatorsCode2": "7533", "OperatorsCode3": "22251", "OperatorsCode4": "brijtam"}]});
```

We can slim it down to:

```
$ curl "http://bh.buscms.com/api/rest/ent/stop.aspx?clientid=BrightonBuses&method=search&format=jsonp&q=boundary+road"
```

```
({"result":[{"stopId": "6830", "stopName": "Boundary Road (stop K)", "lat": "50.8329788888889", "lng": "-0.206588055555556", "NaptanCode": "brimajw", "OperatorsCode1": "06532", "OperatorsCode2": "6532", "OperatorsCode3": "30761", "OperatorsCode4": "brimajw"},{"stopId": "7470", "stopName": "Boundary Road (stop A)", "lat": "50.8328584394913", "lng": "-0.207314655184745", "NaptanCode": "brijdaj", "OperatorsCode1": "07532", "OperatorsCode2": "7532", "OperatorsCode3": "21141", "OperatorsCode4": "brijdaj"},{"stopId": "7471", "stopName": "Boundary Road (stop B)", "lat": "50.8335624239899", "lng": "-0.207225619049081", "NaptanCode": "brijtam", "OperatorsCode1": "07533", "OperatorsCode2": "7533", "OperatorsCode3": "22251", "OperatorsCode4": "brijtam"}]});
```

Not only `request` payload is shorter but also the `response` itself is cleaner.

We can also use `searchexact` instead of generic `search`:

```
$ curl "http://bh.buscms.com/api/rest/ent/stop.aspx?clientid=BrightonBuses&method=searchexact&format=jsonp&q=boundary+road+(Stop+B)"
```

```
({"result":[{"stopId": "7471", "stopName": "Boundary Road (stop B)", "lat": "50.8335624239899", "lng": "-0.207225619049081", "NaptanCode": "brijtam", "OperatorsCode1": "07533", "OperatorsCode2": "7533", "OperatorsCode3": "22251", "OperatorsCode4": "brijtam"}]});
```

> NOTE: [NaPTAN](https://data.gov.uk/dataset/naptan) is interesting.

Now that we know how to query bus stops we can proceed to phase 2 where we query the API with `stopid` to get time tables for it. Following is original pair of request-response (via burp):

```
GET /api/REST/html/departureboard.aspx?callback=BusCms.widgets[%27widgetLookupDepartures_stop-departureboardCanvas%27].loadStopTimes_callback&_=1463771615922&clientid=BrightonBuses&sourcetype=siri&stopid=7731&format=jsonp&servicenamefilter= HTTP/1.1
Host: bh.buscms.com
Accept: */*
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_5) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/50.0.2661.102 Safari/537.36
DNT: 1
Referer: http://www.buses.co.uk/travel/live-bus-times.aspx
Accept-Encoding: gzip, deflate, sdch
Accept-Language: en-US,en;q=0.8,pl;q=0.6
Connection: close
```

```
BusCms.widgets['widgetLookupDepartures_stop-departureboardCanvas'].loadStopTimes_callback("<div class=\"livetimes\">  <table class=\"busexpress-clientwidgets-departures-departureboard\">  <tr class=\"rowStopName\"><th colspan=\"3\" title=\"esudgpdj\">Mayfield Avenue</th><tr>  <tr class=\"textHeader\"><th colspan=\"3\">text esudgpdj to 84268 for live times</th><tr>  <tr class=\"rowHeaders\"><th>service</th><th>destination</th><th>time</th><tr>  <tr class=\"rowServiceDeparture\">  <td class=\"colServiceName\">12</td>  <td class=\"colDestination\" title=\"Brighton Stn\">Brighton Stn</td>  <td class=\"colDepartureTime\" data-departureTime=\"20/05/2016 20:19:38\" title=\"5 mins\">5 mins</td>  </tr>  <tr class=\"rowServiceDeparture\">  <td class=\"colServiceName\">12</td>  <td class=\"colDestination\" title=\"Brighton Stn\">Brighton Stn</td>  <td class=\"colDepartureTime\" data-departureTime=\"20/05/2016 20:30:46\" title=\"16 mins\">16 mins</td>  </tr>  <tr class=\"rowServiceDeparture\">  <td class=\"colServiceName\">12</td>  <td class=\"colDestination\" title=\"Brighton Stn\">Brighton Stn</td>  <td class=\"colDepartureTime\" data-departureTime=\"20/05/2016 20:50:39\" title=\"36 mins\">36 mins</td>  </tr>  <tr class=\"rowServiceDeparture\">  <td class=\"colServiceName\">12</td>  <td class=\"colDestination\" title=\"Brighton Stn\">Brighton Stn</td>  <td class=\"colDepartureTime\" data-departureTime=\"20/05/2016 21:10:39\" title=\"56 mins\">56 mins</td>  </tr>  </table></div>  <div class=\"scrollmessage_container\"><div class=\"scrollmessage\"></div></div>  <div class=\"services\"><a href=\"#\" onclick=\"BusCms.widgets['#domid#'].serviceNameClick('');\" class=\"service selected\">all</a>   <a href=\"#\" onclick=\"BusCms.widgets['#domid#'].serviceNameClick('12');\" class=\"service\">12</a> <a href=\"#\" onclick=\"BusCms.widgets['#domid#'].serviceNameClick('14');\" class=\"service\">14</a> <a href=\"#\" onclick=\"BusCms.widgets['#domid#'].serviceNameClick('14C');\" class=\"service\">14C</a> <a href=\"#\" onclick=\"BusCms.widgets['#domid#'].serviceNameClick('76A');\" class=\"service\">76A</a> <a href=\"#\" onclick=\"BusCms.widgets['#domid#'].serviceNameClick('12A');\" class=\"service\">12A</a> <a href=\"#\" onclick=\"BusCms.widgets['#domid#'].serviceNameClick('494');\" class=\"service\">494</a>   </div>  ");
```

Again we can slim it down to:

```
$ curl --globoff "http://bh.buscms.com/api/REST/html/departureboard.aspx?clientid=BrightonBuses&sourcetype=siri&stopid=7731&format=jsonp"
```

```
<div class=\"livetimes\">  <table class=\"busexpress-clientwidgets-departures-departureboard\">  <tr class=\"rowStopName\"><th colspan=\"3\" title=\"esudgpdj\">Mayfield Avenue</th><tr>  <tr class=\"textHeader\"><th colspan=\"3\">text esudgpdj to 84268 for live times</th><tr>  <tr class=\"rowHeaders\"><th>service</th><th>destination</th><th>time</th><tr>  <tr class=\"rowServiceDeparture\">  <td class=\"colServiceName\">12</td>  <td class=\"colDestination\" title=\"Brighton\">Brighton</td>  <td class=\"colDepartureTime\" data-departureTime=\"21/05/2016 16:52:47\" title=\"5 mins\">5 mins</td>  </tr>  <tr class=\"rowServiceDeparture\">  <td class=\"colServiceName\">12A</td>  <td class=\"colDestination\" title=\"Brighton\">Brighton</td>  <td class=\"colDepartureTime\" data-departureTime=\"21/05/2016 16:53:18\" title=\"6 mins\">6 mins</td>  </tr>  <tr class=\"rowServiceDeparture\">  <td class=\"colServiceName\">14C</td>  <td class=\"colDestination\" title=\"Churchill Sq\">Churchill Sq</td>  <td class=\"colDepartureTime\" data-departureTime=\"21/05/2016 16:55:05\" title=\"8 mins\">8 mins</td>  </tr>  <tr class=\"rowServiceDeparture\">  <td class=\"colServiceName\">12</td>  <td class=\"colDestination\" title=\"Brighton\">Brighton</td>  <td class=\"colDepartureTime\" data-departureTime=\"21/05/2016 17:11:40\" title=\"24 mins\">24 mins</td>  </tr>  <tr class=\"rowServiceDeparture\">  <td class=\"colServiceName\">12</td>  <td class=\"colDestination\" title=\"Brighton\">Brighton</td>  <td class=\"colDepartureTime\" data-departureTime=\"21/05/2016 17:23:12\" title=\"36 mins\">36 mins</td>  </tr>  <tr class=\"rowServiceDeparture\">  <td class=\"colServiceName\">12</td>  <td class=\"colDestination\" title=\"Brighton\">Brighton</td>  <td class=\"colDepartureTime\" data-departureTime=\"21/05/2016 17:33:12\" title=\"46 mins\">46 mins</td>  </tr>  <tr class=\"rowServiceDeparture\">  <td class=\"colServiceName\">12A</td>  <td class=\"colDestination\" title=\"Brighton\">Brighton</td>  <td class=\"colDepartureTime\" data-departureTime=\"21/05/2016 17:38:42\" title=\"51 mins\">51 mins</td>  </tr>  </table></div>  <div class=\"scrollmessage_container\"><div class=\"scrollmessage\"></div></div>  <div class=\"services\"><a href=\"#\" onclick=\"BusCms.widgets['#domid#'].serviceNameClick('');\" class=\"service selected\">all</a>   <a href=\"#\" onclick=\"BusCms.widgets['#domid#'].serviceNameClick('12');\" class=\"service\">12</a> <a href=\"#\" onclick=\"BusCms.widgets['#domid#'].serviceNameClick('14');\" class=\"service\">14</a> <a href=\"#\" onclick=\"BusCms.widgets['#domid#'].serviceNameClick('14C');\" class=\"service\">14C</a> <a href=\"#\" onclick=\"BusCms.widgets['#domid#'].serviceNameClick('76A');\" class=\"service\">76A</a> <a href=\"#\" onclick=\"BusCms.widgets['#domid#'].serviceNameClick('12A');\" class=\"service\">12A</a> <a href=\"#\" onclick=\"BusCms.widgets['#domid#'].serviceNameClick('494');\" class=\"service\">494</a>   </div>
```

EOF.

## Deployment

Easy-peasy with [Heroku](https://devcenter.heroku.com/articles/getting-started-with-ruby-o).
