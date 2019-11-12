# Bosch Smart Home Controller Local REST API

#### Contents

- [Overview](#overview)
- [How to use the Postman Collection](postman/)
  * [Import the collection and the certificate into Postman](postman#import-the-collection-and-the-certificate-into-postman)
  * [Add a New Client to the Bosch Smart Home Controller](postman#add-a-new-client-to-the-bosch-smart-home-controller)
- [Local OpenAPI Documentation](https://apidocs.bosch-smarthome.com/local/)
- [Terms and Conditions](#terms-and-conditions)


## Overview

<img src="images/bosch_smart_home.jpg"/>

[Bosch Smart Home](https://www.bosch-smarthome.com/) products allow you to automatically and remotely control the processes in your home. Each device has a unique purpose in your Smart Home and provides ease and convenience to your changing daily routines. Why do I need this? Because it means you can have more time to enjoy what matters to you. Whether you want to network your heating system or secure the safety of your home, the Bosch Smart Home System offers you complete and personalised solutions to you and your home's needs. This is all controlled by one handy app, so you can control your home, wherever you are. 


## Terms and Conditions
Robert Bosch Smart Home GmbH, Schockenriedstr. 17, 70565 Stuttgart, Germany (“Bosch Smart Home”) provides a description of the local Bosch Smart Home Controller REST API, that allows **private, non-profit** developers to locally control their Bosch Smart Home Devices.


### Definitions
"API" refers to the interface functionality provided by Bosch Smart Home allowing private, non-profit developers to access Bosch Smart Home Data. 

"Client" refers to software created by developers using the API. Clients can be, for example (but are not limited to), mobile apps or web services.

"Data" refers to data made available by accessing the API of a specific, local and personally owned Smart Home Controller ("SHC").

"Documentation" refers to the provided API description and usage examples.

"Commercial" means intended for or directed towards commercial advantage or monetary compensation. 


### Terms of Usage

**By using the Documentation to enable interoperability you agree to the following terms:**

The use of the API only applies to private, non-profit individuals. A commercial use in any way, either direct or indirect, is prohibited.

Using the API requires identification against the local Smart Home Controller with an individual Client ID and Client name that starts with "oss_" followed by the name of the open source project or the name of the developer.

The Client must register with the primary role "ROLE_RESTRICTED_CLIENT".

The use of claims (e.g. “works with Bosch”) or trademarks (e.g. Bosch logos of any kind) that indicates a direct or indirect involvement with Bosch is prohibited. 

Bosch Smart Home may at its sole and exclusive discretion, change, alter, modify, add and remove features, content and other materials of the Documentation and/or API at any time. Bosch Smart Home may suspend or discontinue any feature or functionality provided at any time without any prior notice, obligation or liability. The developer is aware that changes in the API might require changes in the Client. The costs for such changes have to be borne by the developer.

Any technical implementation and fulfillment of system requirements of the API is the sole responsibility of the developer. Developers are obliged to adhere to the requirements of the API, particularly regarding the correct technical integration and use of the API (see documentation). The user acknowledges that an incorrect technical integration or use may lead to defects in the functions of the API, or even the entire Client, or the connected Bosch Smart Home Devices.

Developers are obliged to comply with any and all of these guidelines, which are specified in this document. Developers shall, in particular, not bypass any authentication or encryption mechanisms, or misuse API interfaces for purposes other than those which are clearly intended by this Documentation (e.g. reading Data from the Bosch Smart Home System). 

Developers shall not use the API for any action that can cause damages due to the interaction of the API or the Client with the home appliances (e.g. by bypassing security prompts on the home appliances, or by executing commands that can cause harm to life or limb, or to the home appliances, or to any other goods).

Developers may produce and reproduce this Documentation for their own use, but may not share a modified or derived version of this Documentation.

Developers shall fully comply with all applicable legal provisions when using the API, including but not limited to, copyright, trademark rights and data protection.

Developers shall fully comply with all applicable governmental, legal, regulatory and professional requirements.

Developers shall not use the API for publication of explicit content or offensive communication.


### License
By using this Documentation, the developer accepts and agrees to be bound by these Terms and Conditions (defined above). 

The Documentation is subject to the Creative Commons Attribution-NonCommercial-NoDerivatives 4.0 International Public License (https://creativecommons.org/licenses/by-nc-nd/4.0/legalcode).


### Warranty
We ask all developers working with our Documentation to inform Robert Bosch Smart Home GmbH about any defects found in the API as soon as possible to give us a chance to remedy it.


### Support 
Unfortunately, we are unable to support developers using the Documentation neither via the official hotline nor by e-mail. However, we will try our best to respond to issues raised in [our GitHub tracker](https://github.com/BoschSmartHome/bosch-shc-api-docs/issues). Please report any trouble you might have with the Documentation exclusively there.


*October 29th, 2019*
