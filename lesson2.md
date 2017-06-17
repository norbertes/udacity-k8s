# Simple Dockerfile
```
norbert-google-cloud@ubuntu:~/go/src/github.com/udacity/ud615$ cat app/hello/Dockerfile
> FROM alpine:3.1 # best base
> MAINTAINER Kelsey Hightower <kelsey.hightower@gmail.com>
> ADD hello /usr/bin/hello # take file/path from host and add to container
> cd ud615/app/monolith
go get -u
go build --tags netgo --ldflags '-extldflags "-lm -lstdc++ -static"'ENTRYPOINT ["hello"] # executeable
```

# Create An Image

### Commands to run on the VM Instance
##### Install Go
```sh
wget https://storage.googleapis.com/golang/go1.6.2.linux-amd64.tar.gz
rm -rf /usr/local/bin/go
sudo tar -C /usr/local -xzf go1.6.2.linux-amd64.tar.gz
export PATH=$PATH:/usr/local/go/bin
export GOPATH=~/go
```

##### Get the app code
```sh
mkdir -p $GOPATH/src/github.com/udacity
cd $GOPATH/src/github.com/udacity
git clone https://github.com/udacity/ud615.git
Build a static binary of the monolith app
cd ud615/app/monolith
go get -u
go build --tags netgo --ldflags '-extldflags "-lm -lstdc++ -static"'
```

Why did you have to build the binary with such an ugly command line?

You have to explicitly make the binary static. This is really important in the Docker community right now because alpine has a different implementation of libc. So your go binary wouldn't have had the lib it needed if it wasn't static. You created a static binary so that your application could be self-contained.

##### Create a container for the app
Look at the Dockerfile

```sh
cat Dockerfile
```

Build the app container
```sh
sudo docker build -t monolith:1.0.0 .
```

List the monolith image
```sh
sudo docker images monolith:1.0.0
Run the monolith container and get it's IP
sudo docker run -d monolith:1.0.0
sudo docker inspect <container name or cid>
```
or
```sh
CID=$(sudo docker run -d monolith:1.0.0)
CIP=$(sudo docker inspect --format '{{ .NetworkSettings.IPAddress }}' ${CID})
```
Test the container
```sh
curl <the container IP>
```
or
```sh
curl $CIP
```

#### Important note on security
If you are tired of typing "sudo" in front of all Docker commands, and confused why a lot of examples don't have that, please read the following article about implications on security - Why we don't let non-root users run Docker in CentOS, Fedora, or RHEL


# Create docker images for the remaining microservices - auth and hello.
Repeat the steps you took for monolith.

### Build the auth app
```sh
cd $GOPATH/src/github.com/udacity/ud615/app
cd auth
go build --tags netgo --ldflags '-extldflags "-lm -lstdc++ -static"'
sudo docker build -t auth:1.0.0 .
CID2=$(sudo docker run -d auth:1.0.0)
```

### Build the hello app
```sh
cd $GOPATH/src/github.com/udacity/ud615/app
cd hello
go build --tags netgo --ldflags '-extldflags "-lm -lstdc++ -static"'
sudo docker build -t hello:1.0.0 .
CID3=$(sudo docker run -d hello:1.0.0)
```

### See the running containers
```sh
sudo docker ps -a
```

# Push Images
### See all images
```sh
sudo docker images
```

### Docker tag command help
```sh
docker tag -h
```

### Add your own tag
```sh
sudo docker tag monolith:1.0.0 <your username>/monolith:1.0.0
```

For example (you can rename too!)
```sh
sudo docker tag monolith:1.0.0 udacity/example-monolith:1.0.0
```

# Create account on Dockerhub
To be able to push images to Dockerhub you need to create an account there - https://hub.docker.com/register/

### Login and use the docker push command
```sh
sudo docker login
sudo docker push udacity/example-monolith:1.0.0
```

### Repeat for all images you created - monolith, auth and hello!
