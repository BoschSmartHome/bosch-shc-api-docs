# How to use the Postman Collection

To get started with the postman collection you need the following things:

- The Postman Collection 
- The IP of your Smart Home Controller (SHC) .
- A 2048 bit self signed certificate.
- The key to the certificate.

## Import certificate into Postman
1.  Open the **Postman Settings:**
![Postman Settings](images/postman_settings.png "Postman Settings")

2.  On tab **General**, disable **SSL certification verification**:
![Postman Disable SSL Verification](images/postman_disable_ssl_verification.png "Postman Disable SSL Verification")

3.  On tab **Certificates**, click on **Add Certificate**
![Postman Add Certificate](images/postman_add_certificate.png "Postman Add Certificate")

4. For Host enter the IP or hostname and the port (8443) of the SHC.
![Postman Add PEMs](images/postman_add_pems.png "Postman Add PEMs")

5. For **CRT file** select the provided certificate file
    For **KEY file** select the provided key  file
    For **Passphrase** enter the corresponding passphrase, if needed.
	
6. Click **Add** to add the certificate into your Postman-Settings.

7. Do the same for port 8444

Now you have all the prerequisites to communicate with the SHC. Start by adding a client.
