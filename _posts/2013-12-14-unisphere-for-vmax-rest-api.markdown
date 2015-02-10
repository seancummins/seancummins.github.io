---
layout: post
title: Unisphere for VMAX REST API
date: 2013-12-14 17:02:54.000000000 -05:00
categories:
- Automation
tags: []
status: publish
type: post
published: true
meta:
  _edit_last: '1'
  _post_restored_from: a:3:{s:20:"restored_revision_id";i:13;s:16:"restored_by_user";i:1;s:13:"restored_time";i:1387201809;}
author:
  login: scummins
  email: sean@scummins.com
  display_name: Sean Cummins
  first_name: Sean
  last_name: Cummins
---
*Note: This post primarily covers the REST API in Unisphere for VMAX versions 1.6.x and prior. The REST API was overhauled in Unisphere for VMAX version 8.x, but still maintains backwards compatibility with the old API in 1.6.x.*

This post will show you how to programmatically retrieve diagnostic performance data from the Unisphere for VMAX REST API. The official documentation for the REST API is in the [Unisphere for VMAX REST API Programmer's Guide](https://support.emc.com/docu47002_Unisphere-for-VMAX-1.6-REST-API-Programmer's-Guide.pdf?language=en_US), but as an infrastructure guy, I found this document difficult to interpret.

One simple way to access it is via the [Mozilla RESTClient](https://addons.mozilla.org/en-US/firefox/addon/restclient/); a Firefox add-on. Once you've installed the add-on, you can import [this]({{ site.url }}/assets/RESTClient_Favorite_RESTAPI_Queries_v1.json) JSON "favorites" file, and it will load a bunch of queries into your RESTClient. You'll need to modify the IP address, authentication information, and Symm serial number to point to your instance of Unisphere for VMAX.

And of course you can modify the queries to pull different metrics from various object categories. You can find the list of available categories and metrics in the schema, which can be downloaded from the Uni4VMAX instance. Just point your browser to `https://<unisphere_IP_address>:8443/univmax/restapi`, and it will pull down a zip file. In that zip you'll find a file called "performance.xsd" with the schema info.

Here's some info on how to interpret the contents of this file.

For every "class" of component (e.g. Array, FEDirector, BEDirector, Disk, Storage Group, etc), you'll see two sections within the file. The sections are identified by comment fields. One section is for the "Keys" of that particular component type, and another section is for the "Metrics" of that component type.

The Keys let you enumerate the component ID's that actually exist (e.g. You can get a list of all the FE directors in a particular Symmetrix), and the Metrics let you pull specific performance metrics for a given object (e.g. You can pull the "IO_RATE" for FA-7E).

You can make calls to the API using XML or JSON. I think JSON is simpler and more readable, so that's what I primarily used.

Here's what the FEDirector Keys section of the schema looks like. The only required parameter here (defined in the ParamType section) is the symmetrixId. You have to specify the full SymmID with leading zeros, and without any factory letter designations like "HK" or "CK" — basically the same format that you'd see in a "symcfg list".

![screenshot1]({{ site.url }}/assets/u4v_rest_fakeys.png)

In this case, you need to issue a POST to the URL `https://<IPAddr>:8443/univmax/restapi/performance/FEDirector/keys`.

The headers for your post need to include the following (the garbage string for authentication is just "smc:smc" encoded in [Base64](http://www.base64encode.org/) — your client will typically do this encoding for you):

* **Authorization**: Basic c21jOnNtYw==
* **Content-Type**: application/json
* **Accept**: application/json

And the body of your POST should look like this (modify the SymmID as necessary):

~~~json
{
  "feDirectorKeyParam":{
    "symmetrixId" : "000195700235"
    }
}
~~~


When you execute this POST, you should get back something like the snippet below -- a list of all the FA's in this Symmetrix. Note that you also get "firstAvailable" and "lastAvailable" timestamps (in ['milliseconds from epoch'](http://www.epochconverter.com/) format) -- this shows the time range of performance data that is available for that object. You'll need to specify a time range when you pull metrics in the next step.

~~~json
{
  "feDirectorKeyResult": {
    "feDirectorInfo": [
    {
      "directorId": "FA-5E",
      "firstAvailableDate": 1385586600000,
      "lastAvailableDate": 1386192000000
    },
    {
      "directorId": "FA-6E",
      "firstAvailableDate": 1385586600000,
      "lastAvailableDate": 1386192000000
    }]
}}
..etc
~~~

Now that you know what FA's exist, you can pull metrics from one of them. To do this, examine the schema again, but this time look at the FEDirector Metrics section.

![screenshot2]({{ site.url }}/assets/u4v_rest_fametrics.png)

![screenshot3]({{ site.url }}/assets/u4v_rest_fakeyparams.png)


So say we want to pull all of the IO_RATE values for FA-7E. Again you'll issue a POST to do this.

The headers are the same as before.

The URL changes to `https://<IPAddr>:8443/univmax/restapi/performance/FEDirector/metrics`.

And the body becomes:

~~~json
{
  "feDirectorParam":{
    "startDate" : "1385586600000",
    "endDate" : "1386192000000",
    "symmetrixId" : "000195700235",
    "directorId" : "FA-7E",
    "metrics" : ["IO_RATE"]
  }
}
~~~

You should get back something like this… an entry for each time interval within the start/end range you specified, showing the value for the metric you specified. You can request multiple metrics at once by separating them with a comma within the square brackets — e.g. ["IO_RATE","READS","WRITES"].

~~~json
{
  "iterator": {
    "resultList": {
      "result": [
        {
          "IO_RATE": 0,
          "timestamp": 1386006300000
        },
        {
          "IO_RATE": 0,
          "timestamp": 1386006600000
        }]
}}}
..etc
~~~


These requests can be issued with pretty much any client that can issue an HTTP POST… I think the Mozilla RESTClient is a simple way to demonstrate/explore the API, but you could also use things like Python or curl.

Here's an example of a request for all FA Keys using the Mozilla RESTClient --

![screenshot4]({{ site.url }}/assets/ss3.5.png)

Finally, here's that same request using curl. The output isn't pretty, but it works --

![screenshot5]({{ site.url }}/assets/ss4.png)
