CTRL+SHIF+V\

## 2----starting container name-container
`docker images`\
see running container  `docker ps `\
`docker run react-app`\
when running container im not going to run any command
ctrl+c docker stop  you can see in `docker ps`\
let me show cool technic this command with -d option [d=detach] 
`docker run react-app`\
we can run this container in the detach mode  which mean is background so if write `docker ps` you see  is react-app run in background as well as can type command\
in the last column in `docker ps `table is name thats docker select random name if you in future want to run this container a lot of time consume for finding one\
so this reason given a name for container \
`docker run -d --name emilia-edited react-app` \

## 3--viewing the log 
so we have been running container in background there is problem we don't know  what is going inside  container what if something goes wrong what if server generate error thats where is **logs** command\
`docker ps`\
`docker logs 655 #containerId`\
so here we can see output of web server thats exactly we saw  when lets started in foreground  \
`docker logs --help`\
you can see real time of terminal for exit ctrl+c`docker logs -f 655 ` \
look at five line`docker logs -n 5 655`\
time of each message`docker logs -n 5 -t 655` \

 ## 4---publishing ports
 so currently in previous lesson  running react-app in container as know  i am opning the browser and type `http://localhost:3000` but see error ??!\
because port 3000 is publish in container none on the host so on the same machine multilpe container each container lessening port 3000 but host itself its not lessening its none of them ports  so look at \
`docker ps `\
see column port in table   -p =port \
these number of port don't must same 1️host:2️container\
❌`docker run -d -p 3000:3000 --name c1 react-app ` ❌\
✔`docker run -d -p 80:3000 --name c1 react-app ` ✔\

## 5---executing command in running container  
`docker exec`\
`docker run`\
whats exact different between those  \
docker run starting new container but docker exec is start again running container \
we see workdir `docker exec c1 ls` \
`docker exec -it c1 sh`\

## 6---stopping and starting container
`docker stop c1`\
`docker start c1`\
docker  start run stored container not new container\

## 7---removing container 
`docker container rm c1`  or short this\
`docker rm c1`\
if your container is running given error for this way can use  -f force option \
`docker rm -f c1`\
if to be a bunch of container in list `docker ps -a`
lets find c1 container here we can use piping \
 `docker ps -a | grep c1`\
 all the stop container remove`docker container prune `\

 ## 8---container file sys
 each container has it own  file sys thats invisible for other container  lets experiment\
 `docker ps`\
 ```docker 
 docker exec -it 655 sh`
 /app $ echo data > data.txt
 /app $ exit
```
 `docker ps` so here the other container not find data.txt\ 
 ```docker 
 docker exec -it 6eb sh`
 /app $ ls| grep data
/app $ exit
```
so delete container all file and file sys  delete\

## 9---persisting data using volumes
a volume is a storage outside of container it can be directly on the host  and somewhere on the cloud so we can using of volume \
see help `docker volume `\
calling anything doesn't matter`docker volume create app-data`\
`docker volume inspect app-data`\
createAt :  when this volume created \
Driver: thats is directory on the host\
Mountpoint: this isa where created directory on the host in mac is exist in virtual machine linux\
`docker run -d -p 4000:3000 -v app-data:/app/data react-app`\
if not create app-data docker automatically create and -v=volume\
`docker exec -it 716 sh `
```docker
/app $ cd data
/app/data $ echo data > data.txt
# sh : cant create data.txt :permission denied
/app/data $ cd ..
/app $ ls -l
#  the owner has permission     r-x third group
```

   app-data:/app/data \
/app/data  app dir is path in dockerfile when create image  and data folder is created by docker volume with level root user default not can access to other user write permission in the folder    but in dockerfile we run as  regular user docker  not permission automatically create data.txt directory so we already add  CMD mkdir data  into dockerfile as regular USER  because folder created by USER so write permission  also have by default  \  
```docker
# make echo node_module/ > .dockerignore
FROM node:14.16.0-alpine3.13  #reuse cache
RUN addgroup app && adduser -S -G app emilia #reuse cache
USER emila
WORKDIR /app
RUN mkdir data
COPY package*.json .  #reuse cache when not change
RUN npm install  # when above instruction not rebuild this command not run
COPY . .  #always rebuild
EXPOSE 3000
CMD ["npm" ,"start"]
```
as know test!  if create  data/data.txt in  specific container   then remove container whether data.txt is available or no ?   ✔yes available \
`docker build -t react-app .`\
```docker
/app $ cd data
/app/data $ echo data > data.txt
/app/data $ exit
```
and remove this container \
`docker rm -f 007`\
`docker run -d -p 4000:3000 -v app-data:/app/data react-app`\
and  docker -d run in the background\
`docker exec -it e1c sh` \
```docker
/app $ cd data
/app/data $ ls
# expect : data.txt
```
## 10----copying file between the host and container 
`docker ps`\
`docker exec -it e1c9043ea8ce sh`\
log file inside the app directory inside the container  
```docker
/app $ echo hello > log.txt
/app $ exit
```
`docker cp e1c9043ea8ce:/app/log.txt .`\
[cp=copy ] & . [period sign] current directory \
`ls`\
 #### as know   copy from host to container  
 `echo hello > secret.txt `
`docker cp secret.txt e1c9043ea8ce:/app`\
and go to `docker exec -it e1c9043ea8ce sh`
```docker
/app $ ls -l
/app $ exit
```
## 11--- sharing the source code with a container 
hot reloading 
we change to file and online  see change in `http://localhost:4000`\
if refresh cant see change 
for production create new image but for development for any change whether new image or copy? no this idea bad  so  use mapping \
`docker run -d -p 4000:3000 -v app-data:/app/data`\
using the same syntax  map directly on the host to directly inside container so instead of docker volume use current directory we dont want full path here  use pwd command `pwd:/app/data`  we gonna wrap this `$(pwd):/app` \
`docker run -d -p 4000:3000 -v $(pwd):/app -v app-data:/app/data react-app `\
but this react application doesnt store anything on the disk  like database  is just frontend application we dont need volume here \ 
`docker run -d -p 4000:3000 -v $(pwd):/app -v react-app `\
see live container real time`docker logs -f 696 `\
for  using volume option   host to container directly map