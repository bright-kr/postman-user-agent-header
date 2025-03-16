# Setting and Changing the Postman User-Agent Header

[![Promo](https://github.com/luminati-io/LinkedIn-Scraper/raw/main/Proxies%20and%20scrapers%20GitHub%20bonus%20banner.png)](https://brightdata.com/) 

This guide explains how to set, change, and rotate the User-Agent header in Postman to avoid anti-bot detection and improve your HTTP requests:

- [What Is the Postman Default User Agent?](#what-is-the-postman-default-user-agent)
- [Change the Postman User Agent](#change-the-postman-user-agent)
  - [Set the User Agent on a Single Request](#set-the-user-agent-on-a-single-request)
  - [Set the User Agent on an Entire Collection](#set-the-user-agent-on-an-entire-collection)
  - [Unset the User Agent](#unset-the-user-agent)
- [Implement User Agent Rotation in Postman](#implement-user-agent-rotation-in-postman)
  - [Retrieve a List of User Agents](#retrieve-a-list-of-user-agents)
  - [Randomly Pick a User Agent](#randomly-pick-a-user-agent)
  - [Define the User-Agent Header](#define-the-user-agent-header)
  - [Put It All Together](#put-it-all-together)

## Why You Need to Set a Custom User Agent

The [`User-Agent`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/User-Agent) header identifies the client making an HTTP request. It typically includes details about the client’s machine and the application used for the request. Web browsers, [HTTP clients](https://brightdata.com/blog/web-data/best-python-http-clients), and other software set this header automatically.

Here’s an example of a `User-Agent` string used by Chrome when requesting a web page:

```
Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/127.0.0.0 Safari/537.36
```

A browser's `User-Agent` header consists of several key components:

- **`Mozilla/5.0`** – Originally signaled Mozilla compatibility; now included for broader support.  
- **`Macintosh; Intel Mac OS X 10_15_7`** – Indicates the operating system (`Mac OS X 10.15.7`) and platform (Intel Mac).  
- **`AppleWebKit/537.36`** – Specifies the rendering engine used by Chrome.  
- **`(KHTML, like Gecko)`** – Ensures compatibility with KHTML and Gecko layout engines.  
- **`Chrome/127.0.0.0`** – Represents the browser name and version.  
- **`Safari/537.36`** – Suggests compatibility with Safari.  

Servers use the `User-Agent` header to identify whether a request comes from a browser or another source.  

A common mistake among web scraping bots is using default or non-browser `User-Agent` strings, which anti-bot protections easily detect and block.

## What Is the Postman Default User Agent?

[Postman](https://www.postman.com/) is a widely used desktop HTTP client. Like most HTTP clients, it automatically sets a default `User-Agent` header in each request.

You can observe this behavior by inspecting the auto-generated headers in Postman.

![observe this behavior by examining the auto-generated Postman headers](https://github.com/luminati-io/postman-user-agent-header/blob/main/images/observe-this-behavior-by-examining-the-auto-generated-Postman-headers-1024x396.png)

As you can see, the Postman default user agent follows this format:

```
PostmanRuntime/x.y.z
```

The `PostmanRuntime/x.y.z` string indicates a request made by Postman, where `x.y.z` represents the version number.

To confirm this, send a GET request to [`httpbin.io/user-agent`](https://httpbin.io/user-agent). This endpoint returns the `User-Agent` header of the incoming request, allowing you to verify the user agent used by any HTTP client.

![identify the user agent used](https://github.com/luminati-io/postman-user-agent-header/blob/main/images/identify-the-user-agent-used-1024x593.png)

Notice how the user agent returned by the API matches the one set by Postman by default. In detail, the Postman user agent is:

```
PostmanRuntime/7.41.0
```

Postman’s `User-Agent` differs from browser headers, making requests more likely to be blocked by anti-bot systems. These systems detect bot-like patterns, including unusual user agents. Changing the default `User-Agent` helps avoid detection.

## How to Change the Postman User Agent

### Set the User Agent on a Single Request

Postman allows you to change the user agent on a single HTTP request by manually specifying a `User-Agent` header.

> **Note**:
> 
> The Postman auto-generated headers cannot be directly modified.

Go to the “Headers” tab and add a new `User-Agent` header:

![adding a new user-agent header](https://github.com/luminati-io/postman-user-agent-header/blob/main/images/adding-a-new-user-agent-header-1024x333.gif)

Postman replaces its default `User-Agent` with your custom header. Since [HTTP headers are case-insensitive](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers), you can use `User-Agent`, `user-agent`, or any variation.

Verify the change by sending a GET request to `httpbin.io/user-agent`.

![executing a GET request](https://github.com/luminati-io/postman-user-agent-header/blob/main/images/executing-a-GET-request-1024x578.png)

### Set the User Agent on an Entire Collection

A [Postman collection](https://www.postman.com/collection/) is a group of API requests with shared configurations. Postman allows defining [custom scripts](https://learning.postman.com/docs/tests-and-scripts/write-scripts/intro-to-scripts/) that run before or after each request in a collection.

To apply a custom `User-Agent` to all requests, log in to your Postman account or [create one](https://www.postman.com/postman-account/).  

Assume you have an "HTTPBin" collection with organized HTTPBin endpoints:

![HTTPBin endpoints organized in folders](https://github.com/luminati-io/postman-user-agent-header/blob/main/images/HTTPBin-endpoints-organized-in-folders-1024x554.png)

Execute the request for the `/user-agent` endpoint, and you will get the default Postman user agent:

![the default Postman user agent](https://github.com/luminati-io/postman-user-agent-header/blob/main/images/the-default-Postman-user-agent-1024x555.png)

A [pre-request script](https://learning.postman.com/docs/tests-and-scripts/write-scripts/pre-request-scripts/) is a JavaScript function that runs before each request in a Postman collection. You can use it to set a custom `User-Agent` header.

To create a pre-request script:  
1. Open your collection.  
2. Navigate to the **Scripts** tab.  
3. Select the **Pre-request** option.  

![select the “Pre-request” option](https://github.com/luminati-io/postman-user-agent-header/blob/main/images/select-the-Pre-request-option-1024x557.gif)

In the editor, paste the following code:

```js
pm.request.headers.add({

key: "User-Agent",

value: "<your-user-agent>"

});
```

Replace the `<your-user-agent>` string with the value of the user agent you want to use, as below:

```js
pm.request.headers.add({

key: "User-Agent",

value: "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/127.0.0.0 Safari/537.36"

});
```

[`pm.request.headers.add()`](https://learning.postman.com/docs/tests-and-scripts/write-scripts/postman-sandbox-api-reference/) is a special function from the Postman API to add a specific header to the request.

Click the “Save” button to apply the changes.

Execute the request for the `/user-agent` endpoint again:

![executing the request again ](https://github.com/luminati-io/postman-user-agent-header/blob/main/images/executing-the-request-again-1024x559.png)

This time, the returned user agent will be the one set in the script and not the default Postman one.

### Unset the User Agent

The `User-Agent` auto-generated header is optional, and you can actually uncheck it. When unchecked, Postman will no longer send a `User-Agent` header.

Verify that by sending a request to the [`httpbin.io/headers`](https://httpbin.io/headers) endpoint, which returns all the headers of the incoming request:

![return of all the headers of the incoming request](https://github.com/luminati-io/postman-user-agent-header/blob/main/images/return-of-all-the-headers-of-the-incoming-request-1024x640.png)

Note that the headers object returned by the endpoint does not include the `User-Agent` key.

> **Note**:
>
> Unsetting the `User-Agent` header is not recommended, as nearly web requests typically include that header.

## Implement User Agent Rotation in Postman

Simply replacing Postman’s `User-Agent` with a browser string may not bypass anti-bot systems, especially when making repeated requests from the same IP.

To avoid detection, use _user agent rotation_, assigning a different `User-Agent` to each request. This reduces the likelihood of being flagged as a bot.

Let's learn how to implement user agent rotation in Postman.

### Retrieve a List of User Agents

Visit a site like [WhatIsMyBrowser.com](https://www.whatismybrowser.com/guides/the-latest-user-agent/) and populate a list of valid user agents:

```js
const userAgents = [

"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/127.0.0.0 Safari/537.36 Edg/127.0.2651.86",

"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/127.0.0.0 Safari/537.36 Edg/127.0.2651.86",

"Mozilla/5.0 (Linux; Android 10; HD1913) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/127.0.6533.103 Mobile Safari/537.36 EdgA/127.0.2651.82",

"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/127.0.0.0 Safari/537.36",

"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/127.0.0.0 Safari/537.36",

"Mozilla/5.0 (iPhone; CPU iPhone OS 17_6 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) CriOS/127.0.6533.107 Mobile/15E148 Safari/604.1",

"Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:129.0) Gecko/20100101 Firefox/129.0",

"Mozilla/5.0 (Macintosh; Intel Mac OS X 14.6; rv:129.0) Gecko/20100101 Firefox/129.0",

// other user agents...

];
```

> **Tip**:
> 
> The more real-world user agents this array contains, the better to increase the rotation chances.

### Randomly Pick a User Agent

Use the JavaScript [`Math`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Math) API to randomly select a user agent from the list:

```js
const userAgent = serAgents[Math.floor(Math.random() * userAgents.length)];
```

This is what happens in this line of code:

1.  [`Math.random()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Math/random) generates a random number between 0 and 1.
2.  The generated number is then multiplied by the length of the `userAgents` array.
3.  [`Math.floor()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Math/floor) rounds down the resulting number to the largest integer less than or equal to the source number. The resulting number corresponds to an index that goes from 0 to `userAgents.length - 1`.
4.  The index is used to access a random item from the array of user agents.
5.  The randomly selected user agent is assigned to a variable.

### Define the User-Agent Header

Use the `pm.request.headers.add()` to define a `User-Agent` header with the random `userAgent` value:

```js
pm.request.headers.add({

key: "User-Agent",

value: userAgent

});
```

The HTTP request of the collection will now have a rotating `User-Agent` header.

### Put It All Together

Here is the final Postman pre-request script for user agent rotation:

```js
// a list of valid user agents

const userAgents = [

"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/127.0.0.0 Safari/537.36 Edg/127.0.2651.86",

"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/127.0.0.0 Safari/537.36 Edg/127.0.2651.86",

"Mozilla/5.0 (Linux; Android 10; HD1913) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/127.0.6533.103 Mobile Safari/537.36 EdgA/127.0.2651.82",

"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/127.0.0.0 Safari/537.36",

"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/127.0.0.0 Safari/537.36",

"Mozilla/5.0 (iPhone; CPU iPhone OS 17_6 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) CriOS/127.0.6533.107 Mobile/15E148 Safari/604.1",

"Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:129.0) Gecko/20100101 Firefox/129.0",

"Mozilla/5.0 (Macintosh; Intel Mac OS X 14.6; rv:129.0) Gecko/20100101 Firefox/129.0",

// other user agents...

];

// randomly extract a user agent from the list

const userAgent = userAgents[Math.floor(Math.random() * userAgents.length)];

// set the random user agent header

pm.request.headers.add({

key: "User-Agent",

value: userAgent

});
```

Add the script to your collection and test it by sending requests to `httpbin.io/user-agent`. Run multiple requests to observe the rotating user agents in action.

![executing requests and seeing the rotation of user agents](https://github.com/luminati-io/postman-user-agent-header/blob/main/images/executing-requests-and-seeing-the-rotation-of-user-agents-1024x557.gif)

The returned user agent keeps changing.

## Conclusion

Overriding the `User-Agent` header and implementing user agent rotation helps elude basic anti-bot mechanisms. However, more advanced systems will still be able to block your requests. To prevent IP bans, you could [use a proxy in Postman](https://brightdata.com/integration/postman).

Sign up today to start your free trial.
