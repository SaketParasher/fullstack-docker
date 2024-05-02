# DOCKERIZE USING LOCAL HOST PORT EXPOSE

A) DOCKERIZE MONGO DB
dockerize mongo db by running official mongo image but this time expose port 27017 which is the default port
since mongo container will be exposing port on local host
<docker run -t mongo-db --rm -p 27017:27017 mongo>

B) DOCKERIZE NODE APP
dockerize node app by exposing port 80 on localhost and since node app container needs to talk to mongodb locally
on port 27017 use host.docker.internal:27017 instead of localhost. change the app code to replace localhost to
host.docker.internal and build the image and run the container with port 80 exposed.
<docker build -t goals-node . >
<docker run --name goals-backend --rm -p 80:80 goals-node>

C) DOCKERIZE FRONTEND REACT APP
dockerize frontend react app and start the container in -it interactive mode and Expose port 3000
<docker build -t goals-react .>
<docker run --name goals-frontend -p 3000:3000 -it --rm goals-react>

#-------------------------------------------------------------------------------------------------------

# DOCKERIZING USING THE NETWORK

<docker network create goals-net> to create the network first

A) dockerize the mongodb container using the network and this time no need to expose the port since our node app
container and mongodb conatiner will be running in the same network.
<docker run --name mongo-db --network goals-net mongo>

B) Now for dockerizing node app we need to change the mongodb URI in the app code to use the mongodb container name
insted of host.docker.internal since now mongodb container will not be exposing port on localhost. Then rebuild the image and run the nodejs app container.
But this also needs to expose port 80 on which the node application is running because react app will be running in
the browser and it will not be able to connect with node server if we do not expose port 80 in localhost.
<docker build -t goals-node .>
<docker run --name goals-backend --rm -p 80:80 goals-node>

---

# DOCKERIZING WITH THE VOLUMES TO PERSIST THE DATA

A) Mongodb dockerization with named volumes to persist the data. MongoDB uses /data/db folder to persist the data.
we should start the mongodb container with named volume to persist the data even after removal of container.
<docker run --name mongo-db --rm --network goals-net -v goals-data:/data/db mongo>

<docker run --name mongo-db --rm --network goals-net -v goals-data:/data/db -e MONGO_INITDB_ROOT_USERNAME=saket -e MONGO_INITDB_ROOT_PASSWORD=secret mongo > :: this is for authentication if you want to restrict the data access
of your db then run the container with username and password passed as env
After this if you are using the same volume which you used earlier wihout AUTH then either use different volume
or remove the existing volume.

B) Then in node app we need to update the mongodb connection URI to use username and password
<mongodb://saket:secret@mongo-db:27017/course-goals?authSource=admin>
then build the image and start the the container again

# BIND MOUNT FOR BACKEND APP

create bind mount volume for backend app but also add named volume for /logs folder so that it should be persisted even if container is removed and should be added after bound mount volume so that any changes in host machine /logs folder should not overide /logs folder.
Also add anonymous volume for /app/node_modules folder so that it should not be overriden by bind-mount and should not be taken from host machine code folder and should always be of inside container.

For bind mound complete path of the root app folder should be provided in our case in windows machine it is
C:\Docker\full-stack\backend

<docker run --name goals-backend --rm -v C:\Docker\full-stack\backend:/app -v logs:/app/logs -v /app/node_modules -p 80:80 --networks goals-net goals-node>

# Adding NODEMON so that server restarts itself whenever anychanges are made in node app code and it reflects in container

Install nodemon as dev dependency
add start script in package.json nodemon -L app.js
then change the CMD in dockerfile CMD ["npm","start]

---

# USING .env file to pass username and password

--> change the mongodb connection URI to use process.env instead of hardcding username and password
`mongodb://${process.env.MONGO_USERNAME}:${process.env.MONGO_PASSWORD}@mongo-db:27017/course-goals?authSource=admin`

--> env can be mentioned from dockerfile like ENV MONGO_USERNAME=saket MONGO_PASSWORD=secret
--> env can also be passed from command using -e flag like -e MONGO_USERNAME=saket -e MONGO_PASSWORD=secret
--> or use .env file add all env variables in the file and then use --env-file flag to pass the file path
--env-file=./.env
