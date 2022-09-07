CTRL+SHIFT+V\
 1----solid understand from image docker

## 2---images and containers

✅image include every thing of application need to run \
✔a cut down os ✔third party ✔application file ✔environment variable\
 so image contain all the file of configuration setting needed to run app once we have image we can start container from it
✅containers a container like a virtual machine \
✔provides an isolated environment ✔can be stopped & restarted ✔is just a process which is provided by image\
each container isolated environment \
a container gets file sys at image each container has layout layer \

## 3----file simple react

install node \
npm install\
npm start\

## 4----docker file instruction

a docker file contains instructions for building an image \
✅FROM base image \
✅WORKDIR specify your workspace dir all file command working in dir\
✅COPY copy file as dir \
✅ADD\
✅RUN\
✅ENV set eviremonet variable \
✅EXPOSE telling docker and container starting given port\
✅USER limited privileges\
✅CMD command should be executed let me start container\
✅ENTRYPOINT\

## 5---choosing right base \

go to [samples docker](https://docs.docker.com/samples/)
django FROM python:3
react FROM node:latest
search node python and with combine os in [docker hub](https://hub.docker.com/) and go tag tab<br/>
`FROM node:14.16.0-alpine3.13` <br/>
we gonna build the image\
`docker build -t react-app .` <br/>
we see all image on docker \
`docker image ls ` <br/>
`docker run -it react-app` <br/>

```js
const x = 1;
console.log(x);
```

for exit this workspace press ctrl+c\
`docker run -it react-app bash` <br/>
above command show `MODULE_NOT_FOUND`
because alpine linux doesn't come with bash is very small linux
its comes shell is original program
`docker run -it react-app sh` <br/>

## 6------------ copy and dir

for that me have this instruction copy the other ADD
they have the exact syntax ADD has couple of feature
we this we can COPY more of more file for dir from current dir into the image\
we can not anything outside of dir here the reason let me execute\
`docker build -t react-app .` <br/>
look at the last argument `.` a period means the current dir so let me execute this command docker client sense the current dir to docker engine this is call the build context so docker client sense to build context to docker engine \
example : `COPY package*.json readme.md /app/ `<br/>
package.json and package-lock.json and readme.md make image to app dir
**when** more than file copy app to be `/app/` not `/app` <br/>
**package.json** include dependency and version what actual install version in machine probably little bit different we see package.json\
**package-lock.json** keep track exact version of dependency install on machine \
what if copy everything in current dir to app dir we used of `.`[period]
`COPY . /app` we also use relative path if set `WORKDIR` first\

```docker
FROM node:14.16.0-alpine3.13
WORKDIR /app
COPY . .
```

if can copy a file has error ❌ `COPY hello.txt .` <br/>
this time we can use array form ✔ `COPY ["hello.txt","."]` <br/>
we also use ADD that exact syntax same COPY but ADD has two additional feature with ADD add from url `ADD http://...file.json .` so if access this file we can add to image the other feature if you pass
a compress file ADD automatically uncompressed into dir `ADD file.zip .` <br/>
best practice is use COPY is straight forward
so let build\
` docker build -t react-app .` <br/>
all file and context sent to docker engine  
 `docker run -it react-app sh` and `/app # ls` <br/>

## 7-----excluding file

if transfer node_module to docker engine causes of eight image very mass and speed of build image slow  
we can to ignore node_module till user in your machine write `npm install ` <br/>
needed to make .dockerignore and inside file write `node_module/` like .gitignore

and other machine needed to
`docker run -it react-app sh` and `/app # ls` then `npm install`

## 8--- running command

RUN command execute any command thats normally execute in terminal session\
 `RUN npm install `
or `RUN apt install python` <br/>
❌alpine linux doesn't have apt package manger rather than have apk

```docker
# make echo node_module/ > .dockerignore
FROM node:14.16.0-alpine3.13
WORKDIR /app
COPY . .
RUN npm install
```

## 9---setting environment variable

lets to front of web application need to talk backend with api
good often we set url api using and env variable  
 `ENV API_URL=http://api.myapp.com/` another syntax `ENV API_URL http://api.myapp.com/` use =[equal sign ] or use space<br/>

`docker run -it react-app sh` <br/>
`printenv` <br/>

## 10 ---- Exposing port

`EXPOSE 3000`
when we run the application inside docker container this port open the container so on the same machine multiple container running same image all the container listening port 3000 port 3000 on the host its not going to automatically map the container

```docker
# make echo node_module/ > .dockerignore
FROM node:14.16.0-alpine3.13
WORKDIR /app
COPY . .
RUN npm install
EXPOSE 3000
```

## 10---setting the user

as talk about user by default docker run on application with root user has the highest privilege so that open security sys run the application by regular user limited privileges by before doing in dockerfile
lets open the shell session in alpine linux and play few command
take this approach to implements <br />
`docker run -it alpine` <br/>
` useradd` <br/>
-S create a system user
we gonna use the sys user because the user is not real user just running in our application before using this command we have to create group so we can add user to that group  
`addgroup app` \
adduser -S sys user -G supplementary group app name group app name user \
`adduser -S -G app app `
show user of app group \
`groups app` \
we gonna combine two command \
`addgroup app && adduser -S -G app emilia` \
show user of app group \
`groups app` \

```docker
# make echo node_module/ > .dockerignore
FROM node:14.16.0-alpine3.13
WORKDIR /app
COPY . .
RUN npm install
EXPOSE 3000
RUN addgroup app && adduser -S -G app emilia
USER emila
```

all with this command will be educated by emilia user\
and rebuild image 1 `docker build -t react-app .`\
`docker run -it react-app sh` \
`/emilia $ whoami` \
`/emilia $ ls -l` \
\- rw- r--[primary group] r--[other group] \
all this file owned by this root user root user can write and permission
current user its run this session is emilia user so this user is third group(other group) thats mean emilia user cant write this files\

## 12---defining entrypoint

`docker run react-app`\
 `docker run react-app npm start`\
 we have permission error look at the docker file we set the user at the end so first root privileges this command done then switched the regular user
so we should move on top this command

```docker
# make echo node_module/ > .dockerignore
FROM node:14.16.0-alpine3.13
RUN addgroup app && adduser -S -G app emilia
USER emila
WORKDIR /app
COPY . .
RUN npm install
EXPOSE 3000
```

so back in terminal \
`docker build -t react-app .`\
port 3000 into container not localhost
`docker run react-app npm start`\
rather than type command `npm start ` add this command to dockerfile \

```docker
# make echo node_module/ > .dockerignore
FROM node:14.16.0-alpine3.13
RUN addgroup app && adduser -S -G app emilia
USER emila
WORKDIR /app
COPY . .
RUN npm install
EXPOSE 3000
CMD npm start
```

whats different between CMD with RUN because both with execute command \
run instruction is build time instruction ,so this is executed at time of built in image so built in the image where installing npm dependency as in dependency are store in the image in contrast
command instruction is the run time instruction so executed when started container . \
 **command instruction is two form**

```docker
#shell form   run seprate shell
#/bin/sh linux
#cmd  win
CMD npm start
#Exec form
CMD ["npm","start"]
```

best practice form exec form because this form execute directly there is noe need to spin up shell proses also easier and faster \
so here command insruction we also have another instruction ENTRYPOINT which is similar to CMD shell form `ENTRYPOINT npm start ` and exec form `ENTRYPOINT ["npm", "start"]`\
call ``docker run react-app --entrypoint`\`
select one of them entrypoint or cmd

## 13---speeding up Builds

every time small change we have re build and wait long time how can optimize this\
first thing understand consept of layer of docker
you can think layer small file sys only include modify file \

```docker
# make echo node_module/ > .dockerignore
FROM node:14.16.0-alpine3.13
RUN addgroup app && adduser -S -G app emilia
USER emila
WORKDIR /app
COPY . .
RUN npm install
EXPOSE 3000
CMD ["npm" ,"start"]

```

so when dockerfile build in image for us execute each of instruction in create a new layer \
so first instruction docker is take node image and put it in layer accurately node own include several layer \
then second u\instruction add group in user once again create new layer because as you saw add user in group something is written file sys
let quick see layer\
`docker history react-app`

```docker
# make echo node_module/ > .dockerignore
FROM node:14.16.0-alpine3.13  #reuse cache
RUN addgroup app && adduser -S -G app emilia #reuse cache
USER emila
WORKDIR /app
COPY . .  #always rebuild
RUN npm install  #if COPY always rebuild this command always working
EXPOSE 3000
CMD ["npm" ,"start"]


```

we optimize this command COPY with separating package\*.json from other

```docker
# make echo node_module/ > .dockerignore
FROM node:14.16.0-alpine3.13  #reuse cache
RUN addgroup app && adduser -S -G app emilia #reuse cache
USER emila
WORKDIR /app
COPY package*.json .  #reuse cache when not change
RUN npm install  # when above instruction not rebuild this command not run
COPY . .  #always rebuild
EXPOSE 3000
CMD ["npm" ,"start"]


```

## 14--- removing image

`docker images`\
we get rid of them
`docker image prune`\
also container \
`docker pa -a` \
tons of container is stop lets get rid of them\
`docker container prune`\
if remove main images [have name]\
`docker image rm TagName` or\
`docker image rm ImageId` if have mutilple image\
`docker image rm ImageId ImageId2 ImageId3` \

## 15---tagging images

this feature for troubleshoot thats easy to find exact version\
`docker build -t react-app:1.0.0 .`\
`docker build -t react-app:1.2.0 .`\
by default tag is latest lets see\
`docker images`\
for remove a specific version of image\
`docker image remove react-app:1.0.0` \
`docker images`\
for change tag thats before selected tag \
`docker image tag react-app:latest react-app:1 `\
for change a tag of list image \
`docker image tag b06 react-app:latest`\

## 16--- sharing image

create account in hub [docker](hub.docker.com) \
click to create repository make a name and select between private and public then click create\
go to machine and \
`docker image` check see\
`docker login` write user ans password\
and `docker push userAccountDockerHub/react-app:1.2.0`\
lets little change and \
`docker build -t react-app:1.3.0`\
`docker images`\
`docker image tag b06 userAccountDockerHub/react-app:1.3.0`\
`docker push userAccountDockerHub/react-app:1.3.0`\

## 17----saving and loading images

for output from project for using other machine
`docker image save --help`\
`docker image save -o react-app.tar react-app:1.3.0` tar file in linux like zip in window\
for load tar file in other machine
for test this version thats save term of tar is deleting `docker image rm react-app:1.3.0`\
notice tar file exist in workspace dir
`docker image load -i react-app.tar`
