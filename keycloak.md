# Keycloak

## Installation
- Download Keycloak server from: https://www.keycloak.org/downloads.html (https://www.keycloak.org/archive/downloads-5.0.0.html)
  - >At time of writing the latest version is 6.0.1 but we are using 5.0.0 because Wildfly 15 is deprecated for 6.0.1.
- Follow getting started guide: https://www.keycloak.org/docs/latest/getting_started/index.html (PDF attached below)
  - Created admin user: admin/xxxxxxxx
  - You can add the following to standalone.conf[.bat] so you don't need to start the server with command line arguments:
    - (windows) set "JAVA_OPTS=%JAVA_OPTS% -Djboss.socket.binding.port-offset=100"
    -  (linux) JAVA_OPTS="$JAVA_OPTS -Djboss.socket.binding.port-offset=100
  - >Step 4.3 should be done BEFORE step 4.2 or you will get the following error:
    - The required mechanism 'BASIC' is not available in mechanisms Keycloak from the HttpAuthenticationFactory
  - >Since we are using an older version of Keycloak (5.0.0) I could not use maven to build the demo app (vanilla). I manually built the demo app. It is attached as vanilla.war
  - For https update the keystore in standalone.xml
  - Add the following line to the :keycloak node in standalone.xml
    - ```
      <!-- use the username in the remoteUser not the userID -->
      <principal-attribute>preferred_username</principal-attribute>
      ```
  - Changes to web.xml
    - ```
      <security-constraint>
        <web-resource-collection>
          <web-resource-name>CHP SSO</web-resource-name>
          <url-pattern>/*</url-pattern>
        </web-resource-collection>
        <auth-constraint>
          <role-name>domain users</role-name>
        </auth-constraint>
      </security-constraint>
      <login-config>
        <auth-method>BASIC</auth-method>
      </login-config>
      <security-role>
        <role-name>domain users</role-name>
      </security-role>
      ```

## Setting up Keycloak and AD FS
- Follow Keycloak blog post on setting up AD FS as Identity Provider: https://www.keycloak.org/2017/03/how-to-setup-ms-ad-fs-30-as-brokered-identity-provider-in-keycloak.html (PDF attached below)
- Update all references of http://localhost to https://hostname.domainname.net
- >In "Identity Providers" for your realm, the "NameID Policy Format" needs to be set to "Windows Domain Qualified Name"
- Added additional mapper first name so a user isn't prompted to enter these on first Keycloak login.
  - Mapper Type: Attribute Importer
  - Attribute Name: http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname
  - User Attribute Name: firstName
- Added additional mapper last name so a user isn't prompted to enter these on first Keycloak login.
  - Mapper Type: Attribute Importer
  - Attribute Name: http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname
  - User Attribute Name: lastName

## Setting up Keycloak and Okta
- Follow Keycloak blog post on setting up okta as Identity Provider: https://ultimatesecurity.pro/post/okta-saml/ (PDF attached below)
- Setting up the attributes is the same as the AD FS setup for Keycloak. You will need to add the Attribute Statements (optional) in okta.
- Create the appropriate group and assign your user(s) to that group.
- Add the user(s) to your application
- My sample okta account is:
  - ```
    Okta organization name: xxxxxxxx-org-999999
    Okta homepage: https://xxxxxxxx-xxxxxxxx.okta.com
    Okta username: xxxxxxxx@xxxxxxxx.com
    ```
