# WildFly JWT Demo (WIP)

Example of OpenID secured Client with JWT authentication. JWTs are issued by Keycloak and contain
claims with general user information as well as current user roles.

## Run Keycloak
```
docker run --rm  \
   --name keycloak \
   -e KEYCLOAK_USER=admin \
   -e KEYCLOAK_PASSWORD=admin \
   -e KEYCLOAK_IMPORT=/tmp/quarkus-realm.json  -v /tmp/quarkus-realm.json:/tmp/quarkus-realm.json \
   -p 8180:8180 \
   -it quay.io/keycloak/keycloak:7.0.1 \
   -b 0.0.0.0 \
   -Djboss.http.port=8180 \
   -Dkeycloak.profile.feature.upload_scripts=enabled  
```

Copy quarkus-realm.json in /tmp

## Start WildFly
```
./standalone.sh
```

## Deploy the application
```
mvn install wildfly:deploy
```

## Test the application
```

# Get token for an user belonging to "user" group
export TOKEN=$(\
curl -L -X POST 'http://localhost:8180/auth/realms/quarkus-realm/protocol/openid-connect/token' \
-H 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'client_id=quarkus-client' \
--data-urlencode 'grant_type=password' \
--data-urlencode 'client_secret=mysecret' \
--data-urlencode 'scope=openid' \
--data-urlencode 'username=test' \
--data-urlencode 'password=test'  | jq --raw-output '.access_token' \
 )


# Echo token
JWT=`echo $TOKEN | sed 's/[^.]*.\([^.]*\).*/\1/'`
echo $JWT | base64 -d | python -m json.tool

# Call method which belongs to "Admin" (should fail)
curl -H "Authorization: Bearer $TOKEN" http://localhost:8080/jwt-demo-1.0.0-SNAPSHOT/rest/customers/goadmin
```

