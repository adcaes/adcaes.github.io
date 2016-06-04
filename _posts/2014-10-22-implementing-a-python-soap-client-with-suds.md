---
layout: post
title:  "Implementing a Python SOAP client with Suds"
date:   2014-10-22
tags: [Programming, Python]
---

This is a small introduction on how to implement a [SOAP](http://en.wikipedia.org/wiki/SOAP) client with Python. Most likely you are not thrilled with this task, specially if you are used to the simplicity of Python and REST, but it is not as bad if you use the right library.

After investigating a few options I decided for [Suds](https://fedorahosted.org/suds/wiki/Documentation), seeming quite active and simple to use. The original Suds library which is no longer maintained, but there is an actively maintained fork of suds, [suds-jurko](https://bitbucket.org/jurko/suds/), and is the one I use. I will explain how to build a SOAP client with Suds with a simple example.

## The service

Our client will interact with an example SOAP service that provides the current weather of a given city. As you may know, SOAP services can be described with a [WSDL](https://en.wikipedia.org/wiki/Web_Services_Description_Language) file that contains a XML specification of the operations supported by the service. Our example service supports one operation, getCityWeather, that requires a country and a city. It returns the temperature, weather and update time of the requested city. [Here](https://github.com/adcaes/blog/blob/master/soap_suds_client/weather.wsdl) you can see the WSDL file.

## The client

To implement the client first of all instantiate the SUDS Client class, providing the location of the WSDL file. The WDSL can be served at a URL or as a local file, both options directly supported.

```python
from suds.client import Client
client = Client('http://localhost:8080/weather?wsdl')
```

The SUDS client exposes all the operations provided by the service as functions and provides a factory to instantiate the data types defined in the WSDL file.

Using the client instance we can instantiate a CityWeatherRequest defined in the WSDL, and set the data that will be sent in the request, in this case City and Country, both strings.

```python
request_data = client.factory.create('s1:CityWeatherRequest')
request_data.City = "Stockholm"
request_data.Country = "Sweden"
```

Finally we issue the getCityWeather request by calling the function in the client instance with the request data as a parameter. The result is a Python class that contains all the fields defined in the response type for the getCityWeather operation in the WSDL file.

```python
result = client.service.getCityWeather(request_data)
```
So that's it, the code for the service and the client is available [here](https://github.com/adcaes/blog/tree/master/soap_suds_client). You can run it locally in your machine by installing the requirements with pip and running server.py and client.py.
