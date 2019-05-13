Table of contents
=================
**[500Portal Integration](#500portal-integration-)**

**[Sign up](#sign-up-)**
  
**[Sign in](#sign-in-)**
  
**[Get user profile](#get-user-profile-)**
  
**[Get all users](#get-all-users-)**  

**[Get domain details](#get-domain-details-)**

**[Invite user/Add user](#invite-useradd-user-)**
  
**[Add a new product to existing user](#add-a-new-product-to-existing-user-)**  

**[Remove Subscription](#remove-subscription-)**  

**[9 dots](#9-dots-)**

**[Logout](#logout-)**


---------------  
# 500portal integration :
All the apps should be integrated with 500portal to handle the sign up and sign in process. In addition,500 portal takes care of inviting users, billing, invoicing, cancellations, buy/cancel several apps.

This document talks about integrating other applications with 500Portal .  

# Sign up :  
Signup in 500Portal falls under 5 cases:
## Case 1 - Customer clicks on Sign up button from 500Portal page(without appname) :

When the user clicks sign up button,we take him to sign up page where he needs to give his email address and clicks for the next page where he needs to enter his name,company name,set a new password for his account and select the type of industry his comapany is based on.

In this case,the appname will be empty,source by default would be dashboard and a unique fingerprint value will be generated for every new user.

### dev/insert/signup/domain/domain_user?appname=&fingerprint=87c14e71d763fc527e78ac149c07f21a&source=dashboard
The email that entered should be valid one and the password that entered should be minimum of 8 characters.If the fields doesn't match the required validations,the user cannot be created.

### Using curl

Method: POST
```sh
curl https://500appss.appup.cloud/dev/insert/signup/domain/domain_user?appname=&fingerprint=87c14e71d763fc527e78ac149c07f21a&source=dashboard
-H "Access-Control-Allow-Origin:https://500appss.appup.cloud"
-H "Content-Type: application/json"
-d '{
    "name": "500Portal",
    "company": "Mantra Technologies",
    "industry": "Software",
    "password": "500Portal",
    "email": "500Portal@gmail.com",
    }' \
```
### Response:
- Status 200: User created successfully. And takes him to the verification page.
- Status 404: If the creation of the new user is failed.

If creation of user is succesful then a mail is send to the given address with a verification code in it and user is taken to the new page and asked to enter the verification code that is sent in the mail.
Once the entered code is verified and is successfull,then the user is taken to the home page. 
The home page url looks like as follows
```sh
https://500appss.appup.cloud/dev/#/home
```

## Case 2 - Customer clicks on Sign up button from application(with appname) :

When user opens an application(like supportly),and clicks on sign up button in that application,then user will be redirected to the 500Portal sign up page and the url looks like as follows:

```sh
https://500appss.appup.cloud/dev/#/signup?app=support
```

Now asks user to enter a valid email address,checks for the valid email or not.And if the email is a valid one then takes to the other page and asks to enter his name,company name,set a new password for his account and select the type of industry his comapany is based on.

In this case,the appname will be the name of the application,source by default would be dashboard and a unique fingerprint value will be generated for every new user.

The idea is once the 500 portal has successfully registered the user, the below url would be called upon to perform any default things on the app side for the new account.

The workflow has to set two values in headers to avoid CORS issue and then a successful routing happens. A 200 OK response has to be sent back after the default things are performed on app side.
 
Method:POST

```sh
curl https://<<appname>>.appup.cloud/<<appname>>/accountsignup 
-H "Accept : application/json"
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

Then the 500Portal shall route to the app’s home page as below
```sh
https://<<appname>>.appup.cloud/<<appname>>/#/home
```

## Case 3 - Customer enters email address and clicks “Try for Free” from application :
This is same as above except the email is included. This is needed with the assumption that every app would have a website as shown in the below where customer shall enter the email address directly to “Try for free”. 

![alt text](https://raw.githubusercontent.com/LikhitaPulijala/500Portal/master/img/Screenshot%20from%202019-05-10%2017-56-08.png)

Below is the sample url to test the sign up with email
https://500appss.appup.cloud/copy500apps/#/signup/collab2/take2gk@yopmail.com/2

## Case 4 - Customer signs up with google/facebook/linkedin/office365 from 500Portal(without appname) :
When clicks on any of the above social networking sites,in first step the authentication process takes place and then authorization takes place.After oauth process,the user will be taken to the 500Portal home page.

In this case,source by would be the selected social website.

### dev/user/domain_user?source=google/facebook/linkedin/office
If the oauth is failed then no insertion takes place and it doesn't route to his url

### Using curl

Method: POST
```sh
curl https://500appss.appup.cloud/dev/user/domain_user?source=facebook
-H "Access-Control-Allow-Origin:https://500appss.appup.cloud"
-H "Content-Type: application/json"
-d '{
    "name": "500Portal",
    "company": "Mantra Technologies",
    "industry": "Software",
    "password": "500Portal",
    "email": "500Portal@gmail.com",
    }' \
```
### Response:
- Status 200: User created successfully.
- Status 404: If the creation of the new user is failed.

## Case 5 - Customer signs up with google/facebook/linkedin/office365 from 500Portal(with appname) :
When clicks on any of the above social networking sites,in first step the authentication process takes place and then authorization takes place.After oauth process,the user will be taken to the 500Portal home page.

In this case,source by would be the selected social website.

### dev/user/domain_user?source=google/facebook/linkedin/office
If the oauth is failed then no insertion takes place and it doesn't route to his url

### Using curl

Method: POST
```sh
curl https://500appss.appup.cloud/dev/user/domain_user?source=facebook
-H "Access-Control-Allow-Origin:https://500appss.appup.cloud"
-H "Content-Type: application/json"
-d '{
    "name": "500Portal",
    "company": "Mantra Technologies",
    "industry": "Software",
    "password": "500Portal",
    "email": "500Portal@gmail.com",
    }' \
```
### Response:
- Status 200: User created successfully.
- Status 404: If the creation of the new user is failed.



## Case 6 - From "Bulk user invite" link :

When admin invites his users from "Bulk user invite" link,user gets a sign up link to his mail which admin had mentioned earlier.Once the user clicks the link,he will be redirected to 500Portal sign up page and case-1 repeats.


### Sign in : 

#### Case 1 - Customer clicks on login :
Here 500 portal shall validate the user credentials and when it logins successfully it sets the jwt as shown above and routes it to the home page. 
https://<<appname>>.appup.cloud/<<appname>>/#/home

![alt text](https://github.com/LikhitaPulijala/500Portal/blob/master/img/Screenshot%20from%202019-05-10%2018-43-30.png)

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
# Get all users based on particular domain id :

This rest api is used to get all users information based on given domain id.

Method: GET
```sh
curl https://500appss.appup.cloud/dev/users?domain_id=1
```
## Returns:

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

## Response:
- Status 200: A JSON array with all user information and their related product/roles would be returned.
- Status 404: The domain_id may not exists.

# Get all users based on particular domain id and user id :

This rest api is used to get the particular user information based on given domain id and user id.

Method: GET
```sh
curl https://my.500apps.com/users/2102?domain_id=1570
```
## Returns:

```javascript
{
"id": 2102,
"domain_id": 1570,
"user_id": 2102,
"name": "Gbnreddy",
"email": "gbnreddy@yopmail.com",
"language": "1",
"logo_url": "https://500-apps.s3.us-west-1.amazonaws.com/4674134-Anonymous-Quote-Ignore-the-noise-focus-on-your-work.jpg",
"phone": "8919407439",
"gender": "male",
"timezone": "Universal Coordinated Time(GMT)",
"currency": "Canadian Dollar",
"currency_symbol": "CAD",
"created_by": 12345,
"created_date": "Apr 23, 2019 2:03:07 AM",
"products": [
{
"role": "admin",
"product": "build"
},
{
"role": "admin",
"product": "finder"
},
{
"role": "admin",
"product": "recruit"
},
{
"role": "admin",
"product": "sign"
},
{
"role": "admin",
"product": "flow"
},
{
"role": "admin",
"product": "track"
},
{
"role": "admin",
"product": "schedule"
},
{
"role": "admin",
"product": "hipsocial"
},
{
"role": "admin",
"product": "support"
},
{
"role": "admin",
"product": "collab"
},
{
"role": "admin",
"product": "agilecrm"
},
{
"role": "admin",
"product": "clickdesk"
}
]
}
```

## Response:
- Status 200: A JSON array with particular user information and his related product/roles would be returned.
- Status 404: The domain_id/user id may not exists.

	
# Invite user/Add user : 

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
	







