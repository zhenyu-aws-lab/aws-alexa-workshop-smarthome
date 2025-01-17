# Alexa Account Linking using Amazon Cognito User Pool

This article describes how to create **Account Linking** between developer's account 
system and Amazon account system. In this case, Amazon Cognito User Pool will be
used as the developer's account system. By using it, developer can quickly create 
Account Linking between their account and Amazon account without writing one line of code. 

For more information about Amazon Cognito User Pool, please refer to the 
[developer guide](https://docs.aws.amazon.com/cognito/latest/developerguide/cognito-user-identity-pools.html).

## What is Account Linking

The following is an explanation from [Alexa Docs](https://developer.amazon.com/docs/account-linking/understand-account-linking.html#account-linking-and-the-skill-model)

> You would use account linking if your skill needs personalized data from another system. 
> For example, suppose you own a web-based service "Car Fu" that lets users order taxis. 
> A custom skill that lets users access "Car Fu" by voice would be very useful. For example, 
> "Alexa, ask "Car Fu" to order a taxi." Completing this request requires the skill to access 
> your "Car Fu" service as a specific "Car Fu" user for profile and payment information. Therefore, 
> you need a link between the Amazon account used with the Alexa device and the "Car Fu" account 
> for the user.

Account linking in the Alexa Skills Kit uses [OAuth 2.0](https://tools.ietf.org/html/rfc6749). 
The following diagram explains the flow of obtain an **AccessToken** from your OAuth2.0 system.

![Auth-Code-Flow](assets/auth-code-flow.png)

Alexa will send all the subsequent directives together with **AccessToken**. In the Lambda
backend, the program should verify and decode the **AccessToken** to get user related information.

![Flow](assets/skill-interaction-flow.png)

## Configure App Client OAuth 2.0 Settings

By default, OAuth 2.0 for App Client in Cognito User Pool is not enabled. Follow the following
step to enable Auth2.0. 

1. Go to Cognito User Pool Console

1. On the left side navigation bar, under App integration, select **App client settings**

1. Find the App Client created in [Create a Cognito User Pool Client](#create-a-cognito-user-pool-client), if you
followed the guide, it should be named `alexa`

1. Under **Enabled Identity Providers**，select **Cognito User Pool**

1. In **Callback URL(s)**，enter **Redirect URLs** copied from Alexa Developer Console. In Alexa Developer Console,
choose **Account Linking**, scroll down to the bottom, you should be able to find three **Redirect URL**. Alexa redirect 
to different url based on user's region. To serve all the Alexa users, it is suggested to copy all the URLs.

1. In **Allowed OAuth Flows** session，choose **Authorization code grant**

1. In **Allowed OAuth Scopes** session，choose **openid**

1. Click **Save changes**

![](assets/configure-cup-oauth.png)


## Configure Cognito User Pool domain name

The default domain name of Cognito follows the pattern `https://<domain-prefix>.auth.<region>.amazoncognito.com`。
You can use your own domain，to get more information please refer to 
[Adding a Custom Domain to a User Pool](https://docs.aws.amazon.com/cognito/latest/developerguide/cognito-user-pools-add-custom-domain.html#cognito-user-pools-add-custom-domain-adding)。
In this lab, we will use the default domain name.

1. Go to Cognito User Pool Console

1. On the left side bar, under **App integration**, choose **Domain name**

1. Enter domain prefix and click **Check availability**, the domain name must be unique

1. When promoted **the domain is available**，choose **Save changes**

![](assets/cognito-domain.png)


## Configure Account Linking in Alexa Developer Console

1. Go to [Alexa Console](https://developer.amazon.com/alexa/console/ask)

1. In the **Skills** list，choose the previously created skill

1. On the left side navigation bar，choose **Account Linking**

1. Under **Security Provider Information**，choose **Auth Code Grant**

1. Enter `https://<your-cognito-domain>/oauth2/authorize` in **Authorization URI**

1. Enter `https://<your-cognito-domain>/oauth2/token` in **Access Token URI**

1. Enter **Client ID** and **Client Secret**, you can find in them in Cognito User Pool console, under **App Clients** section
![](assets/find-client-credentials.png)

1. Click **Add scope** and input `openid`. For Smart Home skill, at least one scope should be specified

1. Click **Save** on the top right corner
![](assets/configure-alexa-account-linking.png)

For more about Cognito OAuth2.0 URI, please refer to 
[Amazon Cognito User Pools Auth API Reference](https://docs.aws.amazon.com/cognito/latest/developerguide/cognito-userpools-server-contract-reference.html)

## Enable Account Linking in Alexa App

> Only the Developer Account can see the Skill in development. You must use the same account as creating
> the skill. You may also need a VPN to use the Alexa APP if you are in China.

1. Launch Alexa APP on mobile phone

1. Click the button on the top left corner

1. Choose **Skills & Games** 

1. On **Skills & Games** page，click the **Enabled** dropdown list and choose **DEV**, you should be able
the created Smart Home Skill

1. Click the **Enable To Use** button, 

1. On the popup window, input your **email** and **password**, if have not registered yet, sign up one

1. Account Link success
![](assets/account-linking-success.jpeg)

So far, the account linking between Alexa and Cognito User Pool has been configured successfully. In 
the following directives sending from Alexa, it will contain **accessToken** in the message body. 
The **accessToken** follows the JWT spec. In the backend Lambda, you can verify and decode the JWT token
to get the user identity.

## Reference
[Understand Account Linking](https://developer.amazon.com/docs/account-linking/understand-account-linking.html)

[The OAuth2.0 Authorization Framework](https://tools.ietf.org/html/rfc6749)

[JSON Web Token](https://en.wikipedia.org/wiki/JSON_Web_Token)

[AWS Cognito User Pool](https://docs.aws.amazon.com/zh_cn/cognito/latest/developerguide/cognito-user-identity-pools.html)
