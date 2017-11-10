## Overview

### web3
When imToken browser load DApp, we inject `web3.js` into DApp webiew automatic, so the `web3.js` import by DApp is not necessary, you can read web3 object from `window.web3`.

By default, we set user's current Wallet addrsss as the value of `web3.eth.defaultAccount`.
We also set the web3 provider as the same setting of imToken.


### imToken
imToken SDK add a namespace `imToken` to window object in DApp webview, because of the injected web3 and sdk are injected after webview loaded, your code (which invoke web3 or sdk) should run after sdk script injected. 

Before invoke imToken SDK API and the injected `web3`, You should check if `imToken` namespace exist. We provide a `sdkReady` event which will emit after injected. you can listen this event on `window` object:
```
if(window.imToken) {
  // run your code
} else {
  window.addEventListener('sdkReady', function() {
    // run your code
  })
}

```

### callAPI

imToken DApp SDK only expose one public method `callAPI`, API invoke should always code as this format :

> `imToken.callAPI(apiName, params, callback)` 

- *apiName* is which api you want to call, you can see the available apiNames below.
- *params* is the params will pass to apiName
- *callback* is a callback function, which follow NodeJS style, the first param pass to callback function is always `err` (for keep simple, it just a string message), and the second param is the result return by *apiName*.  


## Example

