# Best Practice
In order to get the best user experience when using the local interface of your Bosch Smart Home Controller, we have a few best practice tips for you. These tips should be considered and implemented in any public open source project that interacts with the Bosch Smart Home Controller.

## Watch this repository
First of all, watch this repository to get notified when we change or update our Terms and Conditions or when there are changes or updates in the API. In case the Terms and Conditions are changed, the commit message will always contain the keyword `T&C`. In case the API is changed or updated, the commit message will always contain the keyword `API`.

## Use Long Polling instead of Short Polling
In order to get notified as soon as a value changes, it is recommended to use the [Long Polling mechanism to receive events](https://github.com/BoschSmartHome/bosch-shc-api-docs/tree/master/postman#get-events-from-the-bosch-smart-home-controller-long-polling) (see Postman collection 'Long Polling Subscribe'). Long Polling is a mechanism in which the client makes a request, and the server keeps the connection open until there is new information available. Once available, the server responds by sending the new information to the client and closes the connection afterwards. As soon as the client receives the new information, it immediately sends another request and the process is repeated. This mechanism also ensures that you do not miss any events, because the Bosch Smart Home Controller keeps all information for the client until the client starts another Long Polling request.
Please be aware that every subscription has a maximum age of 24 hours and an inactivity timeout of 5 minutes, i.e. a new subscription has to be requested if a subscription ID is either used for 24 hours or no long-poll request was sent for 5 minutes. A long-poll request will fail after the corresponding subscription ID has expired. Re-subscribing and then using the new subscription ID solves this.

### Limit the number of requests in a given time period
In addition to the use of the Long Polling mechanism, it is also advisable to keep the number of requests in a given time period low. The number of requests the Bosch Smart Home Controller can process depends heavily on how many edge devices and clients are paired with it. 

You should also consider that a large number of requests flood the log file. In the event of a support request, the error may no longer be found in the log file, making an error analysis difficult.

## Host Verification
The communication with the Bosch Smart Home Controller is cryptographically protected via TLS. However, it is recommended that your OSS implementation verifies the Bosch Smart Home Controller IP and the certificate authority (CA). You should avoid such implementation as the following (example given in Node.js):
```javascript
    const requestOptionsWithoutHostVerification = {
        key: fs.readFileSync('client-key.pem'),
        cert: fs.readFileSync('client-cert.pem'),
        rejectUnauthorized: false // No host verification at all
    }
```
Instead verify the IP and the CA:
```javascript
    const requestOptionsWithHostVerification = {
        key: fs.readFileSync('client-key.pem'),
        cert: fs.readFileSync('client-cert.pem'),
        ca: fs.readFileSync(tls_ca_chain), // Concatenate the Smart Home Controller Productive Root CA with the Smart Home Controller Issuing CA into one file
        checkServerIdentity: function(host) {
            if (host === shcIp) {
                return undefined;
            } else {
                return new Error('Server identity check failed!');
            }
        }
    }
```
The Smart Home Controller Productive Root CA and the Smart Home Controller Issuing CA are attached to this repository.