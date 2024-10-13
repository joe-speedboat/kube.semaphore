# Setup Semaphore Ansible WebUI on k3s
## Prepare config
```
genpasswd() {
   local l=$1
   [ "$l" == "" ] && l=16
   tr -dc A-Za-z0-9_., < /dev/urandom | head -c ${l} | xargs
}

# Semaphore namespace
NAMESPACE=semaphore

# Semaphore Public URL
SEMAPHORE_HOSTNAME=semaphore.domain.tld

# semaphore admin details
SEMAPHORE_ADMIN_NAME='admin'
SEMAPHORE_ADMIN='admin'
SEMAPHORE_ADMIN_EMAIL='admin@domain.tld'

# Generate MySQL password (used for both MYSQL_PASSWORD and SEMAPHORE_DB_PASS)
MYSQL_PASSWORD=$(genpasswd)

# Generate Semaphore admin password
SEMAPHORE_ADMIN_PASSWORD=$(genpasswd)

# Generate Semaphore access key encryption key
SEMAPHORE_ACCESS_KEY_ENCRYPTION=$(head -c32 /dev/urandom | base64)


# convert secrets
MYSQL_PASSWORD_BASE64=$(echo -n "$MYSQL_PASSWORD" | base64)
SEMAPHORE_ADMIN_PASSWORD_BASE64=$(echo -n "$SEMAPHORE_ADMIN_PASSWORD" | base64)
SEMAPHORE_ACCESS_KEY_ENCRYPTION_BASE64=$(echo -n "$SEMAPHORE_ACCESS_KEY_ENCRYPTION" | base64)

```

## create runtime config from template
```
test -d runtime && echo ERROR remove ./runtime dir first
test -d ./runtime && exit 1
mkdir ./runtime
for y in *.yml
do
  cp -av $y ./runtime/$y
  sed -i "s/__NAMESPACE__/$NAMESPACE/g"                                             ./runtime/$y
  sed -i "s/__SEMAPHORE_HOSTNAME__/$SEMAPHORE_HOSTNAME/g"                           ./runtime/$y
  sed -i "s/__SEMAPHORE_ADMIN_NAME__/$SEMAPHORE_ADMIN_NAME/g"                       ./runtime/$y
  sed -i "s/__SEMAPHORE_ADMIN__/$SEMAPHORE_ADMIN/g"                                 ./runtime/$y
  sed -i "s/__SEMAPHORE_ADMIN_EMAIL__/$SEMAPHORE_ADMIN_EMAIL/g"                     ./runtime/$y
  sed -i "s/__MYSQL_PASSWORD__/$MYSQL_PASSWORD/g"                                   ./runtime/$y
  sed -i "s/__SEMAPHORE_ADMIN_PASSWORD__/$SEMAPHORE_ADMIN_PASSWORD/g"               ./runtime/$y
  sed -i "s/__SEMAPHORE_ACCESS_KEY_ENCRYPTION__/$SEMAPHORE_ACCESS_KEY_ENCRYPTION/g" ./runtime/$y
  sed -i "s/__MYSQL_PASSWORD_BASE64__/$MYSQL_PASSWORD_BASE64/g"                     ./runtime/$y
  sed -i "s/__SEMAPHORE_ADMIN_PASSWORD_BASE64__/$SEMAPHORE_ADMIN_PASSWORD_BASE64/g" ./runtime/$y
  sed -i "s/__SEMAPHORE_ACCESS_KEY_ENCRYPTION_BASE64__/$SEMAPHORE_ACCESS_KEY_ENCRYPTION_BASE64/g" ./runtime/$y
done

echo "# current vars
NAMESPACE=$NAMESPACE
SEMAPHORE_HOSTNAME=$SEMAPHORE_HOSTNAME
SEMAPHORE_ADMIN_NAME=$SEMAPHORE_ADMIN_NAME
SEMAPHORE_ADMIN=$SEMAPHORE_ADMIN
SEMAPHORE_ADMIN_EMAIL=$SEMAPHORE_ADMIN_EMAIL
MYSQL_PASSWORD=$MYSQL_PASSWORD
SEMAPHORE_ADMIN_PASSWORD=$SEMAPHORE_ADMIN_PASSWORD
SEMAPHORE_ACCESS_KEY_ENCRYPTION=$SEMAPHORE_ACCESS_KEY_ENCRYPTION
MYSQL_PASSWORD_BASE64=$MYSQL_PASSWORD_BASE64
SEMAPHORE_ADMIN_PASSWORD_BASE64=$SEMAPHORE_ADMIN_PASSWORD_BASE64
SEMAPHORE_ACCESS_KEY_ENCRYPTION_BASE64=$SEMAPHORE_ACCESS_KEY_ENCRYPTION_BASE64
" > ./runtime/env.sh

ls -l ./runtime

```
## setup application
```
echo "
DONE

all templates created in ./runtime
please review them and apply config

for y in *.yml
do
  echo \$y
  kubectl apply -f \$y
done"

```

