# Prerequisites

- Prepare EntraID(Azure AD) environment
  - You can use Microsoft single user account(No groups will be availiable in IdP) or register organisation account
- Prepare AWS environment
  - Create AWS Organization and have some child accounts (Recommended to use Control Tower service)
  - Use **eu-central-1** region (you can use any other, as well, with conditions that all settings will done only in AWS Organisation home region)

# Procedure
## Setup SSO authentification (SAML)
### Init AWS Identity Center to use external IdP
1. AWS > IAM Identity Center > Enable IAM Identity Center > Enable
2. AWS > IAM Identity Center > Settings > Identity source > Actions > Change identity source > Configure external identity provider
3. AWS > ... > Configure external identity provider > Download metadata file [metadata file 1]
### Init EntraId enterprise application for SSO portal
5. Entra ID > Enterprise applications > + New applications > "AWS IAM Identity Center (successor to AWS Single Sign-On)
6. Amazon Web Services, Inc." > Set Name [SSO App] > Create
7. Entra ID > ... > [SSO App] > Single Sign-On > SAML > Upload metadata file > [metadata file 1]
8. Entra ID > ... > [SSO App] > ... > Basic SAML Configuration > Identifier  = [identifier from metadata file 1] ex.: `https://eu-central-1.signin.aws.amazon.com/platform/saml/d-9967708aaa`
9. Entra ID > ... > [SSO App] > ... > Basic SAML Configuration > Reply URL   = [reply url from metadata file 1] ex.: `https://eu-central-1.signin.aws.amazon.com/platform/saml/acs/8c92a9f6-2912-4676-bx01-6dz705fb23aa`
10. Entra ID > ... > [SSO App] > Single Sign-On > SAML Certificates > Certificate (Base64) > Download > [cert file 1]
11. Entra ID > ... > [SSO App] > Single Sign-On > SAML Certificates > Federation Metadata XML > Download > [fed xml file 1]
### Finish configuring AWS Identity Center to use external IdP(EntraId)
12. AWS > ... > Configure external identity provider > IdP SAML metadata > Choose file > [fed xml file 1]
13. AWS > ... > Configure external identity provider > IdP certificate > Choose file > [cert file 1]
14. AWS > ... > Configure external identity provider > Next 
15. Carefully read warning message
```
Review the following consequences of your requested identity source change:
You are changing your identity source to use an external identity provider (IdP).
IAM Identity Center will delete your current multi-factor authentication (MFA) configuration.
All current permission sets and SAML 2.0 application configurations will be retained.
IAM Identity Center preserves your current users and groups, and their assignments. However, only users who have usernames that match the usernames in your identity provider (IdP) can authenticate.
You must complete your identity provider (IdP) SAML configuration for IAM Identity Center so that your users can sign in. Identity Center will use your IdP for all authentications.
You must manage your multi-factor authentication (MFA) configuration and policies in your identity provider (IdP).
You must add (provision) all users in your identity provider (IdP) who will use IAM Identity Center before they can sign in. If you enable System for Cross-domain Identity Management (SCIM) to provision users and groups (recommended), your IdP will be the authoritative source of users and groups, and you must add and modify them in your IdP. Without SCIM, you can provision users and manage groups in IAM Identity Center only; all provisioned usernames must match the corresponding usernames in your IdP.
IAM Identity Center will keep your current configuration of attributes for access control. We recommend that you review your configuration and update it after you complete the identity source change.
```
### Test if SAML mechanism works (or)
16. Entra ID > ... > [SSO App] > Single Sign-On > Test single sign-on with [SSO App] > Test
```
{"message":"No access","__type":"com.amazonaws.switchboard.portal#ForbiddenException"}
```
17. AWS > IAM Identity Center > Settings > Identity source > AWS access portal URL
```
Your administrator has configured the application [SSO App] ('bla-bla') to block users unless they are specifically granted ('assigned') access to the application. The signed in user 'currently.selected.micfosoft.account@domain.com' is blocked because they are not a direct member of a group with access, nor had access directly assigned by an administrator. Please contact your administrator to assign access to this application.
```

## Setup provisioning mechanism(SCIM)
### Configure automactic provisioning in AWS (import User/Groups)
18. AWS > IAM Identity Center > Settings > Automatic provisioning > Enable > copy [SCIM endpoint] [Access token]
### Configure automactic provisioning in EntraId (export User/Groups)
19. Entra ID > ... > [SSO App] > Provisioning > Provisioning > Automatic > Tenant URL = [SCIM endpoint]
20. Entra ID > ... > [SSO App] > Provisioning > Provisioning > Automatic > Secret Token = [Access token]
21. Entra ID > ... > [SSO App] > Provisioning > Provisioning > Automatic > Test Connection
```
The supplied credentials are athorized to enable provisioning
```
22. Entra ID > ... > [SSO App] > Provisioning > Provisioning > Automatic > Save

## Setup authorisation
### Prepare permissions set for users/groups in AWS
23. AWS > IAM Identity Center > Permission sets > Create permission set > AdministratorAccess
24. AWS > IAM Identity Center > Permission sets > Create permission set > SecurityAudit
### Add users/group for provisioning in EntraId 
25. Entra ID > ... > [SSO App] > Users and Groups > + Add user/group > +your_account
> [!NOTE]
> When have a corportae subscription in Azure use groups mirroring roles instead of single users
> - Entra ID > Groups > New Group > Group type = security, Name = AWS_Administrators, Members = +your_account
> - Entra ID > Groups > New Group > Group type = security, Name = AWS_SecurityAuditors, Members = +your_account
### Enable automatic provisioning in EntraId / Run provisioning
26. Entra ID > ... > [SSO App] > Provisioning > Start provisioning
26. Entra ID > ... > [SSO App] > Provisioning > Refresh
```
Initial cycle completed.

View provisioning details
Completed:
13.10.2023, 22:38:58
Duration:
2.000 seconds
Steady state achieved:
13.10.2023, 22:38:58
Provisioning interval(fixed):
40 minutes
```
### Check if users/groups have been provisioned to AWS
27. AWS > IAM Identity Center > Users
```
Users (1)
your_account	Your magic name	Enabled	SCIM
```
### Map provisioned users/groups to permission sets
28. AWS > IAM Identity Center > AWS accounts > [target account] > Assign users or groups > Users > + your_account > Next > Permission sets > +AdministratorAccess, +SecurityAudit > Submit
29. Map for other child AWS Organization AWS accounts

## Test SSO (authentification and authorisation)
Entra ID > ... > [SSO App] > Single Sign-On > Test single sign-on with [SSO App] > Test > Test sign in > Select account 
```
https://d-1967707bad.awsapps.com/start#/

AWS Account (1)

you-account-name
#619537012324 | your-account@your-mailbox.com

AdministratorAccessManagement console  Command line or programmatic access
SecurityAuditManagement console  Command line or programmatic access
```
# Useful links
- https://learn.microsoft.com/ru-ru/azure/active-directory/saas-apps/aws-single-sign-on-tutorial
