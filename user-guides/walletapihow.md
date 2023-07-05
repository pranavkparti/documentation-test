---
title: Wallet API
layout: default
parent: User Guides
nav_order: 3
---

# How to Use the Treetracker Wallet API

*Contents:*
- [Purpose](#purpose)
- [Your API Keys](#your-api-keys)
- [Sample JavaScript](#sample-javascript)
- [Sample Bash and cURL](#sample-bash-and-curl)
- [Authentication](#authentication)
- [List Your Trees](#list-your-trees)
- [Pass Trees to a Client](#pass-trees-to-a-client)
- [Send Trees to a New Manager](#send-trees-to-a-new-manager)
- [Ask for More Trees](#ask-for-more-trees)
- [Trust Someone to Give You Tokens Anytime](#trust-someone-to-give-you-tokens-anytime)
- [Ask for Trust to Give, Take, or Both](#ask-for-trust-to-give-take-or-both)
- [Stop Trusting](#stop-trusting)

You may also find help in a more-detailed, less-explanatory document, the [Treetracker API Reference](walletapiref.md).

## Purpose

You can read, write, download, and move your Greenstand tree data with Treetracker's application programing interface (API).

Send HTTP requests to the API server. The server replies with the results of your request.

For example, send
```
GET https://prod-k8s.treetracker.org/wallet/tokens?limit=1
```

Treetracker returns data about the first tree token in your Treetracker wallet.
```
{ tokens: [ {
  id: "35ab365b-8864-42d4-8ba6-29700dfba334",
  capture_id: "e077293b-61e6-49ab-90d7-207d2f14e06f",
  wallet_id: "f2832ab6-cc58-4922-920a-568a3b63e247",
  transfer_pending: false,
  transfer_pending_id: null,
  created_at: "2021-08-26T20:19:06.089Z",
  updated_at: "2021-08-26T20:19:06.089Z",
  origin: null,
  claim: false,
  links: { capture: "/webmap/tree?uuid=e077293b-61e6-49ab-90d7-207d2f14e06f" }
} ] }
```

Almost all Treetracker data moves in JSON objects, like the example above.

These instructions explain typical uses of the wallet API, and they provide examples in JavaScript and cURL code.

These instructions assume you are familiar with those languages, and with the basics of sending and receiving HTTP requests and parsing JSON data.

[^ back to top ^](#how-to-use-the-treetracker-wallet-api)  

## Your API Keys

To use the API you need to get three keys from Greenstand:
- `TREETRACKER-API-KEY`
- `wallet name`
- `wallet password`

All API requests need two headers:
- `TREETRACKER-API-KEY:<api-key>`
- `Content-Type:application/json`

Your first API request--your [authentication](#authentication) request--uses your 
`wallet name` and `password` to get another key:
- `bearer token`

All subsequent requests need the bearer token in a third header:
- `Authorization:Bearer <token>`

[^ back to top ^](#how-to-use-the-treetracker-wallet-api)  

## Sample JavaScript

These instructions describe HTTP methods, URL paths, and HTTP message bodies. For example, send an authentication request like this:
```
Method: POST
Path: /wallet/auth
Body: {"wallet": "<name>", "password": "<password>"}
```
You can substitute those values into the following sample of JavaScript code for a NodeJS environment.
```
//-- JavaScript for NodeJS ----------------
//
// Set 5 variables below.
// For the body, sometimes supply a JSON data object.
//   Sometimes supply six characters: "null"
// bearerToken is the value returned by your first request:
//   POST /wallet/auth {"wallet": "<name>", "password": "<password>"}
// A bearerToken remains valid for about one year.
//
// Modify 2 functions below to suit your need:
//   handleReply() and handleError()
//
//-- Set 5 variables ----------------------
var config={
  method:"<Method>",
  path:"<Path>",
  body:<Body>,
  apikey:"<yourApiKeyFromGreenstand>",
  bearerToken:"<bearerToken>",
  host:"prod-k8s.treetracker.org"
} // end config
//
//-- Handle reply -------------------------
const handleReply=function(code,msg,obj){
  console.log(code+': '+msg);
  console.log(obj);
} // end handleReply
//
//-- Handle error -------------------------
const handleError=function(src,code,msg,body){
  console.log(src);
  console.log(code+': '+msg);
  console.log(body);
} // end handleError
//
//-- Function to send request -------------
const https=require('https');
const sendRequest=function(){
  try{
    //-- Define request -------------------
    let options={
      "method":config.method,
      "hostname":config.host,
      "path":config.path,
      "headers":{ 
        "TREETRACKER-API-KEY":config.apikey,
        "Content-Type":"application/json",
        "Authorization":"Bearer "+config.bearerToken
      },
      "maxRedirects":20
    };
    var req = https.request(options,function(reply){
      //-- Handle response ----------------
      reply.on("error",function(err){handleError('Reply error',499,err,null);});
      var chunks = [];
      reply.on("data",function(chunk){chunks.push(chunk);});
      reply.on("end",function(){
        let code=parseInt(reply.statusCode);
        let msg=reply.statusMessage;
        let body=Buffer.concat(chunks).toString();
        let obj=null;
        try{obj=JSON.parse(body);} 
        catch(e){handleError('JSON parse error',code,msg,body);return;}
        if((code<200)||(code>299)){handleError('Reply code error',code,msg,obj);return;}
        handleReply(code,msg,obj);
      });
    });
    
    //-- If request has a body, send it ---
    if((config.body)&&(typeof config.body=='object')){
      req.write(JSON.stringify(config.body));
    }//if
    //-- end ------------------------------
    req.end();
  }//try
  catch(err){handleError('sendRequest() error',499,err,null);}
}// end sendRequest
//
//-- Execute ------------------------------
sendRequest();
//-- End JavaScript -----------------------
//-----------------------------------------
```

[^ back to top ^](#how-to-use-the-treetracker-wallet-api)  

## Sample Bash and cURL

For Bash and cURL on Mac or Linux, here is sample code.
```
# -- Bash and cURL -------------------------
#
# Set 5 variables below.
# Path values usually need 'quotes.' 
# For the body, sometimes supply a JSON-formatted string 
#   inside single quotes: '{"key":"value"}'
#   Sometimes supply four characters: null
#   Boolean values need quotes '{"name":"true"}'
# bearerToken is the value returned by your first request: 
#   POST /wallet/auth {"wallet": "<name>", "password": "<password>"}
#   A bearerToken remains valid for about one year.
#
# -- Set 5 variables -----------------------
method=<Method>
path='<Path>'
body='<Body>'
#body=null
apikey='TREETRACKER-API-KEY:'<yourApiKeyFromGreenstand>
bearerToken=<bearerToken>
host='https://prod-k8s.treetracker.org'
type='Content-Type:application/json'
#
# -- Send request --------------------------
if [[ $body == "null" ]]; then
  curl -L -X $method $host$path -H $apikey -H $type -H 'Authorization: Bearer '${bearerToken} 
fi
if [[ $body != "null" ]]; then
  curl -L -X $method $host$path -H $apikey -H $type -H 'Authorization: Bearer '${bearerToken} -d $body
fi
# -- End Bash ------------------------------
```

[^ back to top ^](#how-to-use-the-treetracker-wallet-api)  

## Authentication

Every new user of the API needs to start with an authentication request.

That request returns a "bearer token," a string of 852 characters that goes in the header of all subsequent requests. Without it, requests return an error. A bearer token is valid for about one year.

Send this request:
```
Method: POST
Path: /wallet/auth
Body: {"wallet": "<nameOfYourTreetrackerWallet>", "password": "<yourWalletPassword>"}
```

The API responds with:
```
200: OK
{ token: "<852characters>" }
```

In subsequent requests, include this header
```
"Authorization":"Bearer <852characters>"
```

If you do not, or if the bearer token has expired, the API responds with:
```
403: ERROR: Authentication, token not verified.
```

Or simply:
```
{"code":500,"message":"Unknown error (undefined)"}
```

[^ back to top ^](#how-to-use-the-treetracker-wallet-api)  

## List Your Trees

Treetracker describes trees in data objects called *tree tokens*.

Tokens for your trees are stored in your wallets.

To get a list of the tokens in a given wallet, send this request:
```
Method: GET
Path: /wallet/tokens?limit=999&wallet=<walletName>
```

The API responds with an array of token objects:
```
200: OK
{ tokens: [ {<token>},{<token>},{<token>}, ... ] }
```

Each token object looks like this:
```
{
  id: '2d36cf63-8cb5-4067-8d5e-faa392eb5306',
  capture_id: '3cbdcc97-e4fc-433b-b3bb-521847d19c84',
  wallet_id: 'f2832ab6-cc58-4922-920a-568a3b63e247',
  transfer_pending: false,
  transfer_pending_id: null,
  created_at: '2021-08-26T20:19:06.089Z',
  updated_at: '2021-08-26T20:19:06.089Z',
  origin: null,
  claim: false,
  links: {capture: "/webmap/tree?uuid=6fdd1365-dae5-465c-af6d-4e87e3f25634"}
}
```

The links.capture value is the path to more complete data about the tree, for example:
```
https://prod-k8s.treetracker.org/webmap/tree?uuid=6fdd1365-dae5-465c-af6d-4e87e3f25634
```

Note that the tree ID in the link is different than the token ID.

[^ back to top ^](#how-to-use-the-treetracker-wallet-api)  

## Pass Trees to a Client or Friend

Suppose Alice runs a landscaping business, and she wants to encourage a favorite customer, Bob,
by giving him some of her Greenstand trees.

Alice's task is to create a wallet, put some tree tokens into it, and give Bob the link to view them.

First, Alice creates Bob's wallet with this API request:
```
Method: POST
Path:   /wallet/wallets
Body:   {"wallet": "BobsWallet"}
```

The API responds thus:
```
200: OK
{wallet: "BobsWallet"}
```

Second, Alice moves three of her tokens into Bob's Wallet:
```
Method: POST
Path: /wallet/transfers
Body: 
{
  "sender_wallet": "AlicesWallet",
  "receiver_wallet": "BobsWallet",
  "tokens": ["<token_id>","<token_id>","<token_id>"]
}
```

The API responds with a *transfer object*:
```
201: Created
{
  id: '47c3e3b6-b2be-41e4-8264-37df44df66a6',
  type: 'send',
  parameters:{ tokens: ["<token_id>","<token_id>","<token_id>"] }
  state: 'completed',
  created_at: '2021-10-01T12:27:17.940Z',
  closed_at: '2021-10-01T12:27:17.940Z',
  active: true,
  claim: false,
  originating_wallet: 'AlicesWallet',
  source_wallet: 'AlicesWallet',
  destination_wallet: 'BobsWallet'
}
```

Now Alice can direct Bob (or anyone else) to find his trees on the map:
```
https://map.treetracker.org/?wallet=BobsWallet
```

Note that Bob does not *manage* his wallet. Alice does. BobsWallet does not have its own password. Bob cannot use the API. Only Greenstand administrators can create a new user account with a new managed wallet.

[^ back to top ^](#how-to-use-the-treetracker-wallet-api)  

## Send Trees to a New Manager

Alice and Bob are generous people who support tree farmers. They use Treetracker tokens and wallets to measure their success.

Alice and Bob both manage their own wallets. Alice wants to transfer 200 tokens from Alice's wallet to Bob's.

It's a two-stop process:
1. Alice requests the transfer.
2. Bob *accepts* it.

First, Alice posts this request:
```
Method: POST
Path: /wallet/transfers
Body: 
{
  "sender_wallet": "AlicesWallet",
  "receiver_wallet": "BobsWallet",
  "bundle":{"bundle_size": 200}
}
```

The API replies with a *transfer object*:
```
202: Accepted
{
  id: '47c3e3b6-b2be-41e4-8264-37df44df66a6',
  type: 'send',
  parameters: { bundle: { bundleSize: 200 } },
  state: 'pending',
  created_at: '2021-11-01T12:27:17.940Z',
  closed_at: '2021-11-01T12:27:17.940Z',
  active: true,
  claim: false,
  originating_wallet: 'AlicesWallet',
  source_wallet: 'AlicesWallet',
  destination_wallet: 'BobsWallet'
}
```

Note the `state` of that transfer: `pending`. 
The transfer is not complete, despite the response message: `Accepted`. Also note the `id` of that transfer.

Second, Bob reads that same transfer object with this request:
```
Method: GET
Path: /wallet/transfers?limit=1&wallet=BobsWallet&state=pending
```

The API replies with an array of transfer objects.
```
200: OK
{ transfers: [ {<transfer>},{<transfer>},{<transfer>}, ... ] }
```

In that array, Bob finds the transfer request that originated with Alice:
```
id: '47c3e3b6-b2be-41e4-8264-37df44df66a6',
...
state: pending,
...
originating_wallet: 'AlicesWallet',
source_wallet: 'AlicesWallet',
destination_wallet: 'BobsWallet'
```

Bob completes the transfer by copying the id and sending this request:
```
Method: POST
Path: /wallet/transfers/47c3e3b6-b2be-41e4-8264-37df44df66a6/accept
```

The API responds with a revised transfer request object. The `state` 
is now `completed`.
```
200 OK
{
  id: '47c3e3b6-b2be-41e4-8264-37df44df66a6',
  type: 'send',
  parameters: { bundle: { bundleSize: 200 } },
  state: 'completed',
  created_at: '2021-11-01T22:27:17.940Z',
  closed_at: '2021-11-01T22:27:17.940Z',
  active: true,
  claim: false,
  originating_wallet: 'AlicesWallet',
  source_wallet: 'AlicesWallet',
  destination_wallet: 'BobsWallet'
}
```

[^ back to top ^](#how-to-use-the-treetracker-wallet-api)  

## Ask for More Trees

Alice and Bob are generous people who support tree farmers. They use Treetracker tokens and wallets to measure their success.

Alice and Bob both manage their own wallets. Bob wants Alice to give him 200 of her tokens.

It's a two-stop process:
1. Bob requests the transfer.
2. Alice *fulfills* it.

First, Bob posts this request:
```
Method: POST
Path: /wallet/transfers
Body: 
{
  "sender_wallet": "AlicesWallet",
  "receiver_wallet": "BobsWallet",
  "bundle":{"bundle_size": 200}
}
```

The API replies with a *transfer object*:
```
202: Accepted
{
  id: '47c3e3b6-b2be-41e4-8264-37df44df66a6',
  type: 'send',
  parameters: { bundle: { bundleSize: 200 } },
  state: 'requested',
  created_at: '2021-11-01T12:27:17.940Z',
  closed_at: '2021-11-01T12:27:17.940Z',
  active: true,
  claim: false,
  originating_wallet: 'BobsWallet',
  source_wallet: 'AlicesWallet',
  destination_wallet: 'BobsWallet'
}
```

Note the `state` of that transfer: `requested`. 
The transfer is not complete, despite the response message: `Accepted`. Also note the `id` of that transfer.

Second, Alice reads that same transfer object with this request:
```
Method: GET
Path: /wallet/transfers?limit=1&wallet=AlicesWallet&state=requested
```

The API replies with an array of transfer objects.
```
200: OK
{ transfers: [ {<transfer>},{<transfer>},{<transfer>}, ... ] }
```

In that array, Alice finds the transfer request that originated with Bob:
```
id: '47c3e3b6-b2be-41e4-8264-37df44df66a6',
...
state: requested,
...
originating_wallet: 'BobsWallet',
source_wallet: 'AlicesWallet',
destination_wallet: 'BobsWallet'
```

Alice completes the transfer by copying the id and sending this request:
```
Method: POST
Path: /wallet/transfers/47c3e3b6-b2be-41e4-8264-37df44df66a6/fulfill
Body: {"implicit":true}
```

The API responds with a revised transfer request object. 
The `state` is now `completed`.
```
200 OK
{
  id: '47c3e3b6-b2be-41e4-8264-37df44df66a6',
  type: 'send',
  parameters: { bundle: { bundleSize: 200 } },
  state: 'completed',
  created_at: '2021-11-01T22:27:17.940Z',
  closed_at: '2021-11-01T22:27:17.940Z',
  active: true,
  claim: false,
  originating_wallet: 'BobsWallet',
  source_wallet: 'AlicesWallet',
  destination_wallet: 'BobsWallet'
}
```

[^ back to top ^](#how-to-use-the-treetracker-wallet-api)  

## Trust Someone to Give You Tokens Anytime

Alice is Bob's business partner. She often transfers tokens to Bob's wallet. 
So often, that it is a nuisance for Bob to explicitly accept each and every transfer.

So Alice and Bob create a *trust relationship*, as follows.

First, Bob sends a request to create the trust:
```
Method: POST
Path: /wallet/trust_relationships
Body: 
{
  "trust_request_type": "receive",
  "requestee_wallet": "AlicesWallet"
}
```

The API replies with a *trust relationship object*:
```
200: OK
{
  id: b6b9ed89-5bd4-4b53-9b60-609fb78dc119
  type: send
  request_type: receive
  state: requested
  created_at: 2021-11-24T14:59:38.783Z
  updated_at: 2021-11-24T14:59:38.783Z
  originating_wallet: BobsWallet
  actor_wallet: BobsWallet
  target_wallet: AlicesWallet
}
```

Note the `state` of that trust: `requested`. Also note the `id` of that trust.

Second, Alice reads that same trust object with this request:
```
Method: GET
Path: /wallet/trust_relationships?limit=99&state=requested
```

The API replies with an array of trust objects.
```
200: OK
{ trust_relationships: [ {<trust_relationship>},{<trust_relationship>},{<trust_relationship>}, ... ] }
```

In that array, Alice finds the trust request that originated with Bob:
```
id: 'b6b9ed89-5bd4-4b53-9b60-609fb78dc119',
...
request_type: receive
state: requested,
...
originating_wallet: BobsWallet
actor_wallet: BobsWallet
target_wallet: AlicesWallet
```

Alice creates the trust relationship by copying the id and sending this request:
```
Method: POST
Path: /wallet/trust_relationships/b6b9ed89-5bd4-4b53-9b60-609fb78dc119/accept
```

The API responds with a revised trust object. The `state` is now `trusted`.
```
200: OK
{
  id: b6b9ed89-5bd4-4b53-9b60-609fb78dc119
  type: send
  request_type: receive
  state: trusted
  created_at: 2021-11-24T14:59:38.783Z
  updated_at: 2021-11-24T14:59:38.783Z
  originating_wallet: BobsWallet
  actor_wallet: BobsWallet
  target_wallet: AlicesWallet
}
```

From now on, Alice can transfer tokens to Bob without Bob's explicit permission. 
Alice can `POST /wallet/transfers` and the tokens will immediately move to BobsWallet. 
Bob does not need to find the transfer id and `POST /wallet/transfers/<transfer_id>/accept`.

[^ back to top ^](#how-to-use-the-treetracker-wallet-api)  

## Ask for Trust to Give, Take, or Both

In the instructions above, [Trust Someone to Give You Tokens Anytime](#trust-someone-to-give-you-tokens-anytime),
Bob wanted to let Alice transfer tokens to Bob. So he sent:
```
Method: POST
Path: /wallet/trust_relationships
Body: 
{
  "trust_request_type": "receive",
  "requestee_wallet": "AlicesWallet"
}
```

In other words, Bob, the *request**er***, wants to receive tokens from Alice, the *request**ee***.

There are four `trust_request_types`. They allow transfers in either direction or both: from Alice to Bob, from Bob to Alice, or both. As follows:

`receive`: The requester (Bob) lets the requestee (Alice) give him tokens.

`send`: The requester (Bob) can give tokens to the requestee (Alice).

`manage`: The requester (Bob) can both give tokens to, and take tokens from, the requestee (Alice).

`yield`: The requester (Bob) allows the requestee (Alice) to both give tokens to him and take tokens from him.
```
What trust_request_types mean:
Requestee/target gets to transfer tokens this way:
receive:    origin's wallet <--- requestee's token
yield:      origin's wallet <--- requestee's token
             origin's token ---> requestee's wallet
Originator/requester gets to transfer tokens this way:
send:         origin's token ---> requestee's wallet
manage:       origin's token ---> requestee's wallet
             origin's wallet <--- requestee's token
```

[^ back to top ^](#how-to-use-the-treetracker-wallet-api)  

## Stop Trusting

For a long time, Bob has trusted Alice to transfer tokens into his wallet whenever she wants. 
But Alice and Bob have moved on to different businesses. They need to break that trust relationship.

Either of them can do so:

Either of them finds the necessary trust relationship ID:
```
Method: GET
Path: /wallet/trust_relationships?limit=99&state=trusted
```

The API replies with an array of trust objects:
```
200: OK
{ trust_relationships: [ {<trust_relationship>},{<trust_relationship>},{<trust_relationship>}, ... ] }
```

In that array, they find the trust that let's Bob receive transfers from Alice:
```

{
  id: b6b9ed89-5bd4-4b53-9b60-609fb78dc119
  type: send
  request_type: receive
  state: trusted
  created_at: 2021-11-24T14:59:38.783Z
  updated_at: 2021-11-24T14:59:38.783Z
  originating_wallet: BobsWallet
  actor_wallet: BobsWallet
  target_wallet: AlicesWallet
}
```

They copy the id of that trust, then Bob uses it one way, Alice another:

Bob originated the trust; he sent the request that started it. So Bob can `DELETE` the trust:
```
Method: DELETE
Path: /wallet/trust_relationships/<trust_relationship_id>
```

The API's response is a revised trust relationship object. It says:
```
state: cancelled_by_originator
```

Alice accepted the trust; she agreed to Bob's request. So Alice can now `decline` the trust:
```
Method: POST
Path: /wallet/trust_relationships/<trust_relationship_id>/decline
```

The API's response is a revised trust_relationship object. It says:
```
state: cancelled_by_target
```

[^ back to top ^](#how-to-use-the-treetracker-wallet-api)  

--- end ---