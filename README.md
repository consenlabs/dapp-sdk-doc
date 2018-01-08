## Overview

### web3
When imToken DApp browser load DApp page, we inject `web3.js` into DApp webiew automatically, so the `web3.js` imported by DApp is not necessary, you can access web3 object from `window.web3`.

By default, we set user's current wallet addrsss as the value of `web3.eth.defaultAccount`.
We also set the web3 provider as the same setting of imToken.


### imToken
imToken SDK add a namespace `imToken` to window object in DApp webview, because of the web3 file and SDK file are injected **after** webview loaded, your code (which will invoke web3 and SDK) should run after SDK script injected. 

Before invoke imToken SDK API and the injected `web3`, You should check if `imToken` namespace is exist. We provide a `sdkReady` event which will emit after injected. you can listen this event on `window` object:
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

imToken DApp SDK expose an public method `callAPI`, API invoke should always code as this format :

```js
imToken.callAPI(apiName, params, callback)
```

- *apiName* is which api you want to call, you can see the available apiNames below.
- *params* (optional) is the params will pass to apiName
- *callback* (optional) is a callback function, which follow NodeJS style, the first param pass to callback function is always `err` (if no error occured, `err` is `null`, otherwise it's an object with a message property), and the second param is the result return by *apiName*.  

There is anthor public method `callPromisifyAPI` which do the same thing as `callAPI`, this maybe useful if you want to call api as Promise style:
```js
imToken.callPromisifyAPI(apiName, params).then(result => 
  console.log(result)
).catch(err => {
  console.error(err)
})
```
*📌 your DApp page should include a Promise polyfill already!*

## Example

You can find the api example [here](https://consenlabs.github.io/dapp-sdk-doc/index.html).


## API:

Because of *apiName* is always string, we will only explain the *params* and *callback*.

### native.alert
> display an native `alert` to show a message, like window.alert.

```typescript
// @types
  params: string
```

```js
// @example
imToken.callAPI('native.alert', 'winner winner, chicken dinner！')
```

### native.confirm
> display an native `confirm` dialog, like window.confirm, if user click the button, will return an error to callback function.

```typescript
// @types
  params: {
    title: string
    message: string
    cancelText: string
    confirmText: string
  }
```

```js
// @example
imToken.callAPI('native.confirm', {
  title: 'quick question',
  message: 'is imToken the best useful wallet?',
  cancelText: 'no',
  confirmText: 'yes',
}, function(err, result) {
  if(err) {
    console.log('no')
  } else {
    console.log('yes')
  }
})
```

### native.selectPicture
> select a picture from library or take a photo, will return image data in base64.
```typescript
//@types
  params: {
    maxWidth: number    // optional,  response will be corpped.
    maxHeight: number     // optinal,  response will be corpped.
    allowsEditing: boolean // optinal, enables built in iOS functionality to resize the image after selection
    quality: number // optinal, 0 to 1
  }
```

```js
// @example
imToken.callAPI('native.selectPicture',{maxWidth: 400, maxHeight: 200}, function (err, ret) {
    if(err) {
      alert(err.message)
    } else {
      document.getElementById('imgContainer').src = ret.data
    }
  })
```

### native.setClipboard
> set text to user clipboard
```typescript
//@types
  params: string
```

```js
// @example
imToken.callAPI('setClipboard', 'are you ok?')
```

### native.share
> open native share component

```typescript
// @types
  params:  {
      title: string,
      message: string,
      url: string
    }
```

```js
// @example
imToken.callAPI('native.share', {
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

### native.scanQRCode
> open an scanner to scan QRCode, return string content of QRCode.

```js
// @example
 imToken.callAPI('scanQRCode', function (err, text) {
    if(err) {
      alert(err.message)
    } else {
      alert(text)
    }
  })
```

### native.setCalEvent
> create an calEvent

```typescript
// @types
  params:  {
    title: string,
    settings: {
      notes: string,
      startDate: string,
      endDate: string,
    }
  }
```

```js
//@example
imToken.callAPI('native.setCalEvent', {
    title: 'test save event',
    settings : {
      notes: 'go to china this night',
      startDate: '2017-09-28T19:42:00.000Z',
      endDate: '2017-09-28T09:19:44.000Z',
    }
  }, function(err, id) {
    if(err) {
      alert(err.message)
    } else {
      alert(id)
    }
  })
```

### navigator.closeDapp
> close current dapp webview

```js
// @example
imToken.callAPI('navigator.closeDapp')
```

### navigator.goBack
> go to prev history of current webview, like history.go(-1)
>
> if no prev history, close current dapp webview

```js
// @example
imToken.callAPI('navigator.goBack')
```

### navigator.toggleNavbar
> by default, imToken anded a navbar on top of dapp webview outside.
>
> so users can control the webview navigation (like close、goBack、reload、share..)
>
> if you implement your own navbar in webview, you can hide the default navbar by call `toggleNavbar`. 


```js
// @example
imToken.callAPI('navigator.toggleNavbar')
```

### transaction.tokenPay
> open an imToken pay modal to send a transaction, if pay success, will return the txHash to callback function.

```typescript
// @types
  params:  {
    contractAddress: string,
    to: string,
    from: string, 
    value: string, // bigNumber string,  unit in decimal
    orderInfo: string, // pay modal title
    customizable: boolean, // control whether user can select payer wallet and custom gas
  }
```

```js
// @example
var params = {
    contractAddress: '0x0000000000000000000000000000000000000000',
    to: '0x0fa38abec02bd4dbb87f189df50b674b9db0b468',
    from: web3.eth.defaultAccount,
    value: '1250000000000000',
    orderInfo: 'buy a cup of coffee',
    customizable: true,
  }

  imToken.callAPI('transaction.tokenPay', params, function (err, hash) {
    if (err) {
      alert(err.message)
    } else {
      // return txHash of this transaction
      alert(hash)
    }
  })
```

### user.getCurrentAccount
> get current wallet address

```js
//@example
imToken.callAPI('user.getCurrentAccount', function(err, address) {
    if(err) {
      alert(err.message)
    } else {
      alert(address)
    }
  })
      
```

#### user.getAccountList
> get user's wallet address list, return an Array of wallet address list.

```js
//@example
imToken.callAPI('user.getAccountList', function(err, list) {
    if(err) {
      alert(err.message)
    } else {
      alert(list)
    }
   })
       
```     

### user.getAssetTokens
> get user's assetToken list, return an Array of *tokenObject* list.
>
> **need permission**: if user refused, you will get an err: 'user refused'.

```javascript
// *tokenObject* type
{
  chainType: string // ETHEREUM | BITCOIN
  address: string  // contractAddress
  walletAddress: string
  symbol: string
  decimal: number
  unit: string
  logo: string
  balance: string // bigNumber, unit in decimal
  defaultGas: string
  price: number
  profileUrl: string
}
```


```js
// @example
imToken.callAPI('user.getAssetTokens', function(err, assetTokens){
    if(err) {
      alert(err.message)
    } else {
       alert(assetTokens.map(function(t){
        return t.symbol + ' '
      }))
    }
  }) 
```  

### user.showAccountSwitch

> open a wallet switch modal
return a wallet address depend on which user selected.

```typescript
//@types
  params:  {
    contractAddress: string // pass a token contractAddress, only show wallets which has this token
  }
```

```js
// @example
imToken.callAPI('user.showAccountSwitch', function(err, address){
    if(err) {
      alert(err.message)
    } else {
      alert(address)
    }
  }) 
```  

### user.getContacts
> get wallet contacts of user, return an Array of *ContactObject* list.
>
> **need permission**: if user refused, you will get an err: 'user refused'.

```js
// *ContactObject* type
{
  id: string // chainType-address
  chainType: string // ETHEREUM | BITCOIN
  description: string
  name: string
  address: string
  timestamp: number
}
```

```js
//@example
imToken.callAPI('user.getContacts', function(err, contacts) {
     if(err) {
       alert(err.message)
     } else {
       alert(contacts)
     }
   })
      
```  

### user.getProfile
> get user profile, return a `ProfileObject`, contains username、avatar、kyc... etc.
>
> **need permission**: if user refused, you will get an err: 'user refused'.

```js
// *ProfileObject*
{
  name: string
  avatar: string
}
```


```js
// @example
imToken.callAPI('user.getProfile', function(err, profile) {
  // TODO: profile detail
     if(err) {
       alert(err.message)
     } else {
       alert(profile)
     }
   })
```  

### device.getCurrentLanguage

> if your dapp need to support multi-language, maybe this info is useful
> we already and locale param to your dapp url, most time you no need to call this api.

*available locale:*
- zh-CN
- en-US
- zh-Hant-US
- zh-Hant-HK
- zh-Hant-TW


```js
// @example
imToken.callAPI('device.getCurrentLanguage', function(err, language) {
     if(err) {
       alert(err)
     } else {
       alert(language)
     }
   })
```  

### device.getCurrentCurrency

> get current currency of imToken, if your dapp need to support multi-language, maybe this info is useful
> we already and currency param to your dapp url, most time you no need to call this api.

*available currency:*
- CNY
- USD
- TWD
- HKD

```js
//@example
imToken.callAPI('device.getCurrentCurrency', function(err, currency) {
     if(err) {
       alert(err)
     } else {
       alert(currency)
     }
   })
```  

