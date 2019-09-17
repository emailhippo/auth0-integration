---
title: API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - JavaScript

toc_footers:
  - <a href='https://register.emailhippo.com/signup'>Sign Up for a Developer Key</a>
  - <a href='https://help.emailhippo.com'>Support</a>
  - <a href='https://www.emailhippo.com/compliance/terms-of-service/'>Terms of service</a>
  - <a href='https://www.emailhippo.com/compliance'>Compliance (ISO27001 and GDPR)</a>

includes:
  # errors

search: true
---
```JavaScript




  Please scroll down to see the code for your Auth0 rule. 
```

![Email Hippo and Auth0 integration](images/HeaderImage.png)

# Introduction

The purpose of this function is to prevent new users from signing up to your Auth0 authenticated services with bad or disposable email addresses. Allowing use of email addresses which will hard bounce or are disposable will mean any subsequent attempt at contact with the user after sign-up will fail. Disposable email addresses are an early indicator of fraud. 

# About Email Hippo

Email Hippo is a an email validation service and data services provider you can trust.

We provide accurate, guaranteed cloud-based email validation technology globally under ISO 27001 standards.

Businesses use Email Hippo to get cleaner email data, sort bad email addresses from lists and sign-ups and prevent disposable and other bad email addresses getting onto systems.

You can be up and running with MORE, the Email Hippo API in fifteen minutes or less. MORE delivers 74 datapoints about every email address, so you can filter sign-ups, spot disposable emails and keep your data clean.

[Link to Email Hippo](https://emailhippo.com)

# About Auth0

Auth0 is an Identity as a Platform (IDaaS) service provider.

Auth0 is Identity made simple and secure. With Auth0, you take perhaps the riskiest endeavor your engineering organization will face, managing and securing the identities of your customers and employees, and abstract away this complexity and risk with our Identity Platform. 

Our standards-based Identity-as-a-Service platform is built for developers, by developers so that your engineering organization can focus on building the solutions that delight your customers and drive your revenue. 

Our customers find that what typically either takes months to deliver or simply cannot be delivered via internal or external solutions, takes them only days to deliver with Auth0 due to our extensive SDKs, intuitive API, extensible architecture and our simple management dashboard. 

This is why Auth0 is the leader in developer-focused Identity Management, with more than 5,000 customers trusting us with billions of transactions everyday.

[Link to Auth0](https://auth0.com)

# Configuration

## Prerequisites

1. An Auth0 account with a tenant setup

2. An Email Hippo account with a MORE API subscription and access to your API key.

**To create an account and purchase a subscription for the MORE API please visit** https://emailhippo.com 

## Configuration on Email Hippo

Once you have a subscription set up and your API key there is no further setup required within Email Hippo.

For further information on the MORE API please visit https://www.emailhippo.com/resources/technical-resources/

## Configuration on Auth0

1. Go to the Rules option on the menu

2. Under Settings on this page add a new key value

3. Set the key as 'HIPPO_API_KEY' and the value as your Email Hippo API key

4. Click on ‘+ Create Rule’

5. Select the ‘Empty Rule’ template

6. Name your rule - for example ‘Email Hippo Email Address Validation’

7. Replace the code displayed in Auth0 with the JavaScript shown here

8. Click on ‘Save’ or ‘Try this rule’ to use the function within your Auth0 sign up form and prevent sign ups with bad or disposable email addresses.

**The MORE API (Edition2/Version3) contains multiple data points which you may wish to incorporate in your function, for example for prompting re-input of mis-spelled email addresses.** 

**Our function uses the simple ‘result’ and ‘additional status’ to identify the email addresses which should not be accepted.**


```JavaScript

function (user, context, callback) {

  user.app_metadata = user.app_metadata || {};

  // Users with the emailhippo_valid will return an error on login
  // Setting emailhippo_valid to true will allow the user to log back in
  const valid = user.app_metadata.emailhippo_valid;
  if (valid !== undefined) {
    return valid ? callback(null, user, context) : callback('Email address is not valid');
  }

  if(!user.email) {
    return callback(null, user, context);
  }

  const request = require('request');
  
  const key = configuration.HIPPO_API_KEY;
  
  // Sign up at https://www.emailhippo.com/
  const url = 'https://api.hippoapi.com/v3/more/json/'+ key +'/' + user.email;

  request({ url: url }, function (err, resp, body) {
    if (err) {
        return callback(null, user, context);
    }
    if (resp.statusCode !== 200) {
        return callback(null, user, context);
    }

    const hippo_resonse = JSON.parse(body);
    
    const result = hippo_resonse.emailVerification.mailboxVerification.result;
    const reason = hippo_resonse.emailVerification.mailboxVerification.reason;

    user.app_metadata = user.app_metadata || {};

    // Any email address that is either bad or a Disposable email address
    // will be flagged as invalid. You can add your own custom logic if you want.
    let valid = true;
    if (result === 'Bad' || (result === 'Unverifiable' && reason === 'DomainIsWellKnownDea')){
        valid = false;
    } 

    user.app_metadata.emailhippo_result = result;
    user.app_metadata.emailhippo_reason = reason;
    user.app_metadata.emailhippo_valid = valid;

    auth0.users.updateAppMetadata(user.user_id, user.app_metadata)
      .then(function(){
        return valid ? callback(null, user, context) : callback('Email address is not valid');
      })
      .catch(function(err){
        callback(null, user, context);
      });
  });
}

```

