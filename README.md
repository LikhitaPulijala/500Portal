Table of contents
=================
**[500Portal Integration](#500portal-integration-)**

**[Sign up](#sign-up-)**
  * [Case 1 - Customer clicks on Sign up button](#case-1---customer-clicks-on-sign-up-button-)
  * [Case 2 - Customer enters email address and clicks “Try for Free”](#case-2---customer-enters-email-address-and-clicks-try-for-free-)
  * [Case 3 - Customer signs up with google or facebook](#case-3---customer-signs-up-with-google-or-facebook-#sign-in-)
 
**[Sign in](#sign-in-)**
  * [Case 1 - Customer clicks on login](#case-1---customer-clicks-on-login-)
  * [Case 2 - Customer login with google and facebook](#case-2---customer-login-with-google-and-facebook-)
  
**[Get user profile](#get-user-profile-)**
  
**[Get all users](#get-all-users-)**  

**[Get domain details](#get-domain-details-)**

**[Invite user/Add user](#invite-useradd-user-)**
  * [Case 1: 500 portal is used to add the new user](#case-1-500-portal-is-used-to-add-the-new-user-)
  * [Case 2: Migration of users](#case-2-migration-of-users-)
  * [Case 3: Bulk invite of users](#case-3-bulk-invite-of-users-)
  
**[Add a new product to existing user](#add-a-new-product-to-existing-user-)**  

**[Remove Subscription](#remove-subscription-)**  

**[9 dots](#9-dots-)**

**[Logout](#logout-)**


---------------  
### 500portal integration :
All the apps should be integrated with 500portal to handle the sign up and sign in process. In addition,500 portal takes care of inviting users, billing, invoicing, cancellations, buy/cancel several apps.

This document talks about integrating other applications with 500Portal .  

### Sign up :  

#### Case 1 - Customer clicks on Sign up button :
There should be a workflow with below trigger expression. The idea is once the 500 portal has successfully registered the user, the below url would be called upon to perform any default things on the app side for the new account.

The workflow has to set two values in headers to avoid CORS issue and then a successful routing happens. A 200 OK response has to be sent back after the default things are performed on app side.
 
Method:POST

```sh
curl https://<<appname>>.appup.cloud/<<appname>>/accountsignup 
-H "Access-Control-Allow-Origin:https://500appss.appup.cloud"
-H "Access-Control-Allow-Credentials:true"
```
Once the sign up process is done, a jwt token shall be set with payload data
```javascript
{
  "sub": "jwt",
  "email": "take1gk@yopmail.com",
  "tenant_id": "1077",
  "user_id": "1131"
}
```
email - the new customer’s email address. 

tenant_id - the account id of the customer account in 500 portal.

user_id - the user id of the customer account in 500 portal

Then the 500portal shall route to the app’s home page as below
https://<<appname>>.appup.cloud/<<appname>>/#/home

Below is the sample url to test the sign up with no email
https://500appss.appup.cloud/copy500apps/#/signup/collab2/:value/1

#### Case 2 - Customer enters email address and clicks “Try for Free” :
This is same as above except the email is included. This is needed with the assumption that every app would have a website as shown in the below where customer shall enter the email address directly to “Try for free”. 

![alt text](https://raw.githubusercontent.com/username/projectname/branch/path/to/img.png)

Below is the sample url to test the sign up with email
https://500appss.appup.cloud/copy500apps/#/signup/collab2/take2gk@yopmail.com/2

#### Case 3 - Customer signs up with google or facebook :
Below is the sample url to test the sign up with google oauth
https://500appss.appup.cloud/copy500apps/#/signup/collab2/:value/google


### Sign in : 

#### Case 1 - Customer clicks on login :
Here 500 portal shall validate the user credentials and when it logins successfully it sets the jwt as shown above and routes it to the home page. 
https://<<appname>>.appup.cloud/<<appname>>/#/home



Below is the sample url to test the sign in with email
https://500appss.appup.cloud/copy500apps/#/signin/collab2

#### Case 2 - Customer login with google and facebook :
Coming soon

### Get user profile :
This rest api shall be used to get a particular user profile of a given domain
```sh
curl https://500appss.appup.cloud/copy500apps/users/36?domain_id=1
```
RETURNS 

A JSON Object with his profile info and product roles
```javascript
{
    "domain_id": 1,
    "name": "sdf",
    "email": "asd@yopmail.com",
    "product_name": [
        {
            "name": "Finder.io",
            "role": "user"
        },
        {
            "name": "recruit",
            "role": "user"
        }
    ]
}
```
### Get all users :
This rest api shall be called to get all users info for a given domain id.

Method: GET
```sh
curl https://500appss.appup.cloud/copy500apps/users?domain_id=1
```
RETURNS

A JSON array with all user info and their related product/roles would be returned with 200 OK Response.

```javascript
[
    {
        "user_id": 1,
        "name": "Insane Acers",
        "email": "socialdev@gmail.com",
        "created_by": 12345,
        "created_date": "Jan 16, 2019 2:59:27 PM",
        "product_name": [
            {
                "role": "user",
                "product": "Contacts"
            }
        ]
    },
    {
        "user_id": 4,
        "name": "name123",
        "email": "email233@gmail.com",
        "created_by": 12345,
        "created_date": "Jan 16, 2019 3:02:58 PM",
        "product_name": [
            {
                "role": "manager",
                "product": "Build.ly"
            },
            {
                "role": "manager",
                "product": "recruit"
            },
            {
                "role": "manager",
                "product": "deals"
            }
        ]
    }
]
```
### Get domain details :
This api shall be used to get the domain details, like who is the domain owner and when was it opened.

<< TBD >>
	
### Invite user/Add user : 

#### Case 1: 500 portal is used to add the new user :	
Whenever the domain owner adds a new user to one or more products. The system shall do the necessary verification and setup on its end and may call the below rest api that is expected to be present on the app side. This is to give the app a hook to perform any actions needed for the new user.

https://<< appname >>.appup.cloud/<< appname >>/users for each product with POST method.
	
#### Case 2: Migration of users :	
This scenario happens when migrating customers and their users from AgileCRM to Supportly. 

A rest api on the 500portal side has to be called upon to add all the new users.

<< TBD >>

Assumption: The customer has already setup 500 portal account. 

#### Case 3: Bulk invite of users :

Method:Post

```sh
curl https://{{{app.500_server}}}/core/api/inviteuser
```
Body:
```javascript
email: user1@yopmail.com;user2@yopmail.com
message: test invite link
invite_link: https://{{{app.500_server}}}/#/signup?ZG9tYWluX2lkPTE0NDEmaG91cnM9MTYmbWludXRlcz0yOSZzZWNvbmRzPTcmZGF0ZT0yNS8wNC8yMDE5&app={{appname}}
```
Invite_link is generated in js

var d = new Date();

var hours = d.getHours();

var minutes = d.getMinutes();

var seconds = d.getSeconds();

var date = new Date().toLocaleString().split(',')[0];

var params = "domain_id="+this.domain+"&hours="+hours+"&minutes="+minutes+"&seconds="+seconds+"&date="+date;

var b = btoa(params);

 https://{{{app.500_server}}}/#/signup?b&app={{appname}}

### Add a new product to existing user :
This will be treated same as invite user/add user section. The below api on the app side shall be called upon for each product.

Method:POST
```sh
https://<< appname >>.appup.cloud/<< appname >>/adduser
```

### Remove Subscription :
When the domain owner removes an user to one or more products. The system shall perform necessary business validations on its end and may call the below rest api on the app side for each product.

Method:DELETE

```sh

https://<< appname >>.appup.cloud/<< appname >>/removeuser
```

### 9 dots :
There should be a workflow with a rest node in it.

<apps-launcher url="<<url to hit the defined workflow>>"/>

#### a)Rest call to intimate inside the workflow

Method:GET;
```sh
curl {{{app.500_server}}}/core/api/getallmyapps?appname=yourapp
-H "token:jwt_token"
```
#### b)Trigger at Appside
core/api/ninedots/{appname}

#### c)Workflow at Appside for the trigger
Method:GET;
```sh
curl {{{app.500_server}}}/core/api/ninedots/{{{request.path.appname}}}?app=yourapp
-H "token:jwt_token"
```
#### d)Trigger at Appside
core/api/product/{id}

#### e)Workflow at Appside for the trigger
Method:GET

```sh
curl {{app.500_server}}/core/api/product/{{request.path.id}}?app=yourapp
-H "token:jwt_token"
```

### Logout :
Remove token in the cookie and redirect to login page.
	







