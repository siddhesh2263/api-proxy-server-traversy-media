# API Proxy Server using Node.js

## Introduction

Modern web applications often call third-party APIs directly from the client side. This exposes sensitive information such as API keys in the browser, making them easy to extract and misuse. Exposed keys can lead to unauthorized access, quota exhaustion, and security risks.

To address this issue, a backend proxy server is built using Node.js and Express. Instead of the client communicating directly with the external service, requests are routed through the server. The server securely injects the API key, applies usage controls, and returns only the required data to the client. This approach improves security, control, and performance.

<br>

## Project Overview

This project implements a lightweight API proxy server that sits between the client application and a public weather API.

### Core Workflow

1. The client sends a request to the Express server (e.g., /api?q=Boston).
2. The server reads the API key from environment variables stored in a `.env` file.
3. The server forwards the request to the external API using an HTTP client.
4. The external API responds with data.
5. The proxy server returns the response to the client without exposing the API key.

<br>

## Key Features Implemented

### API Key Protection
Sensitive credentials are stored securely on the server and never exposed to the browser.

### Request Forwarding
Query parameters from the client are dynamically passed to the external API.

### Rate Limiting
Controls how many requests a client can make within a time window to prevent abuse.

### Caching
Frequently requested data is temporarily stored to reduce redundant external calls and improve response time.

### Environment-Based Configuration
All secrets and runtime values are managed using environment variables for safer deployment.

### Static Frontend Hosting
The Express server also serves the frontend, allowing both client and proxy to run from the same origin.

<br>

## Development

The below steps are performed to develop this system.

Create the `public` folder. Modify the query to use your API key.

<p align="center">
  <img 
    src="https://github.com/siddhesh2263/api-proxy-server-traversy-media/blob/main/assets/001_api_key_visible.png?raw=true"
    alt="API Key Visible"
    width="700"
  />
</p>

<br>

Run command:
```
npm init -y
```
This will create the package.json file.

Dependencies used:
```
npm i express dotenv cors needle
```

`needle` is a lighweight HTTP client. The front end will make a request to our server, and then we will use needle to make a request using the public API to OpenWeather.

Install another dependency:
```
npm i -D nodemon
```
`-D` is used to signify that it is installed as a dev dependency. `nodemon` will allow us to make changes without having to restart the server everytime.

Make changes in the scripts part in `package.json`, to add start and dev commands.
```
"scripts": {
    "start": "node index",
    "dev": "nodemon index"
  },
```
The index file will the main entry point, called `index.js`.

Create a file `index.js` in root.

The below line means it will first check if PORT is present in the environment variables, otherwise it will use `PORT 5000`:
```
const PORT = process.env.PORT || 5000
```

Run command to run server:
```
npm run dev
```

Now let's take care of storing the API keys. We installed the dotenv package. Import the `dotenv` package, and then create a `.env` file in root.
In this file, we created environment variables.

Now, create a route, and test if response is received.

<p align="center">
  <img 
    src="https://github.com/siddhesh2263/api-proxy-server-traversy-media/blob/main/assets/002_no_param_success.png?raw=true"
    alt="No Param Success"
    width="700"
  />
</p>

<br>

Then, create a routes folder in root. Create a `index.js` file, export the module as `router`, and move the route information from root `index.js` to this `index.js` under routes folder.

`needle` returns a promise, so we need to use `async`.

We need to use `process.env` to get the environment variables.

Also, when environment variables are added, the server needs to be restarted.

We will use URL `search params` to include the API key name and value.

Brief on below code:
```
const API_BASE_URL = process.env.API_BASE_URL
const API_KEY_NAME = process.env.API_KEY_NAME
const API_KEY_VALUE = process.env.API_KEY_VALUE

router.get('/', async (req, res) => {

    try {
        const params = new URLSearchParams({
            [API_KEY_NAME]: API_KEY_VALUE
        })
        const apiRes = await needle('get', `${API_BASE_URL}?${params}`)
        const data = apiRes.body
        res.status(200).json(data)   
    } catch (error) {
        res.status(500).json({error})
    }
})
```
We have the API key and value in the `.env` file, but to use it, we will use search params. Above is the strucutre how the key name and value are set, and then used to make the request.

