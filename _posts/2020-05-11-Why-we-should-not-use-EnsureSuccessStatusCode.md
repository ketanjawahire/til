---
layout: post
title: "Why we should not use EnsureSuccessStatusCode()?"
excerpt_separator: "<!--more-->"
tags: 
    - C#
    - code quality
---

_[11/05/2020 5:44 PM] Amber R: Hi Ketan, the issue with throwing a specific exception is that we have to parse the error response and throw an exception based on that_

_For eg. if call in GetCollections method fails, if we throw a custom exception when it's not success status code, isn't it the same as as using .EnsureSuccessStatusCode()? Just the message will be different since it's custom exception, but we still won't know why the call fail_

---

I got this message from one of my colleague. We were argueing on why using EnsureSuccessCode() is a bad idea. 

<!--more-->

So in case of HTTP request failure, EnsureSuccessStatusCode() throws an exception with message saying - _'Response does not indicate success status code'_ & there is no other information attached to it. All the information about why your HTTP call failed is lost. Your HTTP request can fail due to multiple reasons like auth issues, network issues, server side issues, DNS related issues but EnsureSuccessStatusCode() will not tell you what exactly caused the HTTP request failure.  <br>

Vs <br>

If we do custom exception, then we have control on the way HTTP Response is being handled. We can analyze the response received from HTTP call & decide what to do with it. 

So even if it looks like the only difference is in message but thats only thing thats important here & we should not loose it. 

Example : 
{% highlight csharp %}
//Case 1 : Use EnsureSuccessStatusCode
using (var payload = GetPayload(postData))
{
  // perform API call
  var response = await httpClient.PostAsync(url, payload).ConfigureAwait(false);
  
  // This like will throw exception if response status code is not 2xx. 
  //Effectively loosing all the information on why I got the error from HTTP request.
  response.EnsureSuccessStatusCode();
  
  //success case. return receive response.
  return await response.Content.ReadAsAsync().ConfigureAwait(false);
}

// Case 2 : Check status code
using (var payload = GetPayload(postData))
{
  // perform API call
  var response = await httpClient.PostAsync(url, payload).ConfigureAwait(false);

  //check if you have got 2xx response
  if (!response.IsSuccessStatusCode)
  {
    // if not 2xx then throw custom exception. 
    // I can parse response content inside my exception class & keep the important information as class property values.
    throw new MyCustomException(response.Content);
  }

  //success case. return receive response.
  return await response.Content.ReadAsAsync().ConfigureAwait(false);
}
{% endhighlight %}
