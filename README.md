Table of contents
=================
**[500Portal Integration](#500portal-integration-)**

**[Sign up](#sign-up-)**
  
**[Sign in](#sign-in-)**
  
**[Get user profile](#get-user-profile-)**

**[Get all users based on particular domain id](#get-all-users-based-on-particular-domain-id-)**
  
**[Get all users based on particular domain id and user id](#get-all-users-based-on-particular-domain-id-and-user-id-)**

**[Get domain details](#get-domain-details-)**

**[Invite user/Add user](#invite-useradd-user-)**
  
**[Add a new product to existing user](#add-a-new-product-to-existing-user-)**  

**[9 dots](#9-dots-)**

**[Logout](#logout-)**


---------------  
# 500Portal integration :
500Portal is the main gateway for all the applications.All the apps should be integrated with 500portal to handle the sign up and sign in process. In addition,500 portal takes care of inviting users,adding products to the user,access for admin and users to subscribe/trial all the apps in the suite,billing for the selected apps,creating invoices for the subscriptions, able to pause/cancel any number of apps by both admin and user.

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
### Using curl

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

After clicking “Try for free” ,user will be redirected to 500Portal signup page and case-1 repeats.

## Case 4 - Customer signs up with google/facebook/linkedin/office365 from 500Portal(from 500Portal page) :
When clicks on any of the above social networking sites,in first step the authentication process takes place and then authorization takes place.After oauth process,the user will be taken to the 500Portal sign up page and needs to give his email address and clicks for the next page where he needs to enter his name,company name and select the type of industry his comapany is based on.

In this case,source by would be the selected social website and we doesn't show the password field and doesn't verify code when signing up from social websites.

After this,the user will be directly taken to 500Portal home page.

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

## Case 5 - Customer signs up with google/facebook/linkedin/office365 from other applications :
When clicks on any of the above social networking sites,in first step the authentication process takes place and then authorization takes place.After oauth process,the user will be taken to the particular home page.

In this case,source by would be the selected social website.

### dev/user/domain_user?source=google/facebook/linkedin/office
If the oauth is failed then no insertion takes place and it doesn't route to appside home url.

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

#### Case 1 - Customer clicks on login from 500Portal :
Here 500 portal shall validate the user credentials and when it logins successfully it sets the jwt as shown above and routes it to the 500portal home page. 
```sh
https://500appss.appup.cloud/dev/#/home
```
![alt text](https://github.com/LikhitaPulijala/500Portal/blob/master/img/Screenshot%20from%202019-05-10%2018-43-30.png)


#### Case 2 - Customer clicks on login from application :
When user clicks login from other application(examle supportly)then  500 portal shall validate the user credentials and when it logins successfully it sets the jwt as shown above and routes it to that application's home page. 

https://<<appname>>.appup.cloud/<<appname>>/#/home

![alt text](https://github.com/LikhitaPulijala/500Portal/blob/master/img/Screenshot%20from%202019-05-10%2018-43-30.png)


#### Case 3 - Customer login with google and facebook from 500portal :
When clicks on any of the above social networking sites from 500portal,in first step the authentication process takes place and then authorization takes place.After oauth process,500Portal verifies th email and if the email is already in database it redirects user to 500Portal home page.

#### Case 4 - Customer login with google and facebook from other applications :
When clicks on any of the above social networking sites from other applications,in first step the authentication process takes place and then authorization takes place.After oauth process,500Portal verifies th email and if the email is already in database it redirects user to that particular app home page.

### Get user profile :
This rest api shall be used to get a particular user profile of a given domain
```sh
curl https://500appss.appup.cloud/copy500apps/users/36?domain_id=1
```
## Returns: 

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

## Case 1: 500 portal is used to add the new user :	
Whenever the domain owner adds a new user to one or more products. The system shall do the necessary verification and setup on its end and may call the below rest api that is expected to be present on the app side. This is to give the app a hook to perform any actions needed for the new user.
The admin needs to provide user's name,email,choose the role,select for trial/subscribe and select the product that he wants to provide for the user.
## 500Portal user insert:

### dev/core/api/addTeamUser/domain_user/domain_user_role?email="user1@yopmail.com"

### Using curl

Method:POST
```sh
curl https://500appss.appup.cloud/dev/core/api/addTeamUser/domain_user/domain_user_role?email=user1@yopmail.com
-q "email: vintage1@yopmail.com"
-d "name: vintage1"
-d "email: vintage1@yopmail.com"
-d "role: user"
-d "products: 10"
-d "is_trial: 0"
https://<< appname >>.appup.cloud/<< appname >>/users
```
## Response:
- Status 200: A user is created succesfully for that admin.
- Status 404: The user given already existed.

## Appside insert:

### dev/core/api/appinsert

### Using curl

Method:POST
```sh
curl https://500appss.appup.cloud/dev/core/api/appinsert
-d "name: vintage1"
-d "email: vintage1@yopmail.com"
-d "role: user"
-d "products: 10"
-d "url: https://<< appname >>.appup.cloud/<< appname >>/adduser"
```
## Response:
- Status 200: When a appside insertion of user is successful.
- Status 400: When url at app is not correct.


## Case 2: Migration of users :	
This scenario happens when migrating customers and their users from AgileCRM to Supportly. 

A rest api on the 500portal side has to be called upon to add all the new users.

<< TBD >>

Assumption: The customer has already setup 500 portal account. 

## Case 3: Bulk invite of users :

This scenario comes when admin adds many number of users at a time by sending link to that user email address.

### Using curl

### dev/core/api/inviteuser

Method:POST

```sh
curl https://500appss.appup.cloud/dev/core/api/inviteuser
-d "email: user1@yopmail.com;user2@yopmail.com"
-d "message: test invite link"
-d "https://500appss.appup.cloud/dev/#/signup?ZG9tYWluX2lkPTM4NDgmaG91cnM9MTUmbWludXRlcz00JnNlY29uZHM9MTQmZGF0ZT0xMy8wNS8yMDE5"
```

# Add a new product to existing user :
This will be treated same as invite user/add user section. The below api on the app side shall be called upon for each product.Admin can add any number of products to the particular user.

Method:POST
```sh
curl https://<< appname >>.appup.cloud/<< appname >>/adduser https://500appss.appup.cloud/dev/core/api/addproduct/domain_user_role/3187
-d "name: vintage1"
-d "email: vintage1@yopmail.com"
-d "role: user"
-d "products: 8"
-d "domain_user_id: 3187"
-d "is_trial: 0"
```

# 9 dots :
To implement nine dots in your applications and get data of that user,your application should contain some workflows with rest node in it.
The below component must mention in your page to get info regarding nine dots.
```sh
<apps-launcher url="<<url to hit the defined workflow>>"/>
```

## a)Rest call to intimate inside the workflow
A workflow should be created at appside with a rest node in it.The token must pass in header.
### Using curl

Method:GET
```sh
curl {{{app.500_server}}}/core/api/getallmyapps?appname=yourapp
-H "token:jwt_token"
```
## b)Trigger at Appside
A trigger with this expression should be created at appside.
```sh
core/api/ninedots/{appname}
```

## c)Workflow at Appside for the trigger
A workflow should be created at appside with a rest node in it.The token must pass in header.
### Using curl

Method:GET
```sh
curl {{{app.500_server}}}/core/api/ninedots/{{{request.path.appname}}}?app=yourapp
-H "token:jwt_token"
```

## d)Trigger at Appside
A trigger with this expression should be created at appside.
```sh
core/api/product/{id}
```
## e)Workflow at Appside for the trigger
A workflow should be created at appside with a rest node in it.The token must pass in header.
### Using curl

Method:GET

```sh
curl {{app.500_server}}/core/api/product/{{request.path.id}}?app=yourapp
-H "token:jwt_token"
```

# Logout :
Remove token in the cookie and redirect to login page.
	







