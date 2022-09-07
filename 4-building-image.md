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
`docker build -t react-app .`   <br/>
we see all image on docker \
`docker image ls ` <br/>
`docker run -it react-app` <br/>
```js
const x=1
console.log(x)
```
for exit this workspace  press ctrl+c\
`docker run -it react-app bash` <br/>
above command show `MODULE_NOT_FOUND`
because alpine linux doesn't come with bash is very small linux
its comes shell is original program
`docker run -it react-app sh` <br/>

## 6------------ copy  and dir
for that me have this instruction  copy  the other ADD
they have the exact syntax ADD has couple of feature 
we this we can  COPY  more of more file for dir  from current dir into the image\
we can not anything outside of dir here the reason  let me execute\
`docker build -t react-app .` <br/>
look at the last argument `.` a period means the current dir  so let me execute this command docker client sense the current dir to docker engine this is call the build context so docker client sense to build context to docker engine   \
example : `COPY package*.json readme.md /app/ `<br/>
package.json and package-lock.json and readme.md make image to app dir
**when** more than file copy  app to be `/app/` not `/app`  <br/>
**package.json** include dependency and version what  actual install version in machine  probably little bit different we see package.json\
**package-lock.json**  keep track  exact version of dependency install on machine \
what if copy everything  in current dir  to app dir we used of `.`[period]
`COPY . /app` we also use  relative path if set `WORKDIR` first\
```docker
FROM node:14.16.0-alpine3.13
WORKDIR /app
COPY . .
``` 
if can copy a file   has error ❌   `COPY hello.txt .`  <br/>
this time  we can use array form ✔ `COPY ["hello.txt","."]` <br/>
 we also  use ADD   that exact syntax same COPY  but ADD has two additional feature with ADD  add from url  `ADD http://...file.json .` so if access this file we can add to image the other feature if you pass
 a compress file ADD automatically uncompressed into dir `ADD file.zip .` <br/>
 best practice is use COPY  is straight forward 
so let build\
` docker build -t react-app .`  <br/>
  all file and context sent to docker engine  
  `docker run -it react-app sh`  and `/app # ls`  <br/>

## 7-----excluding file
if transfer node_module to  docker engine  causes of  eight image very mass  and speed of build image  slow  
we can  to ignore  node_module  till user  in your machine write `npm install `  <br/>
needed  to make  .dockerignore   and inside file write `node_module/` like .gitignore

and  other machine  needed to 
`docker run -it react-app sh`  and `/app # ls` then `npm install`

## 8--- running command
 RUN command execute any command thats normally execute in terminal session\
 `RUN npm install   `
or `RUN apt install python` <br/>
❌alpine linux doesn't have apt  package manger  rather than have apk
```docker
# make echo node_module/ > .dockerignore 
FROM node:14.16.0-alpine3.13
WORKDIR /app
COPY . .
RUN npm install 
``` 
## 9---setting environment variable 
 lets to front of web application need to talk  backend with api
 good often we set  url api  using and env variable  
 `ENV API_URL=http://api.myapp.com/` another syntax  `ENV API_URL http://api.myapp.com/`  use =[equal sign ] or use space<br/>

 `docker run -it react-app sh` <br/>
 `printenv`   <br/>

## 10 ---- Exposing port 
`EXPOSE 3000`
when we run the application inside docker container this port open the container so on the same machine multiple container running same image all the container listening port 3000  port 3000 on the host its  not going to automatically  map  the container 
```docker
# make echo node_module/ > .dockerignore 
FROM node:14.16.0-alpine3.13
WORKDIR /app
COPY . .
RUN npm install 
EXPOSE 3000
```

## 10---setting the user
as talk about user by default docker run on application with root user has the highest privilege   so that open security sys run the application by regular user limited privileges by before doing in dockerfile 
lets open the shell session in alpine linux   and play few command
take this approach to implements  <br />
  `docker run -it alpine` <br/>
 ` useradd` <br/>
 -S create a system user
we gonna use the sys user because the user is not real user just running in our application before using this command we have to create group  so we can add user to that group   
`addgroup app` \
adduser -S sys user   -G supplementary group app name group  app name user  \
`adduser -S -G  app app `
show user of app group \
`groups app` \
we gonna combine  two command \
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
and rebuild image  1 `docker build -t react-app .`\
`docker run -it react-app sh` \
`/emilia $ whoami` \
`/emilia $ ls -l` \
\- rw- r--[primary group] r--[other group]  \
all this file owned by this root user  root user can write and permission
current user  its run this session is emilia user so this user is third group(other group) thats mean emilia user cant write this files\

## 12---defining entrypoint


