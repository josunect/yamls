Generate certificates: 

```
TMPDIR=/tmp/grafana-bearer-proxy-tls
rm -rf "$TMPDIR" && mkdir -p "$TMPDIR"

cat >"$TMPDIR/openssl.cnf" <<'EOF'
[ req ]
distinguished_name = req_distinguished_name
x509_extensions = v3_req
prompt = no

[ req_distinguished_name ]
CN = grafana-bearer-proxy.istio-system.svc.cluster.local

[ v3_req ]
keyUsage = critical, digitalSignature, keyEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names

[ alt_names ]
DNS.1 = grafana-bearer-proxy
DNS.2 = grafana-bearer-proxy.istio-system
DNS.3 = grafana-bearer-proxy.istio-system.svc
DNS.4 = grafana-bearer-proxy.istio-system.svc.cluster.local
EOF

openssl req -x509 -newkey rsa:2048 -nodes \
  -days 365 \
  -keyout "$TMPDIR/tls.key" \
  -out "$TMPDIR/tls.crt" \
  -config "$TMPDIR/openssl.cnf"
``` 
  
Verify the cert

```
openssl x509 -in "$TMPDIR/tls.crt" -noout -subject -issuer -ext subjectAltName
```

In tls.cert: 

`cat "$TMPDIR/tls.crt"`

In tls.key: 

`cat "$TMPDIR/tls.key"`

Apply the yaml: 
`kubectl apply -f grafana.yaml`

Port forward the service: 
`kubectl -n istio-system port-forward svc/grafana-proxy 30001:3000`

Test:
`curl -k -i -H "Authorization: Bearer kiali-test-token" https://localhost:30001/`
