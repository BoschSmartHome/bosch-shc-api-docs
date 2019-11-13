# How to use the Bosch Smart Home Postman Collection

To get started with the Postman Collection you need the following:

- Postman Collection downloaded to your local environment
- The IP address of your Smart Home Controller (SHC)
- A generated 2048 bit self signed certificate
- The key to that certificate

## Import the collection and the certificate into Postman
1. Start by importing the downloaded Postman Collection via the **Import** button of Postman. 

2. In the next step, you need to add the generated certificate and key to Postman. Therefore, open the **Settings** in Postman:

![Postman Settings](images/postman_settings.png "Postman Settings")

2.  On the **General** tab, disable **SSL certificate verification**:

![Postman Disable SSL Verification](images/postman_disable_ssl_verification.png "Postman Disable SSL Verification")

3.  On the **Certificates** tab, click on **Add Certificate**:

![Postman Add Certificate](images/postman_add_certificate.png "Postman Add Certificate")

4. Enter the **IP address** of your SHC and the port **8443**. After that, provide the requested files for the **certificate** and the **key**. If you use a **passphrase**, you should provide this information as well:

![Postman Add PEMs](images/postman_add_pems.png "Postman Add PEMs")
	
6. Click **Add** to add the certificate to your Postman Settings.

7. Do the same for port **8444**.

Now, you have all the prerequisites to communicate with your SHC via Postman. Start by adding a new client.


## Register a New Client to the Bosch Smart Home Controller

A client is a software that has the permission to communicate with the Bosch Smart Home Controller. For example, the Bosch Smart Home App is a client of the the Bosch Smart Home Controller as soon as it has successfully registered to it. All registered clients can be listed in the Bosch Smart Home App. Have a look in your App and go to `Management -> Mobile devices`. The communication between a client and the Bosch Smart Home Controller is always encrypted with TLSv1.2.
![Schematic view](images/shc-client-schematic-view.png "Schematic view")
**Hint:** In order to register a New Client, always **press the button of the Bosch Smart Home Controller until the LEDs are flashing** before calling the **New Client** function.

To register a **New Client** to the Bosch Smart Home Controller you need the following:

- A designated name of your open source software project
- The 2048 bit self-signed certificate from the previous step
- The master password of your Bosch Smart Home Controller, which you created upon initial setup

**Hint:** Your Bosch Smart Home Controller must already be initialized and paired with a Bosch Smart Home App!

### Naming convention for the Client ID and Client Name
Using the API requires identification against the Bosch Smart Home Controller with an individual Client ID and Client Name that starts with `oss_` followed by the name of the open source project, or the name of the developer.

Provide the information of the **Client ID** and **Client Name** in the **body** of the **New Client** call.

### Customize the certificate
To modify the certificate so that it fits into a JSON object, you have to manually remove all carriage returns, and additionally add `\r` before and after the certificate. Take the following example to illustrate this:

`"-----BEGIN CERTIFICATE-----\r` followed by the **2048 bit self signed certificate** and `\r-----END CERTIFICATE-----"`

Provide the **2048 bit self signed certificate** in the **body** of the **New Client** call.

If the certificate, or one of the defined query parameters was invalid, the Bosch Smart Home Controller will respond with `400 bad request`.

### Encode your password to base64

In order for the request to be accepted, you have to encode your Bosch Smart Home Controller password into base64. If you need a hint how to do this, there are many base64 encoders online. Just pick one and encode your password there. For instance, the base64 encoded password for `my_passw0rd` is `bXlfcGFzc3cwcmQ=`.

Enter the information of the **base64 encoded password** in the **header** of the **New Client** call.

If the password is wrong, the Bosch Smart Home Controller will respond with `401 unauthorized`.
