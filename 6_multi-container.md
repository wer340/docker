## 1----running multiple application

## 2---install docker compose
`docker-compose --version`\
go to [docker doc] (docs.docker.com/compose/install/)\

## 3------cleaning up or workspace 
some technic for  clean  container and image by  quick\
for regular method  \
`docker container rm  name or container id`\
for remove image \
`docker image rm  name or container id`\
for prune \
`docker container prune`\
`docker image  prune`\
as know  can delete all by this technic\

this command list id container `docker container ls -q`\
take advantage this command \
`docker image rm $(docker image ls -q)`\
similarly for container
`docker container rm $(docker container ls -a-q)` -a this will bring stop container as well and combine switch -a -q  equal -aq \
then error thats running container  don't can remove  causes of  add -f  force option to command\
`docker container rm -f $(docker container ls -aq)` \
or  another way  go to preferences from icon docker then click on troubleshoot icon and click  clean/purge    after this work  wait half min till container restart and running\

## 4---- sample web Application
in project of course open \
`ls`\
see backend and docker-compose.yml and frontend \
`docker-compose up`\

## 5----json and yaml formats 
json is humaanaable laanguaage 
```json
{
"name":"the ultimate docker course ",
"price":149,
"is_published":true,
"tags":["software" , "devops"],
"author":{
    "first_name":"Mosh",
    "last_name":"hamedani"
        }
}
```
and yaml or yml  file \
```yml
name:the ultimate docker course
price:149
is_published:true
tags:
    -software
    -devops
author:
    first_name:Mosh
    last_name:hamedani
```
in yml we don't use curly braces {} this idea come from python thats indentation representation of hierarchy and in yml im not use  quote ""
and remove comma ,  as separate key , value pair  how do represent of list or array?  we use hyphen as array of list  in each line with same indentation  object also represent with indentation \
+ [] => - + indentation\
+ {} => indentation\
+ ,  => new line\
+ "" => remove\

```json
{
"author":{
    "first_name":"Mosh",
    "last_name":"hamedani"
},

"tags":["software" , "devops"]
}
```
to 
```yml
author:
    first_name:Mosh
    last_name:hamedani

tags:
    -software
    -devops
```
yaml easier to read and understand why the use yaml ?
 because parsing yaml file little bit slower than parsing json file because yaml  from scratch all value as string  contrast json thats string by quote is identified\