I was getting `ECONNREFUSED` due to the following:
```
API_BASE_URL = "https:/api.openweathermap.org/data/2.5/weather"
```
There should be `//` instead of `/` after https.

We get the following response:
```
{
    "cod": "400",
    "message": "Nothing to geocode"
}
```
This is because we need to pass city name, or the required parameters to get the data for specific region. So, whatever is passed in the server, needs to be forwarded to the public URL.

Now in Postman, when we don't pass anything, the `url.parse(req.url, true).query` will return an empty object. However, if we pass let's say `?q=Boston`, then in the console log we can see the param that is passed.

<p align="center">
  <img 
    src="https://github.com/siddhesh2263/api-proxy-server-traversy-media/blob/main/assets/003_object_param_city.png?raw=true"
    alt="Object Param City"
    width="700"
  />
</p>

<br>

In the params, it is passed as a spread operator. So, let's look at the below code:
```
const params = new URLSearchParams({
            [API_KEY_NAME]: API_KEY_VALUE,
            ...url.parse(req.url, true).query,
        })
```
What this essentially means that whatever is passed as a query parameter in the URL, it will be stored in params, which then will be used by the needle lightweight client to make the public API call.

<p align="center">
  <img 
    src="https://github.com/siddhesh2263/api-proxy-server-traversy-media/blob/main/assets/004_city_api_call.png?raw=true"
    alt="City API Call"
    width="700"
  />
</p>

<br>

To check the URL that was sent to the public server, use the below code:
```
if(process.env.NODE_ENV != 'production') {
            console.log(`REQUEST: ${API_BASE_URL}?${params}`)
        }
```

<br>

## Rate Limiting

Run the below command:
```
npm i express-rate-limit apicache
```

<p align="center">
  <img 
    src="https://github.com/siddhesh2263/api-proxy-server-traversy-media/blob/main/assets/005_too_many_requests_error.png?raw=true"
    alt="Too Many Requests Error"
    width="700"
  />
</p>

<br>

In the headers in Postman we can see the rate limit information - what is the limit, and how many rate limits are remaining.

<p align="center">
  <img 
    src="https://github.com/siddhesh2263/api-proxy-server-traversy-media/blob/main/assets/006_header_rate_limiting.png?raw=true"
    alt="Rate Limiting"
    width="700"
  />
</p>

<br>

Because we are using our Express server as a proxy server, we need to use the following:
```
app.set('trust proxy', 1)
```

<br>

## Caching

In Postman under Headers, under `cache-control`, the `max-age` keeps decreasing. Once it reaches 0, it will make the actual request, and store it for 2 more minutes.

It is added as a second argument:
```
// Initialize cache
let cache = apicache.middleware

router.get('/', cache('2 minutes'), async (req, res) => {
    ...
```

The minutes set depends on the API usage. For instance, if it's something that changes too often, it can be set lower.

<p align="center">
  <img 
    src="https://github.com/siddhesh2263/api-proxy-server-traversy-media/blob/main/assets/007_cache_control_header.png?raw=true"
    alt="Cache Control Header"
    width="700"
  />
</p>

<br>

Now we need to be able to use this backend code with out client side application, which we have in the public folder. The below code should load the `index.html` at `localhost:5000`:
```
app.use(express.static('public'))
```

In the `public/index.html` file, we will update the url to following:
```
const url = `/api?q=${city}`
```

Since it is hosted on the same server, we can use this approach.

Now, if we see in console, the API key is not visible. It is stored on the server side, and not on the client.

<p align="center">
  <img 
    src="https://github.com/siddhesh2263/api-proxy-server-traversy-media/blob/main/assets/008_api_hidden.png?raw=true"
    alt="API Key Hidden"
    width="700"
  />
</p>

<br>

## Conclusion

The final system acts as a secure middleware layer. It hides credentials, regulates traffic, improves efficiency, and provides a production-friendly way to consume third-party APIs.