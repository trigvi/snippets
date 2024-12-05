# Generating MFA/2FA authentication codes on Linux Debian

These are the steps for creating a Linux Debian bash script for generating multi-factor authentication codes like you usually do with a smartphone app like Google Authenticator.

The idea is that every time you register for a website that supports multi-factor (2FA) authentication, the website gives you a unique key both in text and QR code format. In the script below, simply add the website name to the *options* section and also create a new *elif* block which contains the text unique key you got given by the website.

Whenever you want to generate an authentication code for that website, run the script.

### How to create it

* Install the *oathtool* package
    ```
    sudo apt install oathtool -y
    ```

* Create an empty bash script called *mfa_generator.sh*, give it strict permissions to prevent other users from viewing it.
    ```
    touch mfa_generator.sh
    chmod 700 mfa_generator.sh
    ```

* Open *mfa_generator.sh*, insert the following sample contents and modify accordingly.
    ```
    #!/bin/bash
    options="Aws Binance Godaddy"
    select opt in $options; do
    if [ "$opt" = "Aws"  ]; then
        oathtool --base32 --totp "YOURUNIQUEAWSKEY" -d 6
    elif [ "$opt" = "Binance" ]; then
        oathtool --base32 --totp "YOURUNIQUEBINANCEKEY" -d 6
    elif [ "$opt" = "Godaddy" ]; then
        oathtool --base32 --totp "YOURUNIQUEGODADDYKEY" -d 6
    fi
    exit
    done
    ```

* Run the script to generate an authentication code:
    ```
    ./mfa_generator.sh
    ```
