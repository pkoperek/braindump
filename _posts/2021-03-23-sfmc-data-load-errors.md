---
layout: post
title: Sales Force Marketing Cloud REST API upload errors
comments: true
---

While trying to insert data into Sales Force Marketing Cloud I frequently encounter one of the following errors:

```
{"message":"Primary key 'subscriberkey' does not exist.","errorcode":10000,"documentation":""}
```

... or ...

```
{"message":"Parameter {guid} is invalid.","errorcode":10001,"documentation":""}
```

... or ...

```
{"message":"Unable to save rows for data extension ID <ID>","errorcode":10006,"documentation":""}
```

Unfortunately those messages are pretty cryptic and its quite hard to identify what exactly is the root cause. Fortunately there are couple things which you can try out:

* Verify that the url has a correct format (mind the `key:` portion:
  * `https://<YOUR_SUBDOMAIN>.rest.marketingcloudapis.com/hub/v1/dataevents/key:<DATA_EXTENSION_NAME>/rowset`
* Check if you there are "null" values in your data. Even if you allowed a specific column to be null in the DE definition, you still have to send empty strings to make the API call work.
* Check if you have set the external key of Data Extension correctly. I found that setting it from Audience Builder does not usually work, but in Email Studio everything is ok.
