---
title: "Azure AD: Restrict Application Access To Users Belonging To A Group"
comments: true
date: 2019-06-04
categories: 
- Azure
---

For one of the web application I was working on, access was to be restricted based on user belonging to a particular Azure AD Group. The application as such did not have any Role Based Functionality. It feels an overhead to set up the [Role Based Access](https://www.rahulpnath.com/blog/dot-net-core-api-and-azure-ad-groups-based-access/) when all we want is to restrict users belonging to a particular group. 

In this post, let us look at how we can set up Azure AD authentication such that only users of a particular group can authenticate against it and get a token. I used the [ASP.NET Core 2.1 Azure AD authentication sample](https://github.com/juunas11/aspnetcore2aadauth), but this applies to any application trying to get a token from an Azure AD application. 

### Setting up Azure AD Application

In the Azure Portal, navigate to the AD application used for authentication, under Enterprise Applications (from Azure Active Directory). Turn on 'User Assignment Required' under Properties for the application (as shown below).

> With User Assignment Required turned on, users must first be assigned to this application before being able to access it. When this is not turned on, any user in the directory can access the application.

![](/images/azure_ad_user_assignment.jpg)

Once this is turned on if you try to access your application, it will throw an error indicating the user is not assigned to a role for the application.

![](/images/azure_ad_user_role_error.jpg)

### Adding User to Role

For users to now be able to access the AD application, they need to be added explicitly to the application. This can be done using the Azure Portal under the 'Users and groups' section for the AD application (as shown below).

![](/images/azure_ad_user_role_add.jpg)

Users can either be added explicitly to the application or via Groups. Adding a group grants access to all the users within the group. In our scenario, we created an AD group for the application and added users that can access the application to the group. If you have different AD applications per environment (which you should), make sure you do this for all the applications.

### Handling Managed Service Identity Service Principal

Even though it is possible to add MSI service principal to an Azure AD Group it does not work as intended. The request was failing to get a token with the error that the user is not assigned to a role. It looks like this is one of the cases where a full service principal is required.

To get this working for an MSI service principal, I had to create a dummy application role for the AD application and grant the MSI service principal that role for the AD application. Check the [Using AD Role section in this article](https://www.rahulpnath.com/blog/how-to-authenticate-azure-function-with-azure-web-app-using-managed-service-identity/#using-ad-role) for full details on setting this up. Note that in this case, you need to explicitly add in the application roles and grant access for the service principal for each of them.


Only users belonging to the group or have been assigned directly to the AD application can get a token for the AD application and hence access the Web application. This is particularly useful when the application does not have any Role Based functionality and all you want is restrict access to a certain group of people within your directory/organization.