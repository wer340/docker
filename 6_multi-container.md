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
make [docker-compose.yml](https://docs.docker.com/compose/compose-file/)   this name is default assume \

```yml
version:"3.8"
services:
    frontend

```