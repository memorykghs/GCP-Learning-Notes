* `gcloud auth list`
```
Credentialed accounts:
 - <myaccount>@<mydomain>.com (active)
```

`gclod config list project`
```
[core]
project = <project_ID>
```

`docker run hellow-world`
local 端沒找到會在 docker hub 上面找 image

`docker images`
```
student_00_d0ffd6a413f0@cloudshell:~ (qwiklabs-gcp-00-e2383957ea8e)$ docker images
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
hello-world   latest    d1165f221234   3 months ago   13.3kB
```

* `docker ps` / `docker ps -a`
查看正在 run 的 container ( `-a` 為 all )。
```
student_00_d0ffd6a413f0@cloudshell:~ (qwiklabs-gcp-00-e2383957ea8e)$ docker ps -a
CONTAINER ID   IMAGE         COMMAND    CREATED              STATUS                          PORTS     NAMES
6b48b411069a   hello-world   "/hello"   About a minute ago   Exited (0) About a minute ago             xenodochial_dewdney
bcf0b29b6bbd   hello-world   "/hello"   10 minutes ago       Exited (0) 10 minutes ago                 cranky_snyder
```
`Names` 也可以透過 `docker run --name [container-name] hellow-world` 來進行命名。

## Build
`mkdir test && cd test`

```
cat > Dockerfile <<EOF
# Use an official Node runtime as the parent image
FROM node:6

# Set the working directory in the container to /app
WORKDIR /app

# Copy the current directory contents into the container at /app
ADD . /app

# Make the container's port 80 available to the outside world
EXPOSE 80

# Run app.js using node when the container launches
CMD ["node", "app.js"]
EOF
```

The initial line specifies the base parent image, which in this case is the official Docker image for node version 6.
In the second, we set the working (current) directory of the container.
In the third, we add the current directory's contents (indicated by the "." ) into the container.
Then we expose the container's port so it can accept connections on that port and finally run the node command to start the application.


```
cat > app.js <<EOF
const http = require('http');

const hostname = '0.0.0.0';
const port = 80;

const server = http.createServer((req, res) => {
    res.statusCode = 200;
      res.setHeader('Content-Type', 'text/plain');
        res.end('Hello World\n');
});

server.listen(port, hostname, () => {
    console.log('Server running at http://%s:%s/', hostname, port);
});

process.on('SIGINT', function() {
    console.log('Caught interrupt signal and will exit');
    process.exit();
});
EOF
```

`docker build -t node-app:0.1 .`
其中 `-t` 代表 tag，用來 tag 在這個 image 名字後面打上標籤，以這個例子來說，這個 image 名稱為 `node-app`，tag 則是 `0.1`。一般來說建議會在每字 build image 時打上標籤，如果沒有標籤的話會一律被視為最新版 ( `latest` )，這樣將會很難去找問題到底出在哪個版。
