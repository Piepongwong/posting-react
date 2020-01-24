
# Sending Post Data with React and CORS

Besides from reading from another data source, we can also use Axios to send data. You already know the most you need to know. You know how to control forms with React and you know how to use Axios. What's left is sending data from your form with Axios! This lesson is also explaining what CORS is and how to deal with it. It's time to have that talk. However, CORS is not something that is specific to React. We're going to use <a href="https://ih-beers-api.herokuapp.com/">this api</a> for the examples.

## CORS
There are a couple of pitfalls when sending data with React, especially in development. Most of them are caused by Cross Origin Resource Sharing (CORS) errors. Those errors are thrown by a security feature that tries to prevent an attack called<a href="https://owasp.org/www-project-top-ten/OWASP_Top_Ten_2017/Top_10-2017_A7-Cross-Site_Scripting_(XSS)"> cross site scripting</a>. To prevent XSS browsers and servers don't trust different origins by default. An origin consists out of a **transport layer (http or https), domain and port: `transportlayer://domain:port`. When any of those parts different, the origins are considered to be different!** In a SPA architecture it's common to host the client and server on different domains. If that's the case with your app, your React client and your Express backend do not trust each other by default. However, you could decide to host your client and backend on the same origin.

In development you're probably using the development server that create react app has set you up with. That development server is running on a different port than your backend. Therefore, in development you're probably hosting your app on different origins. 

Another thing to keep in mind, is that you can and have to configure the CORS setting on both the client and the server. The server needs to trust the client that is requesting information and the client needs to trust the server it's sending data to. Sending data tightens the security settings of CORS on the client stronger. Whether you're sending data or not and in which format (x-www-form-url-encoded or json) influences CORS errors. 

Alright, so how to deal with CORS?

## Method 1: Do not trigger CORS
If you send data in the "x-www-form-urlencoded" format, the CORS settings are less tight and you can send it normally. However, you need to parse the data you have in your state from a javascript object into the x-www-form-urlencoded format. You do that with the <a href="https://www.npmjs.com/package/qs"> qs library </a>. Querystring are in the  x-www-form-urlencoded format. In the qs docs you'll find examples with require, but it works just as fine with import:

```
    import qs from "qs";
    ...

    ...
    handleAddBeerSubmit(){
        axios({
            method: "POST",
            url: "https://ih-beers-api.herokuapp.com/beers/new",
            data: qs.stringify(this.state.beer),
            headers: {
                "content-type": "application/x-www-form-urlencoded"
            }
        })
        .then((response)=> {
            // you could redirect to the detail page of the beer you just created
            // assuming you're using react router:
            this.props.history.push(`/beer/detail/${response.data.id}`)
        })
        .catch((error)=> {
            console.log(error.response.data)
            // you could update the state to show the error to the user
        })
    }
    ...

```

When you're creating your own backend for the final project, you still need to enable cors on the server. You can use the <a href="https://www.npmjs.com/package/cors" />cors library </a> for that.

## Method 2: Allow CORS
If you want to send data from the client to the backend, you have to consider the data format. If you really want to send it in JSON, you have to pass the withCredentials option to axios: 

```
    ...
    handleAddBeerSubmit(){
        axios({
            method: "POST",
            url: "https://ih-beers-api.herokuapp.com/beers/new",
            data: this.state.beer, //different
            withCredentials: true,
            headers: {
                "content-type": "application/json" //different
            }
        }
        .then((response)=> {
            // you could redirect to the detail page of the beer you just created
            // assuming you're using react router:
            this.props.history.push(`/beer/detail/${response.data.id}`)
        })
        .catch((error)=> {
            console.log(error.response.data)
            // you could update the state to show the error to the user
        })
    }
    ...
```

When you're creating your own backend for the final project, you still need to enable cors on the server. You can use the <a href="https://www.npmjs.com/package/cors" />cors library </a> for that.

## Method 3 Circumvent CORS all together
You can also let the development server use a proxy. That way, your browser and server think everything is hosted on the same domain and CORS will not even be an issue. You have to put the following in your package.json file: 
```
{
    ...
    "proxy": "http://localhost:portYouAreRunningYourBackendOn",
    ...
}
```
This works the best if all your Axios requests are reading the baseUrl from your .env file. For further reference look up the cra docs about <a href="https://create-react-app.dev/docs/proxying-api-requests-in-development">proxying</a> and <a href="https://create-react-app.dev/docs/adding-custom-environment-variables"> environment variables</a>. To finish it all off, <a href="https://create-react-app.dev/docs/using-https-in-development/">run your dev-server over https</a>: `HTTPS=true npm start`. <small>(for bash)</small>.

So why is this method all the way at the bottom? Well, nowadays most of the applications are often deployed as small units called services. The term for that is <a href="https://martinfowler.com/articles/microservices.html"> microservice architecture </a>. So normally your back-end would run on another domain than your front-end, your database or your file server (like Cloudinary). Besides, you're going to stumble upon CORS errors sooner or later anyways. ;) It's usefull to know how they are caused and how to deal with them.

## **Questions**
* What is an origin?
* Why would you use the qs library. What does it do?
* Why would you use the cors library. What does it do?
* If you run your react dev server with `HTTPS=true npm start` and you run your local backend normally, would you get a CORS error? Why would you/wouldn't you?
* If you get a CORS error, why wouldn't that show up in the network tab of your dev tools? Where does it show instead?
* Take a look at the following .env.development variable for your react app: 
    ```
        REACT_APP_API=localhost:3000
    ```
    Why would is this likely to cause a CORS error? What part of the origin is missing?
* Where do you change the CORS security settings? On the client side, server side or both?

### Further readings:

* <a href="https://martinfowler.com/articles/microservices.html"> Microservice architecture </a> (Martin Fowler is a leading figure in software architecture)
* <a href="https://owasp.org/www-project-top-ten/OWASP_Top_Ten_2017/Top_10-2017_A7-Cross-Site_Scripting_(XSS)"> Cross Site Scripting</a>. This is the security issue CORS is trying to protect you from. 
* <a href="https://owasp.org/www-project-top-ten/">Top 10 Web Application Security Risks</a>. A good place to get started with web security in general.  
* <a href="https://developer.mozilla.org/nl/docs/Web/HTTP/CORS">CORS </a>. This is bit of a dry read. Advice: read the first couple of paragraphs and use the rest to trouble shoot your CORS errors if you encounter them.