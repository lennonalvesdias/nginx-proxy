# Setup

docker rm -f proxy && docker rm -f letsencrypt && docker rm -f portainer

## nginx-proxy
docker volume create certs
docker volume create vhost.d
docker volume create html

``` sh
docker run -d --name proxy -p 80:80 -p 443:443 --restart=unless-stopped --network nginx-proxy -l com.github.jrcs.letsencrypt_nginx_proxy_companion.nginx_proxy=true -v /var/run/docker.sock:/tmp/docker.sock:ro -v certs:/etc/nginx/certs:rw -v vhost.d:/etc/nginx/vhost.d -v html:/usr/share/nginx/html -v `pwd`uploadsize.conf:/etc/nginx/conf.d/uploadsize.conf:ro jwilder/nginx-proxy
```

## nginx-proxy - letsencrypt
``` sh
docker run -d --name letsencrypt --restart=unless-stopped --network nginx-proxy -e NGINX_PROXY_CONTAINER=proxy -v /var/run/docker.sock:/var/run/docker.sock:ro --volumes-from proxy jrcs/letsencrypt-nginx-proxy-companion
```

## portainer
```sh
docker run -d --name portainer -p 9000:9000 --restart=always --network nginx-proxy -e VIRTUAL_HOST=docker.lennon.cloud -e LETSENCRYPT_HOST=docker.lennon.cloud -e LETSENCRYPT_EMAIL=lennonalvesdias@gmail.com -v `pwd`/portainer:/data -v /var/run/docker.sock:/var/run/docker.sock portainer/portainer
```

## docker registry
``` sh
docker run --name registry-auth --entrypoint htpasswd registry:2 -Bbn lennonalvesdias FSuIaRvZihN5mzpIyzA0 > auth/htpasswd
```
``` sh
docker run -d --name registry -p 5000:5000 --restart=always -e "REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd" --net nginx-proxy -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" -e "REGISTRY_STORAGE_DELETE_ENABLED=true" -e "REGISTRY_AUTH=htpasswd" -e VIRTUAL_HOST=registry.lennon.cloud -e LETSENCRYPT_HOST=registry.lennon.cloud -e LETSENCRYPT_EMAIL=lennonalvesdias@gmail.com -v `pwd`/auth:/auth -v `pwd`/registry:/var/lib/registry registry:2
```

# Services In Use

## mongo
``` sh
docker run -d --name mongo -p 27017:27017 --restart=unless-stopped --network nginx-proxy -e MONGO_INITDB_ROOT_USERNAME=lennonalvesdias -e MONGO_INITDB_ROOT_PASSWORD=08TFYGCHKZ85Q3XQO0YQ684RMFJ7L9FC -e VIRTUAL_HOST=mongo.lennon.cloud -e LETSENCRYPT_HOST=mongo.lennon.cloud -e LETSENCRYPT_EMAIL=lennonalvesdias@gmail.com -v `pwd`/mongodata:/data/db mongo:3.6
```

## redis
``` sh
docker run -d --name redis --restart=always --network nginx-proxy redis:latest redis-server --appendonly yes
```

## user
whoiam
sudo su -
nano /etc/ssh/sshd_config
service sshd restart
adduser docker
// server 35.196.236.49
// user docker
// password $V:zU-r{kc{e4~bG
usermod -aG sudo docker
groups docker
nano /etc/sudoers

// github google token a316274c453945bac48922a3bac293a4fa7ea228
// https://blog.ssdnodes.com/blog/self-hosting-handbook/
