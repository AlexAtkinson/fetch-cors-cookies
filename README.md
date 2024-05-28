# fetch-cors-cookies

:clapper:	: YouTube Video: [here](https://www.youtube.com/watch?v=34wC1C61lg0)<br>
:floppy_disk: Self Signed Cert Script: [here](https://gist.github.com/AlexAtkinson/98efb3718e493abd263c32a0cd5032e6) - or just copy/paste the instructions below ( :bulb: edit NAME).
:warning: Works with Debain -- Untested on Mac.'

## Setup your CA & Self-Signed Certs

```bash
NAME='hell.com'

# Setup ROOT CA Cert if missing
openssl genrsa -des3 -out "$(hostname)CA.key" 4096
openssl req -x509 -new -nodes -key "$(hostname)CA.key" -sha256 -days 825 -out "$(hostname)CA.pem"
sudo mkdir "/usr/local/share/ca-certificates/$(hostname)"
sudo cp "$(hostname)CA.pem" "/usr/local/share/ca-certificates/$(hostname)/$(hostname)CA.crt"
sudo update-ca-certificates

# Setup server certs
if [[ ! -d "certs/$NAME" ]]; then
  mkdir "certs/$NAME"
else
  echo "The directory certs/$NAME already exists. Delete this directory before retrying."
  exit 1
fi

openssl genrsa -out "certs/$NAME/server.key" 4096
openssl req -new              \
-key "certs/$NAME/server.key" \
-subj "$SUBJ"                 \
-out "certs/$NAME/server.csr"

cat <<EOF >>"certs/$NAME/cert.ext"
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
subjectAltName = @alt_names
[alt_names]
DNS.1 = $NAME
DNS.2 = *.$NAME
IP.1 = 127.0.0.1
EOF

openssl x509 -req -in "certs/$NAME/server.csr" \
-CA "$(hostname)CA.pem"                        \
-CAkey "$(hostname)CA.key"                     \
-CAcreateserial                                \
-days "$DAYS"                                  \
-sha256                                        \
-out "certs/$NAME/server.crt"                  \
-extfile "certs/$NAME/cert.ext"

openssl verify -CAfile "$(hostname)CA.pem" -verify_hostname "$NAME" "certs/$NAME/server.crt"
```

## Setup the fetch-cors-cookies project

1. clone
1. cd fetch-cors-cookies/server
1. cp ~/your/self/signed/cert/server.key server.key
1. cp ~/your/self/signed/cert/server.crt server.crt
1. npm i
1. npm run dev
1. add necessary entries on your hosts file, such as:
   <pre>
   127.0.0.1    hell.com
   127.0.0.1    a.hell.com
   127.0.0.1    b.hell.com
   </pre>
1. Navigate to https://hell.com
1. Verify cookies persist across subdomains/play around
