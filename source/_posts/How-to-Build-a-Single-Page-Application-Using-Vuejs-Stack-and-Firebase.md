---
title: How to Build a Single Page Application Using Vuejs Stack and Firebase
date: 2019-02-21 08:07:39
tags: [Vuejs, Vuetify, Vuex, Vue Router, Nodejs, Firebase, SPA]
categories: 
  - Software Architecture
---

### Project Background
When I visited my friend in North Carolina. My friend told me that they want to build a build an online ordering system for restaurants. We usually order our lunch or dinner on our mobile phones when we are in our offices in China, and pay the bills online. The restaurant will deliver the food to me in 10 minutes after I place the order. It is very convenience. When I moved to the US, I found that the delivering price is very expensive, so few people use their phones to order food, and in restaurants people use credit card to pay their checks. I doubt this bussiness model will be successful in this country, because it is hard to change the habits of users. But I think maybe it is an opportunity, so I joined the startup company. 

If you have read other articles about Vuejs, you might see them talking about how to build ...

This tutorial is presented as a series of articles that will show you how to build a realistic(full functional) online ordering system using `Vuejs` stack and `Firebase`. It is broken down into a six-part series:

This series is suitable for everyone regardless of your skill level. I only assume that you have a knowledge of ES6.


### Who am I
When I was in China, I worked for an online grocery company invested by Walmart. We have 20 million active users in China. When people place orders on our web site or on our app, we can deliver their goods in four hours. In the company I lead a front-end team to develop and maintain our applications on `iOS` and `Android` plaform. In our native applications we use lot of html pages embedded in. If we meet some bugs or some features changed, we can fix or update our front-end quickly, and we needn't to submit our application to Appstore. From that time, we choiced `Vuejs` as our Front-End framework.

### Tech Stack
My job is writing a prototype of the online ordering system. First of all, I choice Firebase because it take a lot of jobs which we must build by ourselves. Firebase gives us an realtime database and an authenticate solution. This let us focus on the ordering system itself. 
The second one is I select `Nodejs` and `Koa2` as our back end. If you familiar with Javascipt, you can write the application on both sides, from the back end to the frond end. In fact I am the only programmer in our small company. With the async function support, we can write `Await/Async` on Back End, this makes our asynchronous code be easy to handle. 
`Vue` is a progressive framework for building user interfaces. It is light and flexible. But we still need a UI framework to boost our developing. Comparing with seveval UI framewoks in `Vue` community, I think `Vuetify` is the best choise. Let's hit the road.
[picture_one]()

### System Diagram
[picture_two]()

### Creating a SPA project
I used the Vue CLI to create the scaffolding for our Vue application.

### Integrate Firebase Authenticate in Vuejs
Firebase provides seveval ways to authenticate users to our app. It includes using passwords, phone numbers, popular federated identity providers like Google, Facebook and Twitter, and more. 
Creating a password-based acccount is pretty easy. 
``` javascript
firebase.auth().createUserWithEmailAndPassword(email, password).catch(function(error) {
  // Handle Errors here.
  var errorCode = error.code;
  var errorMessage = error.message;
  // ...
})
```
then users can login with their passwords, like this
``` javascript
firebase.auth().signInWithEmailAndPassword(email, password).catch(function(error) {
  // Handle Errors here.
  var errorCode = error.code;
  var errorMessage = error.message;
  // ...
})
```

Authenticating with a phone number in Firebase is more complicated then using a password. It will take me mant steps to achieve this process.
1. First we need to set up a reCAPTCHA verifier to prevent abuse. In a login view at the `mounted` hook of vue life cycle, we bind a verifier to the login button, and save this verifier into a global variable.
``` javascript
  mounted() {
    const that = this;

    window.recaptchaVerifier = new firebase.auth.RecaptchaVerifier(
      "sign-in-button",
      {
        size: "invisible",
        callback: response => {
          that.submitPhoneNumber();
        },
        "expired-callback": function() {
          window.recaptchaVerifier.render().then(function(widgetId) {
            grecaptcha.reset(widgetId);
          })
        }
      }
    )

    window.recaptchaVerifier.render().then(function(widgetId) {
      window.recaptchaWidgetId = widgetId
    })
  },
```
When the user clicks on the login button, the verifier will popup to verify it is a real user not a machine.

