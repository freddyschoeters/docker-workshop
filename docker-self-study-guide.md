# Docker self-study

The goal of this document is to provide you with a self-study path explaining some common and basic things about docker.

## How to see which docker images are available

Typicaly images are available in a central repository. The public one is https://hub.docker.com, but for security reasons you are not allowed to directly use these images.

Before you are allowed to use any docker image it must be scrutenized to verify there are no known security vulnerabilities in this image. The images for which this already happened are available in Nexus on http://cipadmin-prod.be.net.intra/nexus/. You can easily search there.

An alternative is to search through this url: http://cipadmin-prod.be.net.intra:8085/v1/search?q=wiremock.

## Test with the alpine image

The first thing to do is getting a simple linux docker container and echo 'hello world' to the standard output. This means pulling the image, and running this image. 

### Pull the image



To pull the alpine docker image to your local machine using the following command:

```bash
$ docker pull cipadmin-prod.be.net.intra:8085/alpine:3.6

3.6: Pulling from alpine
935c89b81420: Pull complete
Digest: sha256:e1e3670ac012eb7e5e0434466569885d8e516158e4ab86dc0577533bf339e6b2
Status: Downloaded newer image for cipadmin-prod.be.net.intra:8085/alpine:3.6
```

The url at the end of the command is build up in the following way: registeryname:port/image:version. 

NOTE: Images within te bank are only available over an insecure connection (http) and not over a secure connection (https). This means we explicitely have to add the url as an insecure registery (see following guide: https://docs.docker.com/registry/insecure/).

You can see which images are available locally using the following command: 

```bash
$ docker images

REPOSITORY                                      TAG                  IMAGE ID            CREATED             SIZE
hello-world                                     latest               2cb0d9787c4d        5 days ago          1.85kB
cipadmin-prod.be.net.intra:8085/doct/wiremock   v0.1_2018_QAP2_SP6   5be08f42a142        6 weeks ago         456MB
cipadmin-prod.be.net.intra:8085/alpine          3.6                  e2cd449cde75        7 months ago        3.97MB
```

### Run the image with echo

To actually do something with the image, you can start it with the following command:

```bash
$ docker run cipadmin-prod.be.net.intra:8085/alpine:3.6 echo "Hello-world"
Hello-world
```

It starts the image and executes echo command.
To check if the image is still running, use

```bash
$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES

$ docker ps -a
CONTAINER ID        IMAGE                                        COMMAND              CREATED             STATUS                      PORTS               NAMES
4e340156800b        cipadmin-prod.be.net.intra:8085/alpine:3.6   "echo Hello-world"   5 minutes ago       Exited (0) 5 minutes ago                        loving_shirley
c480abe6ecaa        hello-world                                  "/hello"             22 minutes ago      Exited (0) 22 minutes ago                       objective_jennings
```

The first command asks for the currently running containers, but there are none. The second command asks for running and recently running containers.

### Make a custom image extending the alpine image

In the previous section we did run the hello-world command using the commandline. Now we are going to make our own fancy hello-world image that does exactly the same.

Create a file called ```Dockerfile``` with your favorite editor and add the following:

```Dockerfile
FROM cipadmin-prod.be.net.intra:8085/alpine:3.6

CMD echo "hello-world"
```

We have to use [`vi`](https://pangea.stanford.edu/computing/unix/editing/viquickref.pdf) to create this file.

Lets see what is in there:

- FROM is used to specify the docker image to start from.
- CMD is used to specify the command to execute when starting this docker image. 

You can build and run this custom image in the following way:

```bash
$ docker build . -t fancy-hello-world:1

Sending build context to Docker daemon    1.2MB
Step 1/2 : FROM cipadmin-prod.be.net.intra:8085/alpine:3.6
 ---> e2cd449cde75
Step 2/2 : CMD echo "hello-world"
 ---> Using cache
 ---> 5fa23bd3ca30
Successfully built 5fa23bd3ca30
Successfully tagged fancy-hello-world:1

$ docker run fancy-hello-world:1

hello-world
```

# Create hello-world webpage

## Manually create the hello-world page

### Pull the Nginx image

```bash
$ docker pull cipadmin-prod.be.net.intra:8085/nginx:1.13.6
```

### Run Nginx
```bash
$ docker run --rm --name nginx -d -p <http_port>:80 cipadmin-prod.be.net.intra:8085/nginx:1.13.6
```

Nginx index page should be available at `<docker_host>:<http_port>`, for example http://localhost:1180

### Create the hello world page

```bash
$ docker exec -it nginx /bin/bash
```

Inside the container:

```bash
$ cd /usr/share/nginx/html
```

Since no editor is avaialbe in the nginx image we have to use a trick to add a new file.

```bash
$ cat > hello.html
```

Copy-paste the following snippet (with the last blank line) and then type `ctrl+c`.

```html
<html>
    <body>
        <h1>Hello World!</h1>
    </body>
</html>

```

Now try to access the page in your browser at `<docker_host>:<http_port>/hello.html`, for example http://locahost:1180/hello.html

### Stop the container

```bash
$ docker kill nginx
```

Now try to run the nginx container again.

```bash
$ docker run --rm --name nginx -d -p <http_port>:80 cipadmin-prod.be.net.intra:8085/nginx:1.13.6
```

- What happen if you try to access the hello world page?
- Why?

## Repeatable creation of hello-world page

### Create a custom docker image

Create a `hello.html` with this content:

```html
<html>
    <body>
        <h1>Hello World!</h1>
    </body>
</html>
```

Create a `Dockerfile` with this content:

```Dockerfile
FROM cipadmin-prod.be.net.intra:8085/nginx:1.13.6

COPY hello.html /usr/share/nginx/html
```

Now we can build a new docker image

```bash
$ docker build . -t nginx-hello-world
```

```bash
$ docker images
REPOSITORY                              TAG                 IMAGE ID            CREATED             SIZE
nginx-hello-world                       latest              e5e28ca8b96d        6 seconds ago       108 MB
cipadmin-prod.be.net.intra:8085/nginx   1.13.6              40960efd7b8f        8 months ago        108 MB
```

### Run the nginx hello world image 

```bash
$ docker run --rm --name nginx-hello-world -d -p <http_port>:80 nginx-hello-world
```

```bash
$ docker logs -f nginx-hello-world
```

### Stop the container

```bash
$ docker kill nginx-hello-world
```

```bash
$ docker run --rm --name nginx-hello-world -d -p <http_port>:80 nginx-hello-world
```

- What happen if you try to access the hello world page?
- Why?

## References

- [Docker Commands](https://docs.docker.com/reference/)