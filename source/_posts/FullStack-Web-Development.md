---
title: Full-Stack Web Development
subtitle: Build an online ordering system from scratch
author:
    - I am an author
date: \today{}
---

### Project Background
When I visited my friend in North Carolina. My friend told me that they want to build a build an online ordering system for restaurants. We usually order our lunch or dinner on our mobile phones when we are in our offices in China, and pay the bills online. The restaurant will deliver the food to me in 10 minutes after I place the order. It is very convenience. When I moved to the US, I found that the delivering price is very expensive, so few people use their phones to order food, and in restaurants people use credit card to pay their checks. I doubt this bussiness model will be successful in this country, because it is hard to change the habits of users. But I think maybe it is an opportunity, so I joined the startup company. 

### Who am I
When I was in China, I worked for an online grocery company invested by Walmart. We have 20 million active users in China. When people place orders on our web site or on our app, we can deliver their goods in four hours. In the company I lead a front-end team to develop and maintain our applications on `iOS` and `Android` plaform. In our native applications we use lot of html pages embedded in. If we meet some bugs or some features changed, we can fix or update our front-end quickly, and we needn't to submit our application to Appstore. From that time, we choiced `Vuejs` as our Front-End framework.

### Tech Stack
My job is writing a prototype of the online ordering system. First of all, I choice Firebase because it take a lot of jobs which we must build by ourselves. Firebase gives us an realtime database and an authenticate solution. This let us focus on the ordering system itself. 
The second one is I select `Nodejs` and `Koa2` as our back end. If you familiar with Javascipt, you can write the application on both sides, from the back end to the frond end. In fact I am the only programmer in our small company. With the async function support, we can write `Await/Async` on Back End, this makes our asynchronous code be easy to handle. 
`Vue` is a progressive framework for building user interfaces. It is light and flexible. But we still need a UI framework to boost our developing. Comparing with seveval UI framewoks in `Vue` community, I think `Vuetify` is the best choise. Let's hot the road.
[picture_one]()

### System Diagram
[picture_two]()

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




