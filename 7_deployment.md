## 1----deployment

## 2----deployment options

1-single-host-deployment\
 2-cluster deployment [group of server ] \

deploy on single-host is easy that problem in single host on deployment if i server goes offline our application unaccessible and also if our application goes properly hundred and thousand user a single server is not able to handle to bad load thats why we use cluster so cluster is high availability and scalability \
drawn cluster we need to special tools call our workstation tools docker solution \
`docker swarm` as first as i know popular most people use another tool `kubernetes` which is google products

## 3---- getting a virtual private server

need to debit credit card
vps option✔\
digital ocean\
google cloud platform (GCP)\
microsoft Azure\
Amazon web services\

## 4---- install docker machine

once we have server we need tools call docker machine to talk to docker engine on that is server so this way execute docker command in our terminal and our command we send docker engine for server \
 go to the [machine release ](https://github.com/docker/machine/releases) on this page find end of release we just copy command and execute in terminal\

```
$ curl -L https://github.com/docker/machine/releases/download/v0.16.2/docker-machine-`uname -s`-`uname -m` >/usr/local/bin/docker-machine && \
chmod +x /usr/local/bin/docker-machine
```

then check \
 `docker-machine --version`\

## 5---- provisioning a host

in linux machine for break down command we use `\` and for windows machine use backtick \
 see [check driver](https://docs.docker.com/machine/drivers/) this course is digital ocean \
 if not exist list `--driver none` \
 so need to go digital ocean i get access token so docker machine can talk remotely to digital ocean\
we can not latest docker on the server handle it extra option `--engine-install-url`\
server name `vidly`

```
docker-machine create \
> --driver digitalocean \
> --digitalocean-access-token  api-key\
> --engine-install-url "https://release.rancher.com/install-docker/"\
> vidly
```

## 6---connecting to the host

`docker-machine ls`\
 see vidly driver state url \\
ssh secure shell for connect to server \
 `docker-machine ssh vidly`
now we log to our machine \

```
root@vidly:~#ls
root@vidly:~#cd /
root@vidly:~#ls
bin   dev   home  initrd.omg.old    lib64  media opt root
boot  etc   initrd.img  lib         lost+found   mnt  proc  run
root@vidly:~#
```

## 7-defining the production configuration

make separate compose file `docker-compose.prod.yml` name ais arbitrary but name convention this shape\
web-tets remove\
`ports : -80:3000`\

we don't need volume and remove it\
and reach restart policy see [policy](https://docs.docker.com/compose/compose-file/)\

```
restart:"no"
restart:"always"
restart:"no-failure"
restart:"unless-stopped"
```

and select `unless-stopped` and complete docker-compose\

```yml
version:"3.8"
services:
    web:
        build: ./frontend
        ports :
            - 80:3000
        restart:unless-stopped


    api :
        build:
            context:./frontend
        ports :
            - 3001:3000
        restart:unless-stopped
        environment:
            DB_URL:mongo://db/vidly
    db:
        image: mongo:4.0-xenial
        ports :
            - 27017:27017
        volumes:
            - vidly :/data/db
        restart:unless-stopped

    volumes:
        vidly:
```

## 8--- reducing the image size

for reduce file \
in frontend directory ▶first use command npm build till make optimize size file from frontend\
and make file by name `Dockerfile.prod`\

```docker

FROM node:14.16.0-alpine3.13
RUN addgroup app && adduser -S -G app emilia
USER emila
WORKDIR /app
RUN mkdir data
COPY package*.json .
RUN npm install
COPY . .
EXPOSE 3000
CMD:["npm","start"]
```

know in this file make restructure \

```docker
# step 1  build stage give a label  AS build-stage
FROM node:14.16.0-alpine3.13
WORKDIR /app
RUN mkdir data
COPY package*.json .
RUN npm install
COPY . .
RUN: npm run build

# stage 2 : production we need web server  nginx apache ios microsoft
FROM nginx:1.12-alpine
RUN addgroup app && adduser -S -G app emilia
USER emila
COPY --from=build-stage /app/build/usr/share/nginx/html
EXPOSE 80
ENTRYPOINT [""nginx","-g","daemon off;"]
```

build-stage refer to label
this path /usr/share/nginx/html standard fo serve nginx
and terminal run \
`docker build -t vidly_web_opt -f Dockerfile.prod .`\
and see file reduce from 300 mb to 16mb \
go to `docker-compose.prod.yml` file and change \

```yml
version:"3.8"
services:
    web:
        build:
            context:./frontend
            dockerfile:Dockerfile.prod
        ports :
            - 80:80

        restart:unless-stopped


    api :
        build: ./backend
        ports :
            - 3001:3000
        restart:unless-stopped
        environment:
            DB_URL:mongo://db/vidly
    db:
        image: mongo:4.0-xenial
        ports :
            - 27017:27017
        volumes:
            - vidly :/data/db
        restart:unless-stopped

    volumes:
        vidly:
```

and run \
`cd ..`
`docker-compose -f docker-compose.prod.yml build `\
`docker images `
see vidly_web size 16.2 mb

## 9----deploy the application

`docker-machine ls`\
`docker-machine env`\
environment for seeing the environment variable that to set to talk to vidly machine\
above command execute see `eval $(docker-machine env vidly)` means inside $() value pass to eval (execute val) and run command \
`eval $(docker-machine env vidly)`\
another way\
`docker-machine activate vidly`\
then docker client will be talking with docker engine\
`docker images `\
see we dont have any image\
`docker-compose -f docker-compose.prod.yml up -d`\
see permission error saying Missing write access to /pp directory\
 in the older version of Dockerfile not respect the user instruction related `dockerfile backend`\

### dockerfile backend

```docker
FROM node:14.16.0-alpine3.13
RUN addgroup app && adduser -S -G app emilia
USER emila
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3001
```

give to permission with make dir \

```docker
FROM node:14.16.0-alpine3.13
RUN addgroup app && adduser -S -G app emilia
RUN mkdir /app  && chown emila:app  /app
USER emila
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3001
```

chown` user:group /directory`
[change owner =chown] using this command change owner of directory
then run again and rebuild option
`docker-compose -f docker-compose.prod.yml up -d --build`\

## 10--- Troubleshooting deployment issue

`docker-machine ls`\
see vidly copy url column field and paste to browser 104.131.24.150 \
so frontend application not up what about backend application on backend of application have 104.131.24.150:3001/api  
api is running \
`docker ps `\
see image vidly_web in status column continuously restarting by restart policy\
 `docker logs container id `\
 all these repetitive message staring to user directive make sense only if master process permission \
back to docker file in frontend project this is make a custom none root user

```docker
# step 1  build stage give a label  AS build-stage
FROM node:14.16.0-alpine3.13
WORKDIR /app
RUN mkdir data
COPY package*.json .
RUN npm install
COPY . .
RUN: npm run build

# stage 2 : production we need web server  nginx apache ios microsoft
FROM nginx:1.12-alpine
COPY --from=build-stage /app/build/usr/share/nginx/html
EXPOSE 80
ENTRYPOINT [""nginx","-g","daemon off;"]
```

re creating not root user for nginx remove run , and user command and build compose again\
`docker-compose -f docker-compose.prod.yml up --build`\
`docker ps`\
 back to the browser and see 104.131.24.150 frontend but not fetch data \
 open `devtools` of `chrome` and go to network section and click XHR look at api request\
 look at url request see http://localhost:3001/api/movies that is for test go to frontend directory and set environment variable\

```docker
# step 1  build stage give a label  AS build-stage
FROM node:14.16.0-alpine3.13
WORKDIR /app
RUN mkdir data
COPY package*.json .
RUN npm install
COPY . .
ENV REACT_APP_URL=http://104.131.24.150:3001/api/
RUN: npm run build

# stage 2 : production we need web server  nginx apache ios microsoft
FROM nginx:1.12-alpine
COPY --from=build-stage /app/build/usr/share/nginx/html
EXPOSE 80
ENTRYPOINT [""nginx","-g","daemon off;"]
```

so rebuild /
`docker-compose -f docker-compose.prod.yml up --build`\
back to browser see fetch data
see running machine \
`docker-machine ls`\
`docker-machine env vidly`\
`eval $(ocker-machine env vidly)`\
`docker-compose up` \


## 11---- publishing change  
`docker ps`\
what version of web ? we dont know which version is running ?\
early  build instruction we also add image property  
```yml
version:"3.8"
services:
    web:
        build:
            context:./frontend
            dockerfile:Dockerfile.prod

        image:vidly_web:1
        ports :
            - 80:80

        restart:unless-stopped


    api :
        build: ./backend
        image:vidly_web:1
        ports :
            - 3001:3000
        restart:unless-stopped
        environment:
            DB_URL:mongo://db/vidly
    db:
        image: mongo:4.0-xenial
        ports :
            - 27017:27017
        volumes:
            - vidly :/data/db
        restart:unless-stopped

    volumes:
        vidly:
```
save change   and build \
`docker-compose -f docker-compose.prod.yml up -d --build`\
`docker ps`\
for  another version by manually  `image:vidly_web:2` and ...\
