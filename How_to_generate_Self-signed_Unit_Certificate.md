# How to generate Self-signed Unit Certificate

-------------------------------------------------

## Introduction

This is a manual for the procedure to generate the Self-signed unit certificate, which is required for the ansible execution to build Personium Unit.  
Followings will be created by openssl, after performing the procedure below.

| File name | Explanation |
|---|---|
|unit.key             |This is a unit secret key. Created by RSA secret key of more than 2048bit in PEM format. Managing this unit secret key strictly is highly recommended.|
|unit.csr             |Request for X.509 certificate. This file will be required to create the certificate and not be deployed on the server. |
|unit-self-sign.crt   |It is a PEM format certificate supporting Unit Key. The value of CN should be the FQDN of `web` server. |

Now you can get these certificate and key! Below is the procedure to generate the Self-signed unit certificate.

---------------------------------------

### Part 1. Create Personium Unit secret key (unit.key) on Bastion server

1. Change directory to the certificate deployment directory of Bastion server.

    ```console
    # cd $ansible/resource/ap/opt/x509/
    ```

1. Create secret key

    ```console
    # openssl genrsa -out unit.key 2048
    ```  

    **Example:)**

    ```console
    # openssl genrsa -out unit.key 2048
    Generating RSA private key, 2048 bit long modulus
    ............................................................+++
    ...................................................+++
    e is 65537 (0x10001)
    -----------------------------------------------------------------------------------
    ```

1. Confirm that the unit.key is created

    ```console
    # ls -l
    ```

    **Example:)**

    ```console
    # ls
    total 4
    -rw-r--r--. 1 root root 1675 Sep  1 20:27 unit.key
    ```

### Part 2. Create self-signed unit certificate

1. Use the openssl command to create a Certificate Signing Request file.  
    \* The configuration procedure is interactive. Please use the FQDN of your Personium Unit for the value of "Common Name". (Required)

    ```console
    # openssl req -new -key unit.key -out unit.csr
    ```

    **Example:)**  
    \* In the case when the FQDN of the Personium Unit is "personium.example.com"

    ```console
    # openssl req -new -key unit.key -out unit.csr
    You are about to be asked to enter information that will be incorporated
    into your certificate request.
    What you are about to enter is what is called a Distinguished Name or a DN.
    There are quite a few fields but you can leave some blank
    For some fields there will be a default value,
    If you enter '.', the field will be left blank.
    -----
    Country Name (2 letter code) [XX]:
    State or Province Name (full name) []:
    Locality Name (eg, city) [Default City]:
    Organization Name (eg, company) [Default Company Ltd]:
    Organizational Unit Name (eg, section) []:
    Common Name (eg, your name or your server's hostname) []:personium.example.com            <* Enter the unit domain name (required)
    Email Address []:
    Please enter the following 'extra' attributes
    to be sent with your certificate request
    A challenge password []:
    An optional company name []:

    ```

1. Self-sign the CSR. valid for 3650 days (approximately 10 years)


    ```console
    # openssl x509 -req -days 3650 -signkey unit.key -out unit-self-sign.crt <unit.csr
    ```

    **Example:)**

    ```console
    # openssl x509 -req -days 3650 -signkey unit.key -out unit-self-sign.crt <unit.csr
    Signature ok
    subject=/C=XX/L=Default City/O=Default Company Ltd/CN=example.com
    Getting Private key

    ```

1. Check the contents of the generated certificate

    ```console
    # openssl x509 -in unit-self-sign.crt -text
    ```  
    \* Make sure the CN matches the FQDN of the Personium Unit.

    Also check with the following command

    ```console
    # ls -l
    ```   

    **Example:)**

    ```console
    # ls -l
    total 12
    -rw-r--r--. 1 root root 1041 Sep  1 20:29 unit.csr
    -rw-r--r--. 1 root root 1675 Sep  1 20:27 unit.key
    -rw-r--r--. 1 root root 1277 Sep  1 20:29 unit-self-sign.crt
    ```

### Summary

Though there are several option to generate the certificate, this time we generate it by self-signing.
Generally, self-signed unit certificate can't be used in public, so this procedure should be used only in preparing private Personium Unit.