## 6--- creating compose file 
make [docker-compose.yml](https://docs.docker.com/compose/compose-file/) in root   this name is default assume \

```yml
version:"3.8" #wrap into quote  expect know docker-compose  evaluate as number
services: #in this file we define various building block this is our application 
    frontend: #arbitrary name  can write web 
    backend : #arbitrary name  can write api
    database: #arbitrary name  can write db 

```
the idea here is that we define various services and telling docker how to **build image** service and **how to run service**\
so we gonna have property and value of property  eventually will use running on container so in previous section we are manually run on container volume,port  mapping like `docker run -d -p 4000:3000 react-app`\
 all these value can be defined in our compose file so we dont manually start container , docker compose will take our starting container under the hood \
 so each server  can tell docker  how to build for that service \ notice : each service have your own dockerfile \
```yml
version:"3.8"
services: 
    web: 
        build: ./frontend #where is that exist dockerfile 
    api : 
        build: ./backend #where is that exist dockerfile 
    db: 
        image: mongo:4.0-xenial #which is ubuntu version 30  i find up docker hub you can see  mong also have images build top of windows but windows images is very large over to 2.0gb   thats prefer linux images

```
then write mapping \
and api project needs to environment where are databases?\
and need to volume  \ mongo db to crud data data into temporary file sys  of data   
 
```yml
version:"3.8"
services: 
    web: 
        build: ./frontend  
        ports :
            - 3000:3000
    api : 
        build: ./backend
        ports :
            - 3001:3000
        environment:
              DB_URL:mongo://db/vidly
    db: 
        image: mongo:4.0-xenial 
        ports :
            - 27017:27017
        volumes:
            - vidly :/data/db # mongo db default data sve
    
    volumes:
        vidly:

```

## 7--- building images 
in  desktop /vidly   root \
for help `docker-compose  `\
`docker-compose  build --help`\
`docker-compose  build `\
then look at list image \
`docker images`\
see ▶ vildy_web vildy_api  mongo   curren tdirectory+name sevice\
docker use the cache  when look thats layer of dockfile is exist in cache call this rather than full build it\
`docker-compose  build --no-cache`\

## 8----starting stopping  the application  
lets look at the avliable option \
`docker-compose up --help`\
`docker-compose up --build`\
`docker-compose up -d`\
`docker ps`\
how this take down \
`docker-compose down`\

## 9----docker networking \
 `docker-compose up -d`\
 `docker network ls`\
 our three host  and have tHREE container  (web,api,db)\
 `docker ps`\
 `docker exec -it 8c6 sh`\
 ```
 /app $ ping api 
 # ping : permission denied (are you root?)
 /app $ exit

 ```
 login with root user `docker exec -it -u root 8c6 sh`\
 highest privilege  # pound sign
  ```
 /app # ping api 
 # ping : (172.21.0.3):56 data bytes 
.....
```
each container has ip address as part of network so lets the 
```
 /app # ifconfig
et0 ....
lo  ....
/app # 
```
 back to compose file early define this api service we added and environment variable that contain of db connection stream  in this connection stream have db  which is name host thats db host thats is db container you saw that api container can talk that this container because both the container all container of application the same network \
 this host this only available inside the docker environment 
```yml
version:"3.8"
services: 
    api : 
        build: ./backend
        ports :
            - 3001:3000
        environment:
              DB_URL:mongo://db/vidly
    db: 
        image: mongo:4.0-xenial 
        ports :
            - 27017:27017
        volumes:
            - vidly :/data/db # mongo db default data sve
```

## 10---  viewing logs
 `docker-compose logs `\
 view the logs all of container \
 `docker-compose logs  --help ` \
 `docker ps`\
 show log of  containers `docker  logs 8c6 -f`\


## 11--- publishing and changes 
we Don's want build after we gonna change my application 
```yml
version:"3.8"
services: 
    web: 
        build: ./frontend  
        ports :
            - 3000:3000
    api : 
        build: ./backend
        ports :
            - 3001:3000
        environment:
            DB_URL:mongo://db/vidly
        volumes:
            - ./backend:/app
    db: 
        image: mongo:4.0-xenial 
        ports :
            - 27017:27017
        volumes:
            - vidly :/data/db  
    
    volumes:
        vidly:
```
`docker-compose up -d `\
if see error like nodemon  go to folder check package dependency go to folder  and npm install till package install \

## 12---- migrating database
most the time when release application our database to be particular shape of some data this is call database migration in this backend project using data base migration tools project in this project dependency  `migrate-mongo` other development stack like jango .net have same tools\
**summary how to work migrate db tools** \
 we can create db migrate script in this project script store in migrations folder so we when look script  see basic js file function for insert and remove data from db /
 so using tools `migrate-mongo`   we can go to terminal  write `migrate-mongo up`   this tool look at  all migration folder  and execute all migration script  \
 what if use execute  command multiple time? our script are  not executing twice \
 we can check compass mongo  change log  then see not execute multiple time \
 in `package.json`  this command (`migrate-mongo up `) mapping to 'db:up' in scripts section of file, means by you can say  `npm run db:up`\
how  can we execute database  migration as part of starting our application \
in dockerfile have \
```docker
CMD ["npm","start"]
```
it will start with web server   know our compose file overwrite this command here and do something else 
```yml
version:"3.8"
services: 
    web: 
        build: ./frontend  
        ports :
            - 3000:3000
    api : 
        build: ./backend
        ports :
            - 3001:3000
        environment:
            DB_URL:mongo://db/vidly
        volumes:
            - ./backend:/app
        command: ./wait-for db:27017 && migrate-mongo up && npm start    
    db: 
        image: mongo:4.0-xenial 
        ports :
            - 27017:27017
        volumes:
            - vidly :/data/db  
    
    volumes:
        vidly:
```
in command  its possible  db server not ready when this command  running this here need  wait for db  till completely run so  usae shell script  that write in `wait-for` file and specify wait for running after command  till not heard traffic port 27017 not running later command \
because statement of command to be  long use entrypoint  so create `docker-entrypoint.sh`  and this statement paste to this file  \
```docker
echo "waiting for MongoDB to start ..."
./wait-for db:27017 
echo "Migrating the database ....."
migrate-mongo up
echo "starting the server ......"
npm start    
```
then in compose file write address entrypoint
```yml
version:"3.8"
services: 
    web: 
        build: ./frontend  
        ports :
            - 3000:3000
        volumes:
            -./frontend:/app
    api : 
        build: ./backend
        ports :
            - 3001:3000
        environment:
            DB_URL:mongo://db/vidly
        volumes:
            - ./backend:/app
        command: ./docker-entrypoint.sh
    db: 
        image: mongo:4.0-xenial 
        ports :
            - 27017:27017
        volumes:
            - vidly :/data/db  
    
    volumes:
        vidly:
```
first bring down the application 
`docker-compose down`\
but volume not deleted lets see volume \
`docker volume ls`\
see first `vidly_vidly` represent of application second vidly represent of volume   so delete till mongo-db database delete\
`docker volume rm vidly_vidly `
so mongo db database is gone \
`docker-compose up`\

## 13---- running test
so `cd  ./frontend ` and  run `npm test `\

run test in container 
```yml
version:"3.8"
services: 
    web: 
        build: ./frontend  
        ports :
            - 3000:3000
    web-tests: 
        image: ./frontend  
        volume :
            - ./frontend:/app
        command:npm test
    api : 
        build: ./backend
        ports :
            - 3001:3000
        environment:
            DB_URL:mongo://db/vidly
        volumes:
            - ./backend:/app
        command: ./docker-entrypoint.sh
    db: 
        image: mongo:4.0-xenial 
        ports :
            - 27017:27017
        volumes:
            - vidly :/data/db  
    
    volumes:
        vidly:
```
we are not going to build image    we gonna use image 