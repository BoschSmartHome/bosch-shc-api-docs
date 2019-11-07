# How to use the Bosch Smart Home Postman Collection

To get started with the Postman Collection you need the following:

- Postman Collection downloaded to your local environment
- The IP address of your Smart Home Controller (SHC)
- A generated 2048 bit self signed certificate
- The key to that certificate

## Import the certificate into Postman
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

Now, you have all the prerequisites to communicate with your SHC via Postman. Start by adding a client.
