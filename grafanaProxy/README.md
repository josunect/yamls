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

Create the Kubernetes TLS secret (IMPORTANT: not in `grafana.yaml`):

```
kubectl -n istio-system create secret tls grafana-proxy-tls \
  --cert="$TMPDIR/tls.crt" \
  --key="$TMPDIR/tls.key" \
  --dry-run=client -o yaml | kubectl apply -f -
```

Apply the yaml:
`kubectl apply -f grafana.yaml`

Port forward the service:
`kubectl -n istio-system port-forward svc/grafana-proxy 30001:3000`

Test (token by default: `mi-token-secreto`):

`curl -k -i -H "Authorization: Bearer mi-token-secreto" https://localhost:30001/`

## Patch the Kiali CR

```
kubectl -n istio-system patch kiali NAME --type merge -p '{
  "spec": {
    "external_services": {
      "grafana": {
        "enabled": true,
        "external_url": "https://grafana-proxy.istio-system:3000",
        "internal_url": "https://grafana-proxy.istio-system:3000",
        "auth": {
          "type": "bearer",
          "token": "secret:grafana-bearer-token:TOKEN",
          "use_kiali_token": false,
          "insecure_skip_verify": true
        }
      }
    }
  }
}'
```