2. After the user passing the verification process, we will send the phone number to Firebase, and Firebase will send a comfirm code back.
``` javascript 
  submitPhoneNumber() {
   const that = this;
   const appVerifier = window.recaptchaVerifier;
   auth
     .signInWithPhoneNumber("+1" + that.phoneNumber, appVerifier)
     .then(function(confirmationResult) {
       // SMS sent. Prompt user to type the code from the message, then sign the
       // user in with confirmationResult.confirm(code).
       that.SMSSent()
       window.confirmationResult = confirmationResult;
     })
     .catch(function(error) {
       // Error; SMS not sent
       // ...
       console.log(error);
       window.recaptchaVerifier.render().then(function(widgetId) {
         grecaptcha.reset(widgetId);
       })
     })
  },
```
3. In the last step, we send this confirm code to Firebase to finish the login process.
``` javascript
  submitCode(code) {
   const that = this
   console.log(code)
   window.confirmationResult
     .confirm(code)
     .then(function(result) {
       // User signed in successfully.
       var user = result.user
       console.log(`User signed in successfully, userID = ${user.uid}`)
       that.$store.commit(ADD_PHONE_NUMBER, "+1" + that.phoneNumber)

       that.$router.replace("/")
       // ...
     })
     .catch(function(error) {
       // User couldn't sign in (bad verification code?)
       // ...
       console.log(error)
     })
  }
``` 

#### Verify ID token
Our Firebase client app communicates with our custom backend server, so we need to identify the currently signed-in user on that server.

On the client side, we can get a corresponding ID token when a user or device successfully signs in. So we embed this ID token into every `Get/Post` request headers to send this ID token to our custom backend server.

``` javascript
export function get(url, ssl = true) {
  return async function (params = {}) {
    const currentUser = auth.currentUser
    var authHeader = {}
    if (currentUser) {
      const idToken = await currentUser.getIdToken()
      authHeader = { Authorization: `Bearer ${idToken}` }
    }
    const protocol = ssl ? 'https://' : 'http://'
    const port = ssl ? easyeat.ports : easyeat.port
    const baseUrl = protocol + window.location.hostname + ':' + port + '/api/'
    return axios.get(baseUrl + url, {
      params: params,
      headers: authHeader
    }).then((res) => {
      const { errno, data } = res.data
      if (errno === ERR_OK) {
        return data
      }
    }).catch((e) => {
      console.log("AXIOS ERROR: ", err)
    })
  }
}
```

On the server side, if some of the APIs need to check the ID token, we use the Firebase Admin SDK to verify the ID.
``` javascript
exports.authenticate = async (ctx) => {
  var idToken
  if (ctx.request.header.authorization) {
    idToken = ctx.request.header.authorization.split(' ')[1]
  }
  if (!idToken) {
    ctx.throw(401, 'authentication failed')
  }
  try {
    var decodedToken = await fireStoreOrderService.verifyIdToken(idToken)
    return decodedToken.uid
  } catch(err) {
    ctx.throw(401, 'authentication failed')
  }
}
```

##### 
Because some of these APIs on the server side need to check the ID token, on the client side we add an observer to monitor the status of the user. If the user doesn't sign in, we will not call these APIs
``` javascript
auth.onAuthStateChanged(function (user) {
  if (user) {
    // User is signed in.
    var displayName = user.displayName
    var email = user.email
    var emailVerified = user.emailVerified
    var photoURL = user.photoURL
    var isAnonymous = user.isAnonymous
    var uid = user.uid
    var providerData = user.providerData

    console.log(uid + ' was signed in')
    EventBus.$emit('signed-in', uid)
  } else {
    // User is signed out.
    console.log('signed out')
  }
})
```

In this code, I add an event bus to inform other `Vue` components if the status of the user has changed. 

### Work with Stripe
Our customers can pay the bills on their phones. So we need to integrate a third party solution to support the payment. We choice Stripe. 

After the user signed in, the user can bind his credit card with his account. 

Stripe provides a pre-built UI component called `Stripe Elements` to help us build our checkout flow without touching our customers' sensitive information. We can embed this `Stripe Elements` into `Vue` template

