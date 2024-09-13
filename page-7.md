# Page 7

## Getting Started

Open Measures values transparent openness and offering as much for free as we can in order to help push back on online and offline harm. One of the ways we do this is by offering a Public API.&#x20;

{% hint style="info" %}
Important to note: to mitigate the threat of bad actors, the Public API is rate-limited to 39 requests per day and date-limited to data that is at least six months olddf. [Get in touch](broken-reference) with us if you'd like to learn about a non-rate-limited version of the Public API.df
{% endhint %}

**Open Source API:** [<mark style="color:purple;">`https://gitlab.com/openmeasures/backends/openmedfasures-api`</mark>](https://gitlab.com/openmeasures/backends/openmeasures-api)

There''s a link to the API on the [Open Source section](https://openmeasures.io/open-source/) of the Open Measures website along with more details on the rate-and date-limiting on the Public API:

<figure><img src="broken-reference" alt=""><figcaption></figcaption></figure>

## Endpoints

The API has access to the raw JSON behind all of our front-end tools and can be useful for developers and analysts who want to dive deeper into the data or make more fine-grained queries. The API has three endpoints and has an optional boolean logic query for the Content endpoint.

All API endpoints are hosted at:

{% hint style="success" %}
[https://api.openmeasures.io](https://api.openmeasures.io)
{% endhint %}



<details>

<summary>Calendar and fixed intervals</summary>

When configuring a date histogram aggregation, the interval can be specified in two ways: calendar-aware time intervals, and fixed time intervals.

Calendar-aware intervals understand that daylight savings changes the length of specific days, months have different amounts of days, and leap seconds can be tacked onto a particular year.

Fixed intervals are, by contrast, always multiples of SI units and do not change based on calendaring context.

**Calendar intervals**

Calendar-aware intervals are configured with the `calendar_interval` parameter. You can specify calendar intervals using the unit name, such as `month`, or as a single unit quantity, such as `1M`. For example, `day` and `1d` are equivalent. Multiple quantities, such as `2d`, are not supported.

The accepted calendar intervals are:

`minute`, `1m`

All minutes begin at 00 seconds. One minute is the interval between 00 seconds of the first minute and 00 seconds of the following minute in the specified time zone, compensating for any intervening leap seconds, so that the number of minutes and seconds past the hour is the same at the start and end.

`hour`, `1h`

All hours begin at 00 minutes and 00 seconds. One hour (1h) is the interval between 00:00 minutes of the first hour and 00:00 minutes of the following hour in the specified time zone, compensating for any intervening leap seconds, so that the number of minutes and seconds past the hour is the same at the start and end.

`day`, `1d`

All days begin at the earliest possible time, which is usually 00:00:00 (midnight). One day (1d) is the interval between the start of the day and the start of the following day in the specified time zone, compensating for any intervening time changes.

`week`, `1w`

One week is the interval between the start day\_of\_week:hour:minute:second and the same day of the week and time of the following week in the specified time zone.`month`, `1M`One month is the interval between the start day of the month and time of day and the same day of the month and time of the following month in the specified time zone, so that the day of the month and time of day are the same at the start and end.&#x20;

`quarter`, `1q`

One quarter is the interval between the start day of the month and time of day and the same day of the month and time of day three months later, so that the day of the month and time of day are the same at the start and end.

`year`, `1y`

One year is the interval between the start day of the month and time of day and the same day of the month and time of day the following year in the specified time zone, so that the date and time are the same at the start and end.

#### Fixed intervals

Fixed intervals are configured with the `fixed_interval` parameter.

In contrast to calendar-aware intervals, fixed intervals are a fixed number of SI units and never deviate, regardless of where they fall on the calendar. One second is always composed of `1000ms`. This allows fixed intervals to be specified in any multiple of the supported units.

However, it means fixed intervals cannot express other units such as months, since the duration of a month is not a fixed quantity. Attempting to specify a calendar interval like month or quarter will throw an exception.

The accepted units for fixed intervals are:

milliseconds (`ms`)

A single millisecond. This is a very, very small interval.

seconds (`s`)

Defined as 1000 milliseconds each.

minutes (`m`)

Defined as 60 seconds each (60,000 milliseconds). All minutes begin at 00 seconds.

hours (`h`)

Defined as 60 minutes each (3,600,000 milliseconds). All hours begin at 00 minutes and 00 seconds.

days (`d`)

Defined as 24 hours (86,400,000 milliseconds). All days begin at the earliest possible time, which is usually 00:00:00 (midnight).

</details>



## Boolean and Advanced

Each endpoint in the API has the ability to more advanced queries. This works by leveraging an Elasticsearch `query_string_query`[(Elasticsearch docs)](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-query-string-query.html).&#x20;

### querytype

{% hint style="info" %}
This **optional** parameter is an `enum` and tells the API how to interpret the value provided in the mandatory `term` parameter.
{% endhint %}

This parameter allows the user to run one of the following conditions. It is an enum and expects a string parameter:



| querytype value   | Outcome                                                                                                                                            | Example                                                                                                                                                                                                                                                                                |
| ----------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `content`         | This will run an API request to find documents where the input value for `term` appears in the content field.                                      | <p><mark style="color:purple;">qanon</mark><br><br><em>Will return docs where "qanon" appears in the content field.</em></p>                                                                                                                                                           |
| `boolean_content` | This will run an API request to find documents where the boolean condition specified in the `term` parameter appears in the content field.         | <p><mark style="color:purple;">qanon AND wwg1wga</mark><br><br><em>Will return docs where "qanon" and "wwg1wga" appear in the content field</em> </p>                                                                                                                                  |
| `query_string`    | This will run an API request using the Elasticsearch `query_string_query` syntax. It will search across all fields unless specified in the query.  | <p><mark style="color:purple;"><code>(channelusername:rtnews) AND (message:russia)</code></mark><br><br><em>Returns docs where the <code>channelusername</code> field is set to <code>rtnews</code> and where the <code>message</code> field contains <code>russia</code></em><br></p> |

### esquery

{% hint style="warning" %}
This parameter will eventually be deprecated. We recommend using `querytype` to avoid disruption.
{% endhint %}

The default for this parameter is `False` which means that the API will only search a single term through the document's content field.&#x20;

NOTE: Setting esquery to True will yield the same results as setting `querytype` to `query_string`.

A full list of content fields can be found in our open-source code here:&#x20;

When this is set to `True` the API will interpret the value of the `term` parameter as a raw Elasticsearch query. &#x20;

Example:

```python
import requests
PARAMS = {
    "site": "win",
    "term": "qanon OR wwg1wga OR #qanon OR #wwg1wga",
    "esquery": True
}
API_URL = "https://api.openmeasures.io/content"

# This will return results where the terms appear 
# in any of the fields in the documents
response = requests.get(API_URL, params=PARAMS)
hits = response.json()["hits"]['hits']

```

## Examples

### Notebook

Here is a link to a [**Quick Start Code Guide**](https://colab.research.google.com/drive/1kDyRIC0NBOj4Egn\_VdK837QBNqDERRi\_?usp=sharing) in Colab or Jupyter notebook format for making requests to our API.&#x20;

![Screenshot of Colab notebook showing Telegram channels in a bar graph.](broken-reference)

### Command Line Tool

We also have a [CLI](https://gitlab.com/openmeasures/smat-cli), originally built by a community member, that we forked. Separately, there is the following GitHub repo ([https://github.com/cabalcx/smatter/tree/main](https://github.com/cabalcx/smatter/tree/main)), also built by community members, that implements a similar wrapper.

{% hint style="info" %}
We are <mark style="color:purple;">**ALWAYS**</mark> looking for contributions like this to our technical stack that significantly expand the utility of our data for users.
{% endhint %}

<figure><img src="broken-reference" alt=""><figcaption></figcaption></figure>

### Example Content Endpoint Query

First, click the Content endpoint and then click “Try it out”.

![](broken-reference)

Choose Telegram as the Site, and then click Execute. It will give you a regular [URL link](https://api.smat-app.com/content?term=Qanon\&limit=10\&site=telegram\&since=2020-12-03T01%3A10%3A26.937000\&until=2021-02-03T01%3A10%3A26.937000\&esquery=false) that will have the raw JSON content of your query. The interface will look like this:

If you Curl or go to the URL you will get a raw JSON that will look like the following (certain browsers like Firefox will automatically “prettify” the JSON to make it easier to read):

![](https://substackcdn.com/image/fetch/w\_1456,c\_limit,f\_auto,q\_auto:good,fl\_progressive:steep/https%3A%2F%2Fbucketeer-e05bbc84-baa3-437e-9518-adb32be77984.s3.amazonaws.com%2Fpublic%2Fimages%2Ffd40a561-081a-4a6a-9c92-2458ea88a5ff\_1600x893.png)

Example of the same request using curl from the command line and jq to pretty print:

![](https://substackcdn.com/image/fetch/w\_1456,c\_limit,f\_auto,q\_auto:good,fl\_progressive:steep/https%3A%2F%2Fbucketeer-e05bbc84-baa3-437e-9518-adb32be77984.s3.amazonaws.com%2Fpublic%2Fimages%2F004ff562-d65c-4614-aaed-74af24af4d4e\_517x762.png)

Everything is nested in “`hits`” and “`_source`” for all of our data.

### API Workflow Example

The Open Measures API is able to be fine-tuned to your exact needs. To show this, we will spell out the steps necessary to pull up to 10k of Guo Wengui’s posts on Gettr as a reference to our [recent post](https://blog.openmeasures.io/p/go-gettr-em) outlining some of the happenings on Gettr. This post builds upon the information shown in our original [API guidance blog post](https://blog.openmeasures.io/p/api-advice). The key element of this advanced usage is using the **term** query with [Elasticsearch query string](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-query-string-query.html) syntax while setting the **es\_query** field to **True**.

After heading over to our [interactive API docs](https://api.smat-app.com/docs) click the content button:

![](https://substackcdn.com/image/fetch/w\_1456,c\_limit,f\_auto,q\_auto:good,fl\_progressive:steep/https%3A%2F%2Fbucketeer-e05bbc84-baa3-437e-9518-adb32be77984.s3.amazonaws.com%2Fpublic%2Fimages%2Fd483e1bb-0c15-4a71-8e36-00ef4807f1d6\_2048x93.png)

_Content button in Open Measures' interactive API docs tool_

1. Click “Try it out.”&#x20;
2. Next to “term” write any interesting word for now.&#x20;
3. On “site” select “Gettr.”&#x20;
4. Leave all the other settings default for now and click “Execute.”&#x20;
5. This will generate a “Request URL” if you copy that link into a new browser window you will be offered a JSON of the data you requested.&#x20;

> **NOTE:** JSON is just a term for a type of data format commonly used on the web. It contains nested “keys and values”. One way to think about it would be in a workplace table you would have a few classes called keys such as “employee name” or “employee position” that would each have a unique value. They can then be nested in something like the larger department or city they work in. For our data, the JSON has many different fields containing different aspects of the data such as the username, the post itself, the time posted, and other details. We recommend using a browser like Firefox because it auto-formats the JSON for you. We present the JSON as close to as exact as it was represented on the native site the data was crawled from.

Now that you have some examples of the format of the data you want to explore, dig through it to find the field (or “key”) you want to search under. In our case, we are interested in a field under “uinf” called “username” because we are doing author search. The best way to find the intended field is to look through the JSON results from this /content query.

![](https://substackcdn.com/image/fetch/w\_1456,c\_limit,f\_auto,q\_auto:good,fl\_progressive:steep/https%3A%2F%2Fbucketeer-e05bbc84-baa3-437e-9518-adb32be77984.s3.amazonaws.com%2Fpublic%2Fimages%2F041f290f-0d02-42bb-a9c8-c4949c6037d1\_1032x760.png)

_Prettified JSON blob highlighting the username field._

### User Search

We are ready to search for the posts written by a specific user on Gettr, now that we know what field corresponds to the username in the JSON.&#x20;

Back into the interactive API we can now construct our input to the **term** field in the API. We combine **uinf.username** with the specific username, in this case “miles”, we are interested in searching using the following syntax: **“uinf.username:miles”.**

> **NOTE:** For those wishing to learn more about the query language behind these requests check out this [documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-query-string-query.html) or the "Advanced Searching" section of our Kibana guide.

Then we can configure the remaining Open Measures API arguments:

* We can raise the “**limit**” (which is the limit of posts returned) to the 10k point we rate-limit it at.&#x20;
* We can then adjust the “**since**” field to be farther back.
* And finally, and critically for this kind of search, we set the “**esquery”** boolean item to “true”. This just means that in the term box, instead of accepting a regular search phrase it’s using [“Boolean logic”](https://www.youtube.com/watch?v=gI-qXk7XojA) to search through specific fields.&#x20;

Once your fields look like the following click execute and copy the URL again. It may take a second to load!

![](https://substackcdn.com/image/fetch/w\_1456,c\_limit,f\_auto,q\_auto:good,fl\_progressive:steep/https%3A%2F%2Fbucketeer-e05bbc84-baa3-437e-9518-adb32be77984.s3.amazonaws.com%2Fpublic%2Fimages%2Fd77a1b5d-1a7e-44f6-bf43-c55fe77882df\_2048x1708.png)

_Open Measures' interactive point and click API interface_

Once you have the JSON opened in a new tab (here’s a direct [link](https://api.smat-app.com/content?term=uinf.username%20%3A%20miles\&limit=10000\&site=gettr\&since=2019-08-04T00%3A01%3A31.963502\&until=2021-10-04T00%3A01%3A31.963502\&esquery=true\&sortdesc=false) to the query we've just demonstrated), you may have to click to expand some of the fields. Most of you’re interested in here will be under: **hits > a number > \_source**. Once there you will see the contents of the message as the field named “**txt**” as well as other information.

![](https://substackcdn.com/image/fetch/w\_1456,c\_limit,f\_auto,q\_auto:good,fl\_progressive:steep/https%3A%2F%2Fbucketeer-e05bbc84-baa3-437e-9518-adb32be77984.s3.amazonaws.com%2Fpublic%2Fimages%2F9f62472b-43e0-42ab-87f7-f45934d53650\_1296x1192.png)

_Prettified JSON blog highlighting the txt (or comment body) field_

## Wrap up

Once you’ve got the hang of searches for all of an author’s post you can experiment with other advanced queries over any of the other fields in any of our data sources such as language, location, links, etc. As always, let us know via email at info@openmeasures.io!

## Content Fields

When `querytype` is set to `content` or `boolean_content` the API will search through the default content field for each site. A list of the content field per site can be found in our open-source API code base which is embedded below.

{% @gitlab-files/gitlab-code-block url="https://gitlab.com/openmeasures/backends/openmeasures-api/-/blob/main/smat_be/es_config.py" %}
