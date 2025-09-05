```
DOMAIN="web.test.org"
COUNTRY="HR"
STATE="Grad Zagreb"
CITY="Zagreb"
ORG="Security"
ORG_UNIT="IT"
EMAIL="miha@test.org"

openssl req -x509 -nodes -days 365 \
  -newkey rsa:2048 \
  -keyout web/crts/certificate.key \
  -out web/crts/certificate.crt \
  -subj "/C=$COUNTRY/ST=$STATE/L=$CITY/O=$ORG/OU=$ORG_UNIT/CN=$DOMAIN/emailAddress=$EMAIL"
```
