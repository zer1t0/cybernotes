# Azure

Azure is the cloud service offered by Microsoft. Within Azure an individual or
organization can create Virtual Machines, access to Storage, deploy Web
Services, etc.

As many of these pages, we will examine Azure from a security perspective.


## Azure "managing" tools

The tools used for this purpose will be:

- [Azure CLI](https://learn.microsoft.com/en-us/cli/azure/)
- [Azure Powershell](https://learn.microsoft.com/en-us/powershell/azure/?view=azps-0.10.0)
- [AADInternals (tool)](https://github.com/Gerenios/AADInternals) by Dr Nestori Syynimaa (@DrAzureAD)
- [ROADtools (tool)](https://github.com/dirkjanm/ROADtools) by Dirk-jan Mollema
- [AZE](https://gitlab.com/Zer1t0/aze) by zer1t0
- https://portal.azure.com

## Azure Concepts

Azure is a complex environment, and it is easy to loose yourself with all the
terminology, so let's start by briefly defining some of the general concepts:

**Tenant**: It is an entity that represents the organization, and it has an
associated Entra ID instance (formerly known as Azure AD). Equivalent to a
domain in on-premises AD. Each organization can have one or several tenants
(usually big companies).

**Microsoft Entra ID**: It is the identity service that allows to manage all the
users and groups inside the Azure Tenant.

Under a tenant there are several levels or scopes of management. Permissions
applied to higher scopes are inherit by lower scopes:

**Management Group**: Allows to group subscriptions for an efficient
management. There is always a Root Management Group, and each management group
can have as children subscriptions or other management groups.

**Subscription**: A container inside a tenant to manage resources. A tenant can
have several subscriptions, each one can have a different [Azure Offer](https://azure.microsoft.com/en-us/support/legal/offer-details/). So
you can consider the subscriptions as the container level to which the money is
tied.

**Resource Group**: A group of resources, so they can be managed together.

**Resource**: Any item managed by azure, like VM, networks, vaults, storage,
etc.

Relation between the previous concepts:
```
                                                       .----------.
                                     Tenant <--------> | Entra ID |
                                       |               '----------'
                                       v
                           Tenant Root Management Group
                                       |
                       .------------------------------------.
                       v                                    v
               Management Group 1                   Management Group 2
                       |                                    |
           .-----------------------.                        |
           v                       v                        v
    Subscription 1.1        Subscription 1.2         Subscription 2.3
           |                       |                        |
           v                       v                        v
 Resource Group 1.1.1    Resource Group 1.2.1     Resource Group 2.3.1
           |                       |                        |
      .--------.                   |                .--------------.
      v        v                   v                v              v
     VM     Network          App Service          Vault       Storage Blob
```

You can find more information in [Azure Fundamental Concepts](https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/ready/considerations/fundamental-concepts).

### Tenant

A tenant, as we have mentioned, is an Azure instance that is available for a
customer that can be an individual or an organization (but we will usually in
terms of organization when refering to a tenant management). 

Each tenant has always at least one domain associated with it, by default
in the form of `<company>.onmicrosoft.com`. Other domains can be also associated
with a tenant, but the `onmicrosoft.com` domain will be always present.

Moreover, each tenant has an unique ID, surprisingly known as tenant ID, which
is an uuid in the form `31537af4-6d77-4bb9-a681-d2394888ea26`. It is important
to know the tenant ID since it is used heavily by the Azure API and
applications.

We can get the tenant ID of a domain associated with Azure just making a request
to
`https://login.microsoftonline.com/<domain>/.well-known/openid-configuration`. This
way we can also verify if a domain is associated with Azure.

Here is an example to extract the tenant ID:
```
$ curl -s https://login.microsoftonline.com/contoso.onmicrosoft.com/.well-known/openid-configuration | jq '.token_endpoint' -r | sed 's/^.*.com\///' | sed 's/\/oauth.*$//'
31537af4-6d77-4bb9-a681-d2394888ea26
```

It is also possible doing it with an aze command:
```
$ az-get-tenant-id contoso.onmicrosoft.com
31537af4-6d77-4bb9-a681-d2394888ea26
```

Or by using AADInternals:
```
PS> Get-AADIntTenantID -Domain contoso.onmicrosoft.com
31537af4-6d77-4bb9-a681-d2394888ea26
```

Moreover, by knowing a domain belonging to a tenant we can retrieve the others:
```
$ az-get-tenant-domains google.com
google.com
google0.onmicrosoft.com
```

```
PS> Get-AADIntTenantDomains -Domain openai.com
c-openai.com
democratic-inputs.com
openai.com
openaiinc.mail.onmicrosoft.com
openaiinc.onmicrosoft.com
```

This can be useful to know the domains associated to an organization.

And many more information public information can be retrieved with AADInternals:
```
PS C:\> Invoke-AADIntReconAsOutsider -DomainName contoso.onmicrosoft.com
Tenant brand:       CONTOSO.ONMICROSOFT.COM
Tenant name:        contoso.onmicrosoft.com
Tenant id:          31537af4-6d77-4bb9-a681-d2394888ea26
Tenant region:      AS
DesktopSSO enabled: False

Name    : contoso.onmicrosoft.com
DNS     : True
MX      : False
SPF     : False
DMARC   : False
DKIM    : False
MTA-STS : False
Type    : Managed
STS     :

```

From the authenticated side, if we got account credentials, we can get the
associate tenants with Azure cli (maybe not the most stealh way):
```
$ az account tenant list
Command group 'account tenant' is experimental and under development. Reference and support levels: https://aka.ms/CLI_refstatus
[
  {
    "id": "/tenants/31537af4-6d77-4bb9-a681-d2394888ea26",
    "tenantId": "31537af4-6d77-4bb9-a681-d2394888ea26"
  }
]
```

## Azure Authentication and Authorization

The term Azure authentication can be a little misleading since in
reality Azure is just a part in the authentication process which is
handled by the [Microsoft Identity Platform](https://learn.microsoft.com/en-us/entra/identity-platform/) from which Entra ID is
a part. 

This is a complex (and messy) topic since there are many components
and protocols involved. However, a large part of the authentication is
based on [OAuth-2.0-and Open ID Connect (OIDC) protocols](https://learn.microsoft.com/en-us/entra/identity-platform/v2-protocols), so if
you know the topic, it can a be easier.

### Authentication components

Right off the bat, we can see two initial different parts in the
authentication process, the user which wants to access and the
service, which is Azure.

However, Azure is not a single service, but a compound of many
different services that manages different parts of it, like virtual
machines, storage blobs, etc. And since Azure is integrated with other
Microsoft products, like those from Microsoft 365, an user can also
access Teams, Outlook, OneDrive, Word, etc, from its Azure/company
account, so we must also take those services into account.

So now we have the user on one part, and many Microsoft services into
the other. However, these services are not directly accessible, but
are managed by applications that can query and modify them, so the
user needs to access to these applications. An example of an
application is [Microsoft Graph](https://learn.microsoft.com/en-us/graph/overview). It is important to know that an
application does not can access only to one service, but many of them,
and maybe only to specific resources, and on the other hand, a
service can be accessed by many applications. For example, 
[Microsoft Graph can access to several Microsoft services](https://learn.microsoft.com/en-us/graph/overview-major-services) like
Entra ID, Outlook, etc. This means that there are several ways
(applications) for an user to access to the same service, which can be
confusing (and also opens ways to bypass certain resources access).

On the other hand, in order to access to those applications (and
therefore the services behind them) the user needs to use some kind of
software, like a web browser, powershell, Outlook app, etc. This
software is known as the client, and as we have seen, there are many
of them, that can be granted different levels of access. So maybe an
organization allows to access to user mails through the Outlook
app, but not using Powershell.

So now we have the user, the client (used software), the application
and the services. Here is a little schema of the situation:
```
                                                .-----
                                           .--> | service 1 
                                           |    '-----
										   |		
  \O/    .--------.       .-------------.  |    .-----
   | --->| client |------>| application | -|--> | service 2
  / \    '--------'       '-------------'  |    '-----
  user                                     |
                                           |    .----
                                           '--> | service 3
                            				    '----
```

But in the shake of correctness, we can translate these roles to the
OAuth 2.0 schema, that would be something like the following:
```
  \O/    .--------.       .-------------------.
   | --->| client |------>| resource provider |
  / \    '--------'       '-------------------'
 resource
 owner
```

As you can see a few things have change. First the user it is named as
**resource owner** since we can think that she wants to access to its
own resources. I personally don't like the term cause many times a
user wants to access to resources from other people, like consulting
their email address, but still that is the term used in OAuth 2.0 and
knowing its meaning will make us easier to read documentation. Second,
the target application is named the **resource provider** because it
provides access to the resources, usually through an API. And third we
have eliminated the services since in OAuth 2.0 that part is left
to the specific implementation, but in our case of Azure it is
important to know that behind many applications there are the
Microsoft services and we can access to them in many cases by using
different Microsoft resource providers/applications.

And other thing that is important to note for the sake of clarity is
that clients are also applications. So we can have applications that
act as clients or as resource providers. For example the [Azure
CLI](https://learn.microsoft.com/en-us/cli/azure/) is just a client and [Microsoft Graph](https://learn.microsoft.com/en-us/graph/overview) is just a resource
provider, but others like the [Azure Portal](https://portal.azure.com) can act as both client
or resource provider. This may sound counterintuitive but may think
that an application just have access direct access to a subset of
resources that it manages and it needs to collaborate with other
applications to access to all the services and resources it needs. As
analogy, if you are a clothing retailer, you can access the clothing
directly, but if you need other materials such as boxes, you need to
buy them from a box retailer.

Here is our previous schema with a concrete client, resource provider
and the resources:
```
                                             .-----
                                        .--> | Groups
                                        |    '-----
	                                    |		
  \O/    .--------.       .----------.  |    .-----
   | --->| AZ CLI |------>| MS Graph | -|--> | Users
  / \    '--------'       '----------'  |    '-----
  user                                  |
                                        |    .----
                                        '--> | Mails
                            		         '----
```

Hope this allows to understand more the role of applications in the
OAuth 2.0 process. But I would like also to point that an application
can even act as both the client and resource provider at the same
time, like the case of Azure Portal.

Now we have most of the ingredients, but we are still missing a central
piece. What is in charge of authenticate the user and grant access to
the services? Yep, the [Microsoft Identity Platform](https://learn.microsoft.com/en-us/entra/identity-platform/) we already
commented, that acts as Identity Provider (or authorization server) in
which applications trust to provide access.

Here is a more complete picture of the Azure authentication components:
``` 
                   .-------------.
                   | MS Identity |
			  .--> | Platform    |
              |    '-------------'                    .-----
              |                                  .--> | resource 1 
              |                                  |    '-----
			  |                                  |		
  \O/    .--------.       .-------------------.  |    .-----
   | --->| client |------>| resource provider | -|--> | resource 2
  / \    '--------'       '-------------------'  |    '-----
  user                                           |
                                                 |    .----
                                                 '--> | resource 3
                            				          '----
```

In order to recap we have the following components:

- **User** / **Resource Owner**: The entity that wants to access to
  the resources. It needs to have permissions to access to the
  resources.
  
- **Client** : The application that acts on behalf of the user to
  access to the resources. Sometimes it just acts on its own behalf
  without requiring the user. Anyway, it needs to have permissions
  (apart from those of the user) to access the resources. Examples are
  the Azure CLI, Azure Powershell, Outlook application, Teams
  application or the Azure Portal.
  
- **Resource provider**: The application that provides access to the
  resources. Examples are Microsoft Graph or Azure Storage.
  
- **Resources**: The items that the client wants to access or
  manipulated through the resource provider. Examples can be Users,
  Groups, Virtual Machines, Mails, etc. This is not part of OAuth, but
  it is important to understand that resources are not the same as
  resource providers and in fact, many are accessible from different
  resource providers.
  
- **Identity Provider** / **Authorization Server** : The entity in
  charge of the authentication and authorization. This provider
  identifies the user and client application and indicates the access
  they have into the target resources. If you are familiar with
  Kerberos in Active Directory, this acts similar to a KDC.


### Authentication flow

And how is the access provided? Well, mostly following the 
[OAuth 2.0](https://oauth.net/2/) and [OIDC](https://openid.net/developers/how-connect-works/) protocols (even if others like [SAML](https://en.wikipedia.org/wiki/Security_Assertion_Markup_Language)
are also supported).

The basic flow is the following:

1. The client will ask to the Identity Provider for an access token to
  the desired application on behalf of the user, whose credentials
  (like username and password) are sent.
2. If the provided credentials are correct, the Identity Provider will
  return an access token for the client to the application. This
  access token will contain the scope for which the client has access
  in the target application (for example the Outlook client can read
  emails but not list user Intune devices in Microsoft Graph (I'm not
  sure, just an example)).
3. The client will then request actions to the target application
  including the access token in the request.
  
We can perform the previous flow by using roadtx. Here is an example:
```
roadtx gettokens -u ada@contoso.onmicrosoft.com -p Sup3rP4ssw0rd -c aadps -r msgrap
```

That will create the following HTTP request to ask for an access token
for Microsoft Graph (sent to
`https://login.microsoftonline.com/common/oauth2/token`):
```
POST /common/oauth2/token HTTP/2
Host: login.microsoftonline.com
Content-Length: 218
Content-Type: application/x-www-form-urlencoded

client_id=1b730954-1685-4b74-9bfd-dac224a7b894&grant_type=password
&resource=https%3A%2F%2Fgraph.microsoft.com%2F
&username=ada%40contoso.onmicrosoft.com&password=Sup3rP4ssw0rd
```

Let's examine the HTTP request parameters:

- **client_id**: Identifies the client which is requesting
  access. In this case `1b730954-1685-4b74-9bfd-dac224a7b894`
  indicates that is Powershell. You can get a list of Microsoft Client
  IDs in [dafthack/azure_client_ids.txt](https://gist.github.com/dafthack/2c0bbcac72b10c1ee205d1dd2fed3fe7). Moreover, non Microsoft
  clients will include also a **client_secret** parameter that allows
  them to authenticate against Microsoft.

- **resource**: Indicates the application for which are
  requesting access, in this case Microsoft Graph
  (`https://graph.microsoft.com/`). The application is known as
  *resource provider* in OAuth 2.0, so that is the reason for the
  parameter name.

- **grant_type**: Indicates which type of credentials are we sending
  to request an access token. In this case the value `password`
  indicates that we are sending username and password pair. The
  supported grant types are:
  + [authorization_code](https://learn.microsoft.com/en-us/entra/identity-platform/v2-oauth2-auth-code-flow)
  + [client_credentials](https://learn.microsoft.com/en-us/entra/identity-platform/v2-oauth2-client-creds-grant-flow): Used when the client is not impersonating
    an user, but accessing by itself.
  + [urn:ietf:params:oauth:grant-type:device_code](https://learn.microsoft.com/en-us/entra/identity-platform/v2-oauth2-device-code)
  + [urn:ietf:params:oauth:grant-type:jwt-bearer](https://learn.microsoft.com/en-us/entra/identity-platform/v2-oauth2-on-behalf-of-flow)
  + [password](https://learn.microsoft.com/en-us/entra/identity-platform/v2-oauth-ropc)
  + [refresh_token](https://www.oauth.com/oauth2-servers/access-tokens/refreshing-access-tokens/)
  
- **username** and **password**: The credentials of the user.

If for some reason the access token request is rejected, a response
like the following will be received:
```
HTTP/2 400 Bad Request
Content-Type: application/json; charset=utf-8
Content-Length: 486

{
	"error":"invalid_grant",
	"error_description":"AADSTS50126: Error validating credentials due
	to invalid username or password. Trace ID:
	533bf45e-1c05-4e8a-bb1b-313991a73a00 Correlation ID:
	7c77db90-c37c-465a-9e7a-553fe968c700 Timestamp: 2025-02-27
	22:38:35Z",
	"error_codes":[50126],
	"timestamp":"2025-02-27 22:38:35Z",
	"trace_id":"533bf45e-1c05-4e8a-bb1b-313991a73a00",
	"correlation_id":"7c77db90-c37c-465a-9e7a-553fe968c700",
	"error_uri":"https://login.microsoftonline.com/error?code=50126"
}
```

As we can see, in case of error, the identity endpoint provide us with
an error message.
  
On the other hand, in case credentials are correct, the response will
be something like the following:
```
HTTP/2 200 OK
Content-Type: application/json; charset=utf-8
Content-Length: 3966

{
	"token_type":"Bearer",
	"scope":"Agreement.Read.All Agreement.ReadWrite.All
	AgreementAcceptance.Read AgreementAcceptance.Read.All
	AuditLog.Read.All Directory.AccessAsUser.All
	Directory.ReadWrite.All Group.ReadWrite.All
	IdentityProvider.ReadWrite.All Policy.ReadWrite.TrustFramework
	PrivilegedAccess.ReadWrite.AzureAD
	PrivilegedAccess.ReadWrite.AzureADGroup
	PrivilegedAccess.ReadWrite.AzureResources
	TrustFrameworkKeySet.ReadWrite.All User.Invite.All",
	"expires_in":"4294",
	"ext_expires_in":"4294",
	"expires_on":"1740688710",
	"not_before":"1740684115",
	"resource":"https://graph.microsoft.com/",
	"access_token":"eyJ0eXAiO...stripped...",
	"refresh_token":"1.AUEBsR...stripped..."
}
```

Here we can see the following interesting items:

- **token_type**: The type of token the Identity Provider is
  returning. Usually a [Bearer token](https://stackoverflow.com/questions/25838183/what-is-the-oauth-2-0-bearer-token-exactly) that can be used in a
  [Authorization header](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Authorization).

- **access_token**: The token the client must use to access to the
  application. It is worth further examination since it is a
  [JWT](https://en.wikipedia.org/wiki/JSON_Web_Token).

- **scope**: Indicates the permissions the access token grants in the
  target application. This scope depends on both the user and the
  client permissions.
  
- **expires_on**: Indicates when the access token will expire.
  
- **refresh_token**: A [refresh token](https://oauth.net/2/refresh-tokens/) is a token we can use to ask
  for another access token when the current one is expired (the
  [refresh token has a fixed expiration date](https://learn.microsoft.com/en-us/entra/identity-platform/refresh-tokens)). If you are
  familiar with Kerberos, the refresh token is similar to a TGT.


### Azure tokens


The following resources are interesting to learn more about Azure
tokens:
- [Workshop: Tokens, everywhere! (Video)](https://www.youtube.com/watch?v=8qEh1pc0tT8&t=1694s)
- [Workshop: Tokens, everywhere! (Slides)](https://aadinternals.com/talks/Tokens%20Everywhere.pdf)


#### Access Token

Let's inspect the access token:
```
$ az-inspect-token eyJ0eXAiO...stripped...
{
    "aud": "https://graph.microsoft.com/",
    "iss": "https://sts.windows.net/1c3f13b1-7446-46a9-9f66-37b267bf84d7/",
    "iat": 1740684115,
    "nbf": 1740684115,
    "exp": 1740688710,
    "acct": 0,
    "acr": "1",
    "aio": "ATQAy/8ZAAAALNgbMGtK573VwZNbRo/U1dbz70uE+iBOT//DZSvX25PAn1Ygz7f2lzICRXUS1Pk7",
    "amr": [
        "pwd"
    ],
    "app_displayname": "Azure Active Directory PowerShell",
    "appid": "1b730954-1685-4b74-9bfd-dac224a7b894",
    "appidacr": "0",
    "family_name": "Lovelace",
    "given_name": "Ada",
    "idtyp": "user",
    "ipaddr": "42.42.42.42",
    "name": "Ada Lovelace",
    "oid": "963641b0-9772-4303-b447-7e87eec95f3b",
    "platf": "14",
    "puid": "1003200435DDAB02",
    "rh": "1.AUEBsRM_HEZ0qUafZjeyZ7-E1wMAAAAAAAAAwAAAAAAAAABHATtBAQ.",
    "scp": "Agreement.Read.All Agreement.ReadWrite.All AgreementAcceptance.Read AgreementAcceptance.Read.All AuditLog.Read.All Directory.AccessAsUser.All Directory.ReadWrite.All Group.ReadWrite.All IdentityProvider.ReadWrite.All Policy.ReadWrite.TrustFramework PrivilegedAccess.ReadWrite.AzureAD PrivilegedAccess.ReadWrite.AzureADGroup PrivilegedAccess.ReadWrite.AzureResources TrustFrameworkKeySet.ReadWrite.All User.Invite.All",
    "sid": "0022ab19-2d83-2597-935a-aca54755f7d7",
    "sub": "9MuacgjWPd6RFnrYGwUp3nTJfP2_Kk2EBIKKCJ3cwSM",
    "tenant_region_scope": "EU",
    "tid": "1c3f13b1-7446-46a9-9f66-37b267bf84d7",
    "unique_name": "ada@contoso.OnMicrosoft.com",
    "upn": "ada@contoso.OnMicrosoft.com",
    "uti": "ODrMaoSqNEKkrRXOvkFyAQ",
    "ver": "1.0",
    "wids": [
        "b79fbf4d-3ef9-4689-8143-76b194e85509"
    ],
    "xms_idrel": "16 1",
    "xms_tcdt": 1737118650,
    "xms_tdbr": "EU"
}
```

It is important to understand access token as it is the main
authentication item and from time to time we may have to deal with
them directly. So let's take a closer look at the most relevant
fields, also known as claims:

- *aud*: The audience of the token, which is the target application.
- *amr*: The authentication methods used by the user, in this case
  only `pwd` meaning password, but if MFA is also requested, it will
  also contain `mfa`.
- *exp*: Indicates when the token expires. Usually an access token
  lasts 24 hours and it is not affected by password changes.
- *appid*: The client id to which the token is linked.
- *idtyp*: Indicates if the client is impersonating an user or not.
- *oid*: The user ID.
- *scp*: The scopes for which the user and client received access.
- *sid*: Session ID.
- *tid*: The tenant ID in which the user is singning in to.
- *wids*: Identify the roles associated to the user.

You can see the description of the rest of claims in 
[Access token claims reference: Payload Claims](https://learn.microsoft.com/en-us/entra/identity-platform/access-token-claims-reference#payload-claims).

In order to test other tokens for other clients, applications and
authentication methods, you can use roadtx:
```
$ roadtx gettokens -u ada@contoso.onmicrosoft.com -p Sup3rP4ssw0rd --client aadps --resource msgraph --tokens-stdout | jq '.accessToken' | xargs roadtx describe -t
{
    "alg": "RS256",
    "kid": "imi0Y2z0dYKxBttAqK_Tt5hYBTk",
    "nonce": "m11X6LF5U6BYX03bWpska85YYOZ0G-nI5a2vK6LoTJQ",
    "typ": "JWT",
    "x5t": "imi0Y2z0dYKxBttAqK_Tt5hYBTk"
}
{
    "aud": "https://graph.microsoft.com/",
    "iss": "https://sts.windows.net/1c3f13b1-7446-46a9-9f66-37b267bf84d7/",
    "iat": 1740684115,
    "nbf": 1740684115,
    "exp": 1740688710,
    "acct": 0,
    "acr": "1",
    "aio": "ATQAy/8ZAAAALNgbMGtK573VwZNbRo/U1dbz70uE+iBOT//DZSvX25PAn1Ygz7f2lzICRXUS1Pk7",
    "amr": [
        "pwd"
    ],
    "app_displayname": "Azure Active Directory PowerShell",
    "appid": "1b730954-1685-4b74-9bfd-dac224a7b894",
    "appidacr": "0",
    "family_name": "Lovelace",
    "given_name": "Ada",
    "idtyp": "user",
    "ipaddr": "42.42.42.42",
    "name": "Ada Lovelace",
    "oid": "963641b0-9772-4303-b447-7e87eec95f3b",
    "platf": "14",
    "puid": "1003200435DDAB02",
    "rh": "1.AUEBsRM_HEZ0qUafZjeyZ7-E1wMAAAAAAAAAwAAAAAAAAABHATtBAQ.",
    "scp": "Agreement.Read.All Agreement.ReadWrite.All AgreementAcceptance.Read AgreementAcceptance.Read.All AuditLog.Read.All Directory.AccessAsUser.All Directory.ReadWrite.All Group.ReadWrite.All IdentityProvider.ReadWrite.All Policy.ReadWrite.TrustFramework PrivilegedAccess.ReadWrite.AzureAD PrivilegedAccess.ReadWrite.AzureADGroup PrivilegedAccess.ReadWrite.AzureResources TrustFrameworkKeySet.ReadWrite.All User.Invite.All",
    "sid": "0022ab19-2d83-2597-935a-aca54755f7d7",
    "sub": "9MuacgjWPd6RFnrYGwUp3nTJfP2_Kk2EBIKKCJ3cwSM",
    "tenant_region_scope": "EU",
    "tid": "1c3f13b1-7446-46a9-9f66-37b267bf84d7",
    "unique_name": "ada@contoso.OnMicrosoft.com",
    "upn": "ada@contoso.OnMicrosoft.com",
    "uti": "ODrMaoSqNEKkrRXOvkFyAQ",
    "ver": "1.0",
    "wids": [
        "b79fbf4d-3ef9-4689-8143-76b194e85509"
    ],
    "xms_idrel": "16 1",
    "xms_tcdt": 1737118650,
    "xms_tdbr": "EU"
}
```


#### Refresh token

On the other hand we have the [refresh token](https://oauth.net/2/refresh-tokens/) that is  a token we
can use to ask for another access token when the current one is
expired. As we have mentioned, it is similar to TGT in Kerberos.

For example, we can use roadtx to ask for a new refresh and access
token by using a refresh token:
```
$ roadtx gettokens --refresh-token 1.AUEBsRM_HEZ0qUafZjeyZ7...stripped... --client aadps --resource msgraph --tokens-stdout | jq
{
  "tokenType": "Bearer",
  "expiresOn": "2025-03-01 10:57:22",
  "tenantId": "1c3f13b1-7446-46a9-9f66-37b267bf84d7",
  "_clientId": "1b730954-1685-4b74-9bfd-dac224a7b894",
  "accessToken": "eyJ0eXAiOiJKV1QiLCJub25jZSI6...stripped...",
  "refreshToken": "1.AUEBsRM_HEZ0qUafZjey...stripped...",
  "expiresIn": "5281"
}
```

If we examine the HTTP request we will find `refresh_token` as `grant_type`:
```
POST /common/oauth2/token HTTP/2
Host: login.microsoftonline.com
Content-Length: 941
Content-Type: application/x-www-form-urlencoded

client_id=1b730954-1685-4b74-9bfd-dac224a7b894&grant_type=refresh_token
&refresh_token=1.AUEBsRM_HEZ0qUafZjeyZ7...stripped...
&resource=https%3A%2F%2Fgraph.microsoft.com%2F
```

A bad notice is that the refresh tokens cannot be inspected as the
access tokens since they are encrypted, and their information is only
available to Microsoft services. 

However, we know that each refresh token is tied to both the user and
the client (excep in case of Family Refresh Tokens). So a refresh
token can only be used by a client to request access on behalf on an
specific user. 

If we tried to use a refresh token for another client id, we will
receive a similar response to the following:
```
$ roadtx gettokens --refresh-token 1.AUEBsRM_HEZ0qUafZjeyZ7...stripped... --client fc0f3af4-6835-4174-b806-f7db311fd2f3 --resource msgraph
Error during authentication: AADSTS70000: Provided grant is invalid or 
malformed. Trace ID: 41eb8263-e7ab-4529-8910-a8a376676001 
Correlation ID: 23d48881-43e4-470e-b243-f88295f86c2d Timestamp: 
2025-03-01 09:37:16Z
```

The refresh token also has an expiration date, but this is fixed:
- refresh tokens for SPA (Single Page Applications) will last 24 hours
- refresh tokens for other applications will live 90 days

However, we can request newer refresh tokens by using the previous
ones, thus avoiding the timeout. So, how can the refresh tokens
expire?

The other option apart from the lifetime, is to revoke the token by
requiring the user to sign-in again, or change her
credentials. However there are some caveats and you can read all the
details in [Refresh tokens in the Microsoft identity platform](https://learn.microsoft.com/en-us/entra/identity-platform/refresh-tokens).

#### Family Refresh Token

A Family Refresh Token (FRT) is a refresh token issued to an special
set of Microsoft clients, which are part of the Family Of Client IDs
(FOCI). The particularity of these tokens is that they can be used to
ask for access tokens for any client of the family.

The list of clients belonging to the FOCI can be found in
[known-foci-clients.csv](https://github.com/secureworks/family-of-client-ids-research/blob/main/known-foci-clients.csv).

This strange behaviour was found by researchers Ryan Marcotte Cobb and
Tony Gore in its [Abusing Family Refresh Tokens for Unauthorized
Access and Persistence in Azure Active Directory](https://github.com/secureworks/family-of-client-ids-research) research.

This behaviour has security implications, since scopes on target
applications (also known as resource providers) are tied to the client
permissions (and user too). However, by having a FRT it could be
[possible to bypass some restrictions by asking for an access token
linked to other client](https://github.com/secureworks/family-of-client-ids-research?tab=readme-ov-file#what-are-the-security-implications-of-family-refresh-tokens) of the family. 


Let's see a pratical example. First we will ask for a FRT token with
roadtx. In this case we will ask for a Azure CLI (a FOCI client)
refresh token for Microsoft Graph:
```
$ roadtx gettokens -u ada@contoso.onmicrosoft.com -p Sup3rP4ssw0rd --client azcli --resource msgraph --tokens-stdout | jq '.refreshToken' -r
1.AUEBsRM_HEZ0qUafZjeyZ7...stripped...
```

Let's now check the scope we achieve if we ask for a access token by
using the retrieved refresh token, still for Azure CLI:
```
$ roadtx gettokens --refresh-token 1.AUEBsRM_HEZ0qUafZjeyZ7...stripped... --client azcli --resource msgraph --tokens-stdout | jq '.accessToken' | xargs roadtx describe -t | jq '.scp'
null
"AuditLog.Read.All Directory.AccessAsUser.All Group.ReadWrite.All User.ReadWrite.All"
```

We see we have receive permission for accessing users and groups. Now
let's check by using the same refresh token and changing the client to
the Teams application, which is also FOCI:
```
$ roadtx gettokens --refresh-token 1.AUEBsRM_HEZ0qUafZjeyZ7..stripped... --client teams --resource msgraph --tokens-stdout | jq '.accessToken' | xargs roadtx describe -t | jq '.scp'
null
"AppCatalog.Read.All Channel.ReadBasic.All Contacts.ReadWrite.Shared Files.ReadWrite.All FileStorageContainer.Selected InformationProtectionPolicy.Read Mail.ReadWrite Mail.Send MailboxSettings.ReadWrite Notes.ReadWrite.All People.Read Place.Read.All Sites.ReadWrite.All Tasks.ReadWrite Team.ReadBasic.All TeamsAppInstallation.ReadForTeam TeamsTab.Create User.ReadBasic.All"
```

The picture looks different now, we have retrieve additional
permissions for other services like Teams or Notes.

#### ID token

An ID token is a token similar to an access token in format, since
both are JWT and have similar claims, but with a different
purpose. Access tokens are intended to be transmitted from the client
to the resource provider (target application) whereas an ID token is
transmitted from the Identity Provider to the client to allow this one
to verify user data. Therefore ID tokens are used for authentication
purposes, but not authorization, in fact they lack claims like
scopes.

Let's inspect a ID token:

```
$ roadtx describe -t eyJ0eXAiOiJKV1QiLCJ...stripped...
{
    "alg": "none",
    "typ": "JWT"
}
{
    "amr": [
        "pwd",
        "rsa"
    ],
    "aud": "38aa3b87-a06d-4817-b275-7a316988d93b",
    "exp": 1740750565,
    "family_name": "Lovelace",
    "given_name": "Ada",
    "iat": 1740684115,
    "idtyp": "user",
    "ipaddr": "42.42.42.42",
    "iss": "https://sts.windows.net/1c3f13b1-7446-46a9-9f66-37b267bf84d7/",
    "name": "Ada Lovelace",
    "nbf": 1740684115,
    "oid": "963641b0-9772-4303-b447-7e87eec95f3b",
    "pwd_url": "https://portal.microsoftonline.com/ChangePassword.aspx",
    "rh": "1.AUEBsRM_HEZ0qUafZjeyZ7-E14c7qjhtoBdIsnV6MWmI2TtHATtBAQ.",
    "sid": "S-1-12-1-278177916-1122736458-3900963728-4273191113",
    "sub": "wIAjLD5xSOm3-rAuKF7qt_L_R5zU1NQqjVc-O_ALaxo",
    "tenant_display_name": "Contoso",
    "tid": "1c3f13b1-7446-46a9-9f66-37b267bf84d7",
    "unique_name": "ada@contoso.OnMicrosoft.com",
    "upn": "ada@contoso.OnMicrosoft.com",
    "ver": "1.0",
    "xms_idrel": "1 2"
}
```

We can appreciate that fields are similar to those in a access token,
but some differences arise, like that it has no expiration time and
there is no scope information. This as we have said, makes sense since
it is not a token used to request access, but just used by the
Identity Provider to inform about the user data. In this example you
may notice that the ID token is not signed, but this is not always the
case. For more information about the fields you can check [ID token
claim reference](https://learn.microsoft.com/en-us/entra/identity-platform/id-token-claims-reference).

In case you are curious, you can get an example of ID token by inspecting the communication
while adding a device to Intune with [pytune](https://github.com/secureworks/pytune) like this (and in
many other situations):
```
$ export HTTPS_PROXY=http://127.0.0.1:8080

$ python3 ./pytune.py entra_join -o Windows -d fake-device -u ada@contoso.onmicrosoft.com -p SuperP4ssw0rd
Saving private key to fake-device_key.pem
Registering device
Device ID: ae8e388b-2bfe-4d1e-9207-6bc77d9d2839
Saved device certificate to fake-device_cert.pem
[+] successfully registered fake-device to Entra ID!
[*] here is your device certificate: fake-device.pfx (pw: password)

$ python3 ./pytune.py enroll_intune -o Windows -d fake-device -c fake-device.pfx -u ada@contoso.onmicrosoft.com -p SuperP4ssw0rd
[*] resolved enrollment url: https://fef.msub06.manage.microsoft.com/StatelessEnrollmentService/DeviceEnrollment.svc
[+] successfully enrolled fake-device to Intune!
[*] here is your MDM pfx: fake-device_mdm.pfx (pw: password)

```

If we inspect the HTTP request through a proxy like BurpSuite, we can
see requests and responses from
`https://login.microsoftonline.com/common/oauth2/token`. Some of them
should contain a ID token in a JSON identified by the (surprise)
`id_token` field.

## Resources


- [Awesome Azure Penetration Testing](https://github.com/Kyuu-Ji/Awesome-Azure-Pentest) by Kyuu-Ji
- [AADInternals.com](https://aadinternals.com/) by Dr Nestori Syynimaa (@DrAzureAD)
- [Good Workaround!](https://goodworkaround.com/) by Marius Solbakken
- [dirkjanm.io](https://dirkjanm.io/) by Dirk-jan Mollema




