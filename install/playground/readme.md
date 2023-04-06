
# Meshery Playground Bare Metal Configuration 

### DNS
playground.meshery.io - 147.28.141.9

### Host: meshery02

#### Static IP address configuration
File: `/etc/netplan/00-elastic.yaml`
```
network:
  version: 2
  renderer: networkd
  ethernets:
    lo:
      addresses:
        - 127.0.0.1/8
        - 147.28.141.9/32
```

Affect changes:
`netplan apply`

Needed?
`systemctl restart networking`


### Nginx Configuration for websocket (until annotations issue is solved)
Nginx ingress controller by default (sometimes) doesn't pick up the custom configuration passed to it via annotations so after the nginx pod starts below steps are required to make sure that websocket requests go through.

- exec into the nginx pod and go to /etc/nginx/conf.d.
- There will be a .conf file present there. Take that file out in your local system to edit since the container doesn't have vi or any code editor.
- Add the below location blocks just above the location block for `/`

```
	location /api/system/graphql/query {
	   set $service "meshery"; 
	   proxy_set_header Upgrade $http_upgrade;
	   proxy_http_version 1.1;
	   proxy_set_header X-Forwarded-Host $http_host;
	   proxy_set_header X-Forwarded-Proto $scheme;
	   proxy_set_header X-Forwarded-For $remote_addr;
	   proxy_set_header Host $host;
	   proxy_set_header Connection "upgrade";
	   proxy_cache_bypass $http_upgrade;
	   proxy_pass http://meshery-meshery-playground.meshery.io-meshery-9082;
	 }
	location /api/provider/extension/server/graphql/query {
	   set $service "meshery"; 
	   proxy_set_header Upgrade $http_upgrade;
	   proxy_http_version 1.1;
	   proxy_set_header X-Forwarded-Host $http_host;
	   proxy_set_header X-Forwarded-Proto $scheme;
	   proxy_set_header X-Forwarded-For $remote_addr;
	   proxy_set_header Host $host;
	   proxy_set_header Connection "upgrade";
	   proxy_cache_bypass $http_upgrade;
	   proxy_pass http://meshery-meshery-playground.meshery.io-meshery-9082;
	 }	
```

- In the above configuration, make sure the proxy_pass directive is the same as the one present for the already existing `/` location block. If not, then replace it with the one already present

- Enter `cat > <name of the file>` to get the stdin and then paste the edit file on the console. Exit using cmd+c or cmd+d

- Perform an nginx reload with `nginx -s reload`

- Confirm that the websocket requests are passing through.