``` Vue
 <v-content>
   <v-card ref="payment-form">
     <v-card-title primary-title>
       <span class="headline">Credit or debit card</span>
     </v-card-title>
     <div ref="card-element" id="card-element">
       <!-- A Stripe Element will be inserted here. -->
     </div>
     <v-card-actions>
       <v-btn flat color="orange" @click="createToken">Submit Payment</v-btn>
     </v-card-actions>
   </v-card>
 </v-content>
```

``` javascript
var stripe = Stripe('pk_test_GrY8gvfkfkXKks0woG0')
var elements = stripe.elements()

var style = {
  base: {
    color: '#32325d',
    lineHeight: '18px',
    fontFamily: '"Helvetica Neue", Helvetica, sans-serif',
    fontSmoothing: 'antialiased',
    fontSize: '16px',
    '::placeholder': {
      color: '#aab7c4'
    }
  },
  invalid: {
    color: '#fa755a',
    iconColor: '#fa755a'
  }
}

var card = elements.create('card', {style: style})

export default {
   ...
   mounted() {
     card.mount('#card-element')
   },
}
```

And we can also customize its style.

When the user fill the credit card infomation, and click the submit, our client app will call Stripe SDK to send this card infomation to Stripe backend, and then the Stripe will a token to us. The user can save the token for the futune payment. In this process we can't touch the sensitive information of the user, we can lony get the payment token, and we encrypt this token and save in our backend server if the user allowed. 

``` javascript
createToken() {
   const that = this

   stripe.createToken(card).then(function(result) {
     if (result.error) {
       // Inform the user if there was an error.
       console.log(result.error.message)
     } else {
       // Send the token to your server.
       that.stripeTokenHandler(result.token)
     }
   })
 },
 // Submit the form with the token ID.
 stripeTokenHandler(token) {
   const user = auth.currentUser
   console.log(`userID: ${user.uid}, token: ${token.id}, phoneNumber: ${this.$store.state.phoneNumber}, email: ${this.$store.state.email}`)
   const that = this

   stripe.createSource(card).then(result => {
     if (result.error) {
       // Inform the user if there was an error.
       console.log(result.error.message)
     } else {
       // Send the token to your server.
       saveStripeCustomerID( { 
         userID: user.uid,
         stripeToken: token.id,
         phoneNumber: this.$store.state.phoneNumber,
         email: this.$store.state.email,
         cardInfo: {
           brand: result.source.card.brand,
           country: result.source.card.country,
           exp_month: result.source.card.exp_month,
           exp_year: result.source.card.exp_year,
           last4: result.source.card.last4
         }
       }).then(res => {
         that.$store.commit(ADD_CREDIT, true)
         that.$router.replace('/')
       })  
     }        
   })
 }
},
```

This tutorial is presented as a series of articles that will take you from installing Vue for the first time to creating a full functional online ordering system using full stack Javascript.

It is broken down into seveval parts, and is suitable for everyone regardless of your skill level, I only assume that you have a knowledge of Javascript.

### Flex box
### class or prop
### cache load 
### Router Routes and route
### 

