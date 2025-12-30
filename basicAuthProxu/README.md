This is an example to create a proxy with basic auth for Kiali.

Create the auth data: 

htpasswd -nb myuser myawd | base64 -w0

And modify in the secret yaml file. 
