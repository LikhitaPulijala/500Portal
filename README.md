Table of contents
=================
**[Sign up](#sign-up)**
  * [Case 1 - Customer clicks on Sign up button](https://github.com/agilecrm/python-api)
  * [Case 2 - Customer enters email address and clicks “Try for Free”](https://github.com/agilecrm/python-api)
  * [Case 3 - Customer signs up with google or facebook](https://github.com/agilecrm/php-api)
 
**[Sign in](#)**
  * [Case 1 - Customer clicks on login](https://github.com/agilecrm/python-api)
  * [Case 2 - Customer login with google and facebook](https://github.com/agilecrm/python-api)
  
**[Get user profile](#)**
  
**[Get all users](#)**  

**[Get domain details](#)**

**[Invite user/Add user](#)**
  * [Case 1: 500 portal is used to add the new user](https://github.com/agilecrm/python-api)
  * [Case 2: Migration of users](https://github.com/agilecrm/python-api)
  
**[Add a new product to existing user](#)**  

**[Remove Subscription](#)**  

**[9 dots](#)**

Things to know:
---------------  
### 500 portal integration :
All the apps has to be integrated with 500 portal to handle the sign up and sign in process. In addition 500 portal takes care of inviting users, billing, invoicing, cancellations, buy/cancel several apps.

This document talks about integrating various  

### Sign up :  

#### Case 1 - Customer clicks on Sign up button :
There should be a workflow with below trigger expression. The idea is once the 500 portal has successfully registered the user, the below url would be called upon to perform any default things on the app side for the new account.
 
https://<<appname>>.appup.cloud/<<appname>>/accountsignup with POST method


The workflow has to set two headers to avoid CORS issue and successful routing happens. A 200 OK response has to be sent back after the default things are performed on app side

Access-Control-Allow-Origin:https://500appss.appup.cloud
Access-Control-Allow-Credentials:true

Once the sign up process is done, A jwt token shall be set with payload data 

{
  "sub": "jwt",
  "email": "take1gk@yopmail.com",
  "tenant_id": "1077",
  "user_id": "1131"
}

email - the new customer’s email address. 
tenant_id - the account id of the customer account in 500 portal
user_id - the user id of the customer account in 500 portal

The 500 portal shall route to the app’s home page as below

https://<<appname>>.appup.cloud/<<appname>>/#/home

Below url is sample to test the sign up with no email
https://500appss.appup.cloud/copy500apps/#/signup/collab2/:value/1

#### Case 2 - Customer enters email address and clicks “Try for Free” :
This is same as above except the email is included. This is needed with the assumption that every app would have a website as shown in the below where customer shall enter the email address directly to “Try for free”. 




Below url is sample to test the sign up with email
https://500appss.appup.cloud/copy500apps/#/signup/collab2/take2gk@yopmail.com/2

#### Case 3 - Customer signs up with google or facebook :
Below url is sample to test the sign up with google oauth
https://500appss.appup.cloud/copy500apps/#/signup/collab2/:value/google


### Sign in : 

#### Case 1 - Customer clicks on login :
Here 500 portal shall validate the user credentials and when successful shall set the jwt as detailed above and route it to home page. 
https://<<appname>>.appup.cloud/<<appname>>/#/home



Below url is sample to test the sign up with email
https://500appss.appup.cloud/copy500apps/#/signin/collab2

#### Case 2 - Customer login with google and facebook :
Coming soon

### Get user profile :
This rest api shall be used to get a particular user profile of a given domain
https://500appss.appup.cloud/copy500apps/users/36?domain_id=1

RETURNS 

A JSON Object with his profile info and product roles

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

### Get all users :
This rest api shall be called to get all users info for a given domain id.

Method: GET
https://500appss.appup.cloud/copy500apps/users?domain_id=1

RETURNS

A JSON Array with all user info and their related product/roles would be returned with 200 Ok Response


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

### Get domain details :
This api shall be used to get the domain details, like who is the domain owner and when was it opened.

<<TBD>>
	
### Invite user/Add user : 

#### Case 1: 500 portal is used to add the new user :	
Whenever the domain owner adds a new user to one or more products. The system shall do the necessary verification and setup on its end and may call the below rest api that is expected to be present on the app side. This is to give the app a hook to perform any actions needed for the new user.

https://<<appname>>.appup.cloud/<<appname>>/users for each product with POST method.
	
#### Case 2: Migration of users :	
This scenario happens for example when migrating customers and their users from AgileCRM to Supportly. 

A rest api on the 500 portal side has to be called upon to add all the new users.

<<TBD>>

Assumption: The customer has already setup 500 portal account. 

#### Case 3: Bulk invite of users :
https://{{{app.500_server}}}/core/api/inviteuser
Method:post
bosy:
email: user1@yopmail.com;user2@yopmail.com
message: test invite link
invite_link: https://{{{app.500_server}}}/#/signup?ZG9tYWluX2lkPTE0NDEmaG91cnM9MTYmbWludXRlcz0yOSZzZWNvbmRzPTcmZGF0ZT0yNS8wNC8yMDE5&app={{appname}}

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
This will be treated same as invite user/add user section. The below api on the app side shall be called upon 
https://<<appname>>.appup.cloud/<<appname>>/adduser for each product with POST method.

### Remove Subscription :
When the domain owner removes an user to one or more products. The system shall perform necessary business validations on its end and may call the below rest api on the app side

https://<< appname >>.appup.cloud/<< appname >>/removeuser for each product with DELETE method.

### 9 dots :
There should be a three workflow with a rest node in it.

<apps-launcher url="<<url to hit the defined workflow>>"/>

#### a)Rest call to intimate inside the workflow
----------------------------------------------------- 
restcall:{{{app.500_server}}}/core/api/getallmyapps?appname=yourapp;
method:get;
headers:token:jwt_token;

#### b)Trigger at Appside
-------------------------
core/api/ninedots/{appname}

#### c)Workflow at Appside for the trigger
----------------------------------------------
restcall:{{{app.500_server}}}/core/api/ninedots/{{{request.path.appname}}}?app=yourapp;
method:get;
headers:token:jwt_token;

#### d)Trigger at Appside
------------------------
core/api/product/{id}

#### e)Workflow at Appside for the trigger
----------------------------------------------
restcall:{{app.500_server}}/core/api/product/{{request.path.id}}?app=yourapp;
method:get;
headers:token:jwt_token;


### Logout :
Remove token in the cookie and redirect to login page.
	







