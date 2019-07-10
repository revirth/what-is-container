### Basic

```bash
#show containers
docker ps

#kill container
docker kill `container id`

#stop all containers by force
docker kill $(docker ps -q)

#purge the rest
docker system prune

#remove image
docker rmi `image name`

#run bash command in container
docker exec -it `container name` /bin/bash
```



## Deployment
- Scalability - docker run

- Standardity - every deployment process is the same, what ever it is (nodejs, go, php, etc) 

- Image - create an image, save the image, load the image in server

- Configuration - containers work by config variables like `MYSQL_PASSWORD=password` 

- Resources - container's data will be initialized when it starts, so it needs external storage like S3


## Examples

### Redis

```bash
docker run -d -p 1234:6379 redis

# redis test
$ telnet localhost 1234
set mykey hello
+OK
get mykey
$5
hello
```



### MySQL (for Wordpress)

```bash
 docker run -d -p 3306:3306 \
  --name wordpressdb \
  --restart always \
  -v /var/wordpress_db:/var/lib/mysql \
  -e MYSQL_ROOT_PASSWORD=123456 \
  -e MYSQL_DATABASE=wordpress \
  mysql:5.7
```

### Wordpress

```bash
docker run -d \
 -e WORDPRESS_DB_PASSWORD=123456 \
 --name wordpress \
 --link wordpressdb:mysql \
 --restart always \
 -p 8080:80 \
 wordpress
```

### Tensorflow

```bash
docker run -d \
 -p 8888:8888 \
 -p 6006:6006 \
teamlab/pydata-tensorflow:0.1
```


### Ref
- https://www.44bits.io/ko/post/why-should-i-use-docker-container
 ![](https://static.hubtee.com/files/cdf/cdf3ae0a6ca574c036ba35e3957f100d7d77bb6e4eed91ccb065838441188266.m.png)
 ![](https://static.hubtee.com/files/548/54855bc2f1cbd3143832027cbbe7e651cee964e8ac83a9d86e93b75c06bccb06.m.png)
 