You can find the api example [here](https://consenlabs.github.io/dapp-sdk-doc/index.html).


## API:

Because of *apiName* is always string, we will only explain the *params* and *callback*.

### alert
> display an native `alert` to show a message, like window.alert.

```typescript
//@types
  params: string
  callback: null
```

```
//@example
imToken.callAPI('alert', 'winner winner, chicken dinner！')
```

### close
> close current dapp webview

```typescript
//@types
  params: null
  callback: null
```

```
//@example
imToken.callAPI('close')
```

### goBack
> go to prev history of current webview, like history.go(-1)
if no prev history, close current dapp webview

```typescript
//@types
  params: null
  callback: null
```

```
//@example
imToken.callAPI('goBack')
```

### toggleNavbar
> by default, imToken will and a navbar out of dapp webview
  so the user can control the webview navigation (like close、goBack、reload、share..)
  if you implement your own navbar in webview, you can hide default navbar by `toggleNavbar`. 


```typescript
//@types
  params: null
  callback: null
```


```js
//@example
imToken.callAPI('toggleNavbar')
```

### setClipboard
> set text to user clipboard
```typescript
//@types
  params: string
  callback: null
```

```js
//@example
imToken.callAPI('setClipboard', 'are you ok?')
```

### share
> open native share component

```typescript
//@types
  params:  {
      title: string,
      message: string,
      url: string
    }
  callback: null
```

```js
//@example
imToken.callAPI('share', {
    title: 'dapp example',
    message: 'this is example of dapp sdk',
    url: location.href,
  }, function (err, ret) {
    if(err) {
      alert('user not share')
    } else {
      alert('shared')
    }
  })
      
```

### scan
> open an scanner to scan QRCode, return string content of QRCode

```typescript
//@types
  params:  null
  callback: function
```

```js
//@example
 imToken.callAPI('scan', null, function (err, ret) {
    if(err) {
      alert(err)
    } else {
      alert(ret)
    }
  })
```

### setCalEvent
> create an calEvent

```typescript
//@types
  params:  {
    title: string,
    settings: {
      notes: string,
      startDate: string,
      endDate: string,
      alarms?: [{   // ios only
        date: number
      }]
    }
  }
  callback: function
```

```js
//@example
imToken.callAPI('setCalEvent', {
    title: 'test save event',
    settings : {
      notes: 'go to china this night',
      startDate: '2017-09-28T19:42:00.000Z',
      endDate: '2017-09-28T09:19:44.000Z',
      alarms: [{
        date: -1 //ios only
      }]
    }
  }, function(err, id) {
    if(err) {
      alert(err)
    } else {
      alert(id)
    }
  })
```

### routeTo
> routeTo a specific imToken screen

```typescript
//@types
  params:  string
  callback: null
```

```js
//@example
imToken.callAPI('routeTo', 'Profile')
      
```

### createPay
> open an imToken pay modal to send a transaction

```typescript
//@types
  params:  {
    contractAddress: string,
    to: string,
    from: string, 
    value: string, // bigNumber string,  unit in decimal
    orderInfo: string, // pay modal title
    customizable: boolean, // control if user can change payer wallet, and custom gas
  }
  callback: function
```

```js
//@example
var params = {
    contractAddress: '0x0000000000000000000000000000000000000000',
    to: '0x0fa38abec02bd4dbb87f189df50b674b9db0b468',
    from: web3.eth.defaultAccount,
    value: '1250000000000000',
    orderInfo: 'buy a cup of coffee',
    customizable: true,
  }

  imToken.callAPI('createPay', params, function (err, hash) {
    if (err) {
      alert(err)
    } else {
      alert(hash)
    }
  })
```

### getCurrentAccount
> get current wallet address
```typescript
//@types
  params:  null
  callback: function
```

```js
//@example
imToken.callAPI('getCurrentAccount', null, function(err, address) {
    if(err) {
      alert(err)
    } else {
      alert(address)
    }
  })
      
```

#### getAccountList
> get user's wallet address list

```typescript
//@types
  params:  null
  callback: function
```

```js
//@example
imToken.callAPI('getAccountList', null, function(err, list) {
    if(err) {
      alert(err)
    } else {
      alert(list)
    }
   })
       
```     

### getAssetTokens
> get user's assetToken list
>
> **need permission**: if user refused, you will get an err: 'user refused'.

```typescript
//@types
  params:  null
  callback: function
```

```js
//@example
imToken.callAPI('getAssetTokens', null, function(err, assetTokens){
    if(err) {
      alert(err)
    } else {
       alert(assetTokens.map(function(t){
        return t.symbol + ' '
      }))
    }
  }) 
```  

### showAccountSwitch

> open a wallet switch modal
return a wallet address depend on which user selected.

```typescript
//@types
  params:  {
    contractAddress: string // pass a token contractAddress, only show wallets which has this token
  }
  callback: function
```

```js
//@example
imToken.callAPI('getAssetTokens', null, function(err, assetTokens){
    if(err) {
      alert(err)
    } else {
       alert(assetTokens.map(function(t){
        return t.symbol + ' '
      }))
    }
  }) 
```  

### getContacts
> get wallet contacts of user
>
> **need permission**: if user refused, you will get an err: 'user refused'.

```typescript
//@types
  params: null
  callback: function
```

```js
//@example
imToken.callAPI('getContacts', null, function(err, contacts) {
     if(err) {
       alert(err)
     } else {
       alert(contacts)
     }
   })
      
```  

### getCurrentLanguage

> get current language of imToken, if your dapp need to support multi-language, maybe this info is useful.

```typescript
//@types
  params: null
  callback: function
```

```js
//@example
imToken.callAPI('getCurrentLanguage', null, function(err, language) {
     if(err) {
       alert(err)
     } else {
       alert(language)
     }
   })
```  

### getCurrentCurrency

> get current currency of imToken, if your dapp need to support multi-language, maybe this info is useful.

```typescript
//@types
  params: null
  callback: function
```

```js
//@example
imToken.callAPI('getCurrentCurrency', null, function(err, currency) {
     if(err) {
       alert(err)
     } else {
       alert(currency)
     }
   })
```  

### getUserProfile
> get user profile, like username、avatar、kyc info...
>
> **need permission**: if user refused, you will get an err: 'user refused'.

```typescript
//@types
  params: null
  callback: function
```

```js
//@example
imToken.callAPI('getUserProfile', null, function(err, profile) {
  // TODO: profile detail
     if(err) {
       alert(err)
     } else {
       alert(profile)
     }
   })
      
```  