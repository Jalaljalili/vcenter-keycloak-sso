# vcenter-keycloak-sso
Vcenter authenticate SSO by Open-ldap users

## Create Directory
Create the necessary directory in the main directory
```bash
mkdir -p ./postgres
mkdir -p ./openldap/slapd/database
mkdir -p ./openldap/slapd/config:/etc/ldap/slapd.d
mkdir -p ./openldap/slapd/schema
```
### Schema
Incert schema at scheam directory
```bash
cp schema.local ./openldap/slapd/schema
```

## Installation
Running postgres, keycloak, openldap, phpldapadmin container
```bash
docker-compose up -d --force-recreate
```

## Config Keycloack
* Add new client, then specify :
        
        - Client ID : {name}
        - Client Protocol : {openid-conect}
        - Root URL : {vcenter-URL}
        - * Valid Redirect URIs : {https://VCENTER_URL/ui/login/oauth2/authcode}
        - Admin URL :  {vcenter-URL}
        - Web Origins : {*}
        - Backchannel Logout URL : {https://VCENTER_URL/ui/login}
        - Access Type : {confidential}
        - Direct Access Grants Enabled : {ON}
        - Backchannel Logout Session Required {ON}
        - Client Authenticator : {Clienr ID and Secret}

     In Mappers TAB:

        Create 3 Mappers:

        - domain >>> Hardcoded claim >>> Protocol : {openid-connect} >>> Claim JSON Type : {string} >>> Add to ID token, Add to access token, Add to userinfo  : {on} >>> includeInAccessTokenResponse.label {OFF}

        - groups_script >>> Script Mapper >>> 
        ```script
        token.setOtherClaims("group", user.getGroups().toArray()[0].getName())
        ```
        
        - nameid >>> Script Mapper >>> 
        ```script
        token.setSubject(user.getUsername());
  
        ```
* User Federation :

        Add provider >>> ldap :
        - Import Users: {ON}
        - Sync Registration: {ON}
        - Username LDAP attribute : {cn}
        - RDN LDAP attribute : {cn}
        - UUID LDAP attribute : {entryUUID}
        - User Object Classes : {user}
        - Connection URL : {ldap://openldap:389}
        - Users DN : {dc=test,dc=com}
        - Bind Type : {simple}
        - Bind DN : {cn=admin,dc=test,dc=com}
        -  Bind Credential : {PASSWORD}

        Add Mapper:
        - Name: {groups_mapper} >>> Type:{group-ldap-mapper} 
            - LDAP Groups DN : {dc=test,dc=com}
            - Group Name LDAP Attribute : {cn}
            - Group Object Classes : {group}
            - Membership LDAP Attribute : {member}
            - Membership Attribute Type : {DN}
            - Membership User LDAP Attribute : {cn}
            - Mode: {READ_ONLY}
            - User Groups Retrieve Strategy :{LOAD...}
            - Member-Of LDAP Attribute : {memberOff}
            - Groups Path : {/}

        - If need verified email automaticly:
          - Name: {Email_verified} >>> Mapper Type :{hardcoded-attribute-mapper} >>> User Model Attribute Name : {emailVerified} >>> Attribute Value : {true} 

## Config Open-LDAP


## Config Vcenter
 CHANGE IDENTITY PROVIDER on Single Sing On Configuration
  
  * 1. Identity Provider
        Microsoft ADFS
  * 2. ADFS server
        Client Identifier : {Cline Name KeyCloak}
        Shared secret : {Credentials Secret Client from KeyCloak}
        OpenID Address : {http://{keycloakIP}:28080/auth/realms/{Domain}/.well-known/openid-configuration}

  *  3. Users and Groups
        Base distinguished name for users: {dc=test,dc=com}
        Base distinguished name for groups: {dc=test,dc=com}
        Username: {cn=admin,dc=test,dc=com}
        Password : {"Domain_Password"}
        Primary server URL: {ldap://"Open_ldap_IP"}

   After that Reboot the Vcenter in order to be applied the Configurations 
   



