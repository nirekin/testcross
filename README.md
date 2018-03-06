# Go compliation and image generation with docker

Sources:
https://medium.com/travis-on-docker/how-to-dockerize-your-go-golang-app-542af15c27a2

***

# Note is you use Docker Toolbox for Windows

Because I was using Docker Toolbox for Windows my project need to be in `C:\Users`. 
This is due to the fact that the toolbox has limited access to Windows resources.

It's kind of weird but if you are outside of `c:\Users` everything seems to work fine until the compilation time when you get an error message saying that there is no go files to compile. Just because nothing has been mouted into the volume...

***

# Build Inside The Dockerfile
#### (Fat image version )

Docker file : `BigDockerfile`

```
FROM iron/go:dev
WORKDIR /app
ENV SRC_DIR=/go/src/github.com/nirekin/testcross/
# Dependencies should be resolved first
ADD . $SRC_DIR
# Build it:
RUN cd $SRC_DIR; go build -o myapp; cp myapp /app/
ENTRYPOINT ["./myapp"]
```

Once the dependencies has been resolved, we just have to run the dockerfile `docker build -t testcross/fat -f BigDockerfile .`

Created image :

```sh
$ docker images
REPOSITORY                      TAG                 IMAGE ID            CREATED              SIZE
testcross/fat                   latest              cab15046b040        About a minute ago   451MB
```

___
# Build Outside the Dockerfile
#### (Slim image version )



Docker file : `Dockerfile`

```
# iron/go ( alpine with ca-certificates )
FROM iron/go
WORKDIR /app
# Now just add the binary into the image
ADD slimapp /app/
ENTRYPOINT ["./slimapp"]
```

First, once the dependencies has been resolved, we compile into the a container

Using `iron/go:dev`:
```sh
docker run --rm -v "$PWD":/go/src/github.com/nirekin/testcross -w /go/src/github.com/nirekin/testcross iron/go:dev go build -o slimapp
```

Or using an identified version of `Go`:
```sh
docker run -d --rm -v "$PWD":/go/src/github.com/nirekin/testcross -w /go/src/github.com/nirekin/testcross golang:1.8 go build -v - o slimapp
```

Then we run the dockerfile `docker build -t testcross/slim -f Dockerfile .`

Created image:

```sh
$ docker images
REPOSITORY                      TAG                 IMAGE ID            CREATED              SIZE
testcross/slim                  latest              4e48bffc3ea6        19 hours ago         14.1MB
```

***

# Resolve dependencies


### With DEP
```sh
# Init
docker run -it --rm -v $PWD:/go/src/github.com/nirekin/testcross -w /go/src/github.com/nirekin/testcross lushdigital/docker-golang-dep init
# Ensure
docker run -it --rm -v $PWD:/go/src/github.com/nirekin/testcross -w /go/src/github.com/nirekin/testcross lushdigital/docker-golang-dep ensure
```


### With GLIDE
```sh
# Init
docker run --rm -it -v $PWD:/go/src/github.com/nirekin/testcross -w /go/src/github.com/nirekin/testcross treeder/glide init
# Say No to the question it asks, then:
# Update
docker run --rm -it -v $PWD:/go/src/github.com/nirekin/testcross -w /go/src/github.com/nirekin/testcross treeder/glide update
```

Because of the question asked It can be tricky to use `Glide` into a script in charge of resolving dependencies and generated the image... ( Investigate more on this... ) 