### System Diagram for Building a Web Application Using Full Stack JavaScript(MEVN)
![System Diagram](http://pnd8z1q22.bkt.gdipper.com/ordering-system-diagram.png)

### From MEVN to FKVN
MEVN(MongoDB, Express.js, AngularJS, Node.js) -> MEVN(MongoDB, Express.js, VueJS, Node.js) -> MKVN(MongoDB, Koa.js, VueJS, Node.js) -> FKVN(Firebase, Koa.js, VueJS, Node.js)
1. Choosing Node.js is unanimous, because Node.js use event-driven, non-blocking IO model that make it lightweight and fast as compared to other commonly used back end technologies. And our back end is not computation-oriented.
2. Due to its gentle learning curve, VueJS is growing so fast. VueJS is also a lightweight(20KB) and high performance front end framework(a progressive framework). One of the core features of Vue.js is its reactive data binding system, this make our programming easier.
[Exploring Vue.js: Reactive Two-Way Data Binding](https://medium.com/js-dojo/exploring-vue-js-reactive-two-way-data-binding-da533d0c4554)
3. We replace `Express.js` with `Koa`, because we can use async/await in `Koa2` with the `Node.js 7.6`. `Async/await` allows developers to write asynchronous code in a syntax that looks synchronous.
4. Instead of setting up MongoDB or other databases by ourselves, we build our application based on Firebase. Firebase abstracts away most of our complex server-side feature like user authentication, data persistence, file storage and micro services. So we can focus on building an application in the front end and let us finish our prototype as quickly as possible.

[The Good and the Bad of JavaScript Full Stack Development](https://www.altexsoft.com/blog/engineering/the-good-and-the-bad-of-javascript-full-stack-development/)
[Build full stack web apps with MEVN Stack [Part 1/2]](https://medium.com/@anaida07/mevn-stack-application-part-1-3a27b61dcae0)

### Vuetify
We still need an UI framework to style our font end application. There are a lot of candidates: Element, Quasar, Vuetify, and so on. Vuetify supports Google Material Design, and this is the main factor why we choose it. And Vuetify also gives us a lot of components. Likewise Vuetify has a 12 point grid system which makes our application work on many different screen size.

[Vue.js UI Component Libraries you Should Know in 2019](https://blog.bitsrc.io/11-vue-js-component-libraries-you-should-know-in-2018-3d35ad0ae37f)


### Scaffold a full-stack project
#### Using Vue-cli or using Webpack to setup a project from scratch?
##### step 1: create a root folder for this demo project, and a sub-folder for a project using Vue-cli 2.0
##### step 2: create a Koa server
1. Host the static bundle on the Koa server
2. Host the static bundle on the webpack-dev-server, using different servers in the development environment, but need add CORS middleware in the Koa server.

➜  TodaySpecial vue init webpack vue-cli-2

? Project name earth
? Project description a project built from scratch using Vue-cli 2 and Webpack
? Author Qubo Qin <qubo.qin.2018@gmail.com>
? Vue build (Use arrow keys)
❯ Runtime + Compiler: recommended for most users
  Runtime-only: about 6KB lighter min+gzip, but templates (or any Vue-specific HTML) are ONLY allowed in .vue files -
render functions are required elsewhere

➜  TodaySpecial vue init webpack vue-cli-2

? Project name earth
? Project description a project built from scratch using Vue-cli 2 and Webpack
? Author Qubo Qin <qubo.qin.2018@gmail.com>
? Vue build standalone
? Install vue-router? Yes
? Use ESLint to lint your code? Yes
? Pick an ESLint preset (Use arrow keys)
❯ Standard (https://github.com/standard/standard)
  Airbnb (https://github.com/airbnb/javascript)
  none (configure it yourself)

➜  TodaySpecial vue init webpack vue-cli-2

? Project name earth
? Project description a project built from scratch using Vue-cli 2 and Webpack
? Author Qubo Qin <qubo.qin.2018@gmail.com>
? Vue build standalone
? Install vue-router? Yes
? Use ESLint to lint your code? Yes
? Pick an ESLint preset Standard
? Set up unit tests Yes
? Pick a test runner (Use arrow keys)
❯ Jest
  Karma and Mocha
  none (configure it yourself)  

➜  TodaySpecial vue init webpack vue-cli-2

? Project name earth
? Project description a project built from scratch using Vue-cli 2 and Webpack
? Author Qubo Qin <qubo.qin.2018@gmail.com>
? Vue build standalone
? Install vue-router? Yes
? Use ESLint to lint your code? Yes
? Pick an ESLint preset Standard
? Set up unit tests Yes
? Pick a test runner jest
? Setup e2e tests with Nightwatch? Yes
? Should we run `npm install` for you after the project has been created? (recommended) (Use arrow keys)
❯ Yes, use NPM
  Yes, use Yarn
  No, I will handle that myself

You can put a .gitignore in any directory, and it will only affect that directory and its subdirectories.  

#### How to hosting the bundle of the front-end and how to use CDN to accelerate the loading speed?
#### How to deploy the back-end server on Heroku, AWS and Google Cloud?


Vue CLI v3.1.3
┌───────────────────────────┐
│  Update available: 3.7.0  │
└───────────────────────────┘
? Please pick a preset: Manually select features
? Check the features needed for your project: 
 ◉ Babel
 ◯ TypeScript
 ◯ Progressive Web App (PWA) Support
 ◉ Router
 ◉ Vuex
 ◉ CSS Pre-processors
 ◉ Linter / Formatter
 ◉ Unit Testing
❯◉ E2E Testing

1. Understanding the difference between ES6 and Node.jS(CommonJS) module
2. 