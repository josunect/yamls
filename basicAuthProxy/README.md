This is an example to create a proxy with basic auth for Kiali.

Create the auth data: 

`htpasswd -nb myuser myawd | base64 -w0`

And modify in the secret yaml file. 

```
curl -fsSL https://raw.githubusercontent.com/josunect/yamls/refs/heads/main/basicAuthProxy/secret.yml \
| sed "s|<BASE64_OUTPUT>|$(htpasswd -nb myuser mypwd | base64 -w0)|g" \
| kubectl apply -f -
kubectl apply -f https://raw.githubusercontent.com/josunect/yamls/refs/heads/main/basicAuthProxy/configMap.yml
kubectl apply -f https://raw.githubusercontent.com/josunect/yamls/refs/heads/main/basicAuthProxy/deployment.yml
kubectl apply -f https://raw.githubusercontent.com/josunect/yamls/refs/heads/main/basicAuthProxy/service.yml

```
