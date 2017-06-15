# Keycloak: get started with Wildfly web app

### Start Keycloak and Wildfly
```
docker-compose up
```

Add the following to your `/etc/hosts` file:
```
127.0.0.1       keycloak
```

Verify the Docker containers are started:
```
$ docker-compose ps
             Name                           Command               State                       Ports
------------------------------------------------------------------------------------------------------------------------
keycloakwildflydemo_keycloak_1   /opt/jboss/docker-entrypoi ...   Up      0.0.0.0:8080->8080/tcp
keycloakwildflydemo_wildfly_1    /opt/jboss/wildfly/bin/sta ...   Up      0.0.0.0:8081->8080/tcp, 0.0.0.0:9990->9990/tcp
```

Add admin user to Wildfly:
```
docker exec -it keycloakwildflydemo_wildfly_1 /opt/jboss/wildfly/bin/add-user.sh admin password
```

Your Keycloak server is now available at http://keycloak:8080/. You can log in as admin/password.

Your Wildfly server is available at http://localhost:8081/. You can log in Admin Console as admin/password.

### Install sample app on Wildfly
Log in Wildfly Admin Console (http://localhost:9990/console/) and deploy web app `vanilla.war`. The app should be available at http://localhost:8081/vanilla. It is initially configured to use basic authentication.

### Setup Keycloak
Add a new realm `demo` and user `johndoe` in realm `demo`.

Under realm `demo`, create a new client: 

| Field        | Value           |
| ------------- |:-------------:|
| Client ID      | vanilla |
| Client Protocol     | openid-connect |
| Root URL | http://localhost:8081/vanilla |


### Setup Wildfly for Keycloak
Download WildFly OpenID Connect adapter distribution from [keycloak.org](keycloak.org) and unpack it into folder `keycloak-wildfly-adapter-dist-3.1.0.Final`

Install the OpenID Connect adapter:

```
cd keycloak-wildfly-adapter-dist-3.1.0.Final 
docker cp . keycloakwildflydemo_wildfly_1:/opt/jboss/wildfly
docker exec -it keycloakwildflydemo_wildfly_1 /opt/jboss/wildfly/bin/jboss-cli.sh --file=wildfly/bin/adapter-install-offline.cli
```

Configure vanilla app to use Keycloak for authentication. 

SSH into Wildfly container:
```
$ docker exec -it keycloakwildflydemo_wildfly_1 bash
```
Edit `/opt/jboss/wildfly/standalone/configuration/standalone.xml` file, replace
```
<subsystem xmlns="urn:jboss:domain:keycloak:1.1"/>
```
with
```
<subsystem xmlns="urn:jboss:domain:keycloak:1.1">
    <secure-deployment name="vanilla.war">
        <realm>demo</realm>
        <auth-server-url>http://keycloak:8080/auth</auth-server-url>
        <public-client>true</public-client>
        <ssl-required>EXTERNAL</ssl-required>
        <resource>vanilla</resource>
    </secure-deployment>
</subsystem>
```
, which is copied and modified from client `vanilla`'s Installation tab in Keycloak.

Now you can log in Vanilla app with Keycloak.










