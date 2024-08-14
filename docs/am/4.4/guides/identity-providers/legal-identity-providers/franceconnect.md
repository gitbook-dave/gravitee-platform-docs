# FranceConnect

## Overview

You can authenticate users in AM with [FranceConnect](https://franceconnect.gouv.fr/). FranceConnect is the French government Identity Provider that connects millions of legal accounts. You can connect to it with credentials such as your National Insurance Number, postal address, and more.

<figure><img src="https://docs.gravitee.io/images/am/current/graviteeio-am-userguide-legal-franceconnect-logo.png" alt=""><figcaption><p>FranceConnect logo</p></figcaption></figure>

Before you begin, you need to sign up for a [FranceConnect account](https://partenaires.franceconnect.gouv.fr/).

## Steps

To connect your application to FranceConnect, you will:

* Register a new application in FranceConnect
* Create a FranceConnect identity provider in AM
* Set up the connection in FranceConnect
* Test the connection

## Register a new application in FranceConnect

To connect your application to FranceConnect, you must follow all the steps described [here](https://franceconnect.gouv.fr/partenaires).

{% hint style="info" %}
FranceConnect will generate a client\_ID and client\_secret. Make a note of these for later use.
{% endhint %}

## Create a FranceConnect identity provider

1. Log in to AM Console.
2. Click **Settings > Providers**.
3. Click the plus icon ![plus icon](https://docs.gravitee.io/images/icons/plus-icon.png).
4. Select **FranceConnect** as your identity provider type and click **Next**.

{% hint style="info" %}
Ensure you have the Client ID and Client Secret generated by FranceConnect to hand.
{% endhint %}

5. Give your identity provider a name.
6. Enter your FranceConnect Client ID and Client Secret.
7. Select at least the **openid** scope, which is mandatory.
8.  Click **Create**.

    <figure><img src="https://docs.gravitee.io/images/am/current/graviteeio-am-userguide-legal-idp-franceconnect.png" alt=""><figcaption><p>Create FranceConnect IdP</p></figcaption></figure>

{% hint style="info" %}
Copy the URL in **1. Configure the Redirect URI** to the right of the page. You need this value to update your FranceConnect application settings in the next section.
{% endhint %}

## Set up the connection

1. Go to your FranceConnect application settings and click **Add a Redirect URI**.
2. Enter the Redirect URI value you copied in the previous section.

## Test the connection

You can test your FranceConnect connection using a web application created in AM.

1.  In AM Console, click **Applications** and select your legal identity provider.

    <figure><img src="https://docs.gravitee.io/images/am/current/graviteeio-am-userguide-social-idp-list.png" alt=""><figcaption><p>Select FraceConnect IdP</p></figcaption></figure>
2.  Call the Login page (the `/oauth/authorize` endpoint). If your connection is working you will see a **Sign in with** button.

    If you do not see the button, there may be a problem with the identity provider settings. Check the AM Gateway log for more information.

    <figure><img src="https://docs.gravitee.io/images/am/current/graviteeio-am-userguide-social-idp-login.png" alt=""><figcaption><p>Sign in options</p></figcaption></figure>

{% hint style="info" %}
The [FranceConnect frequently asked questions (FAQ)](https://partenaires.franceconnect.gouv.fr/faq) can help you to set up your connection.
{% endhint %}