# NODE MONGODB AND REACT DOCKERIZED GOALS TRACKING APP

# MONGODB

Mongodb container is run on goals-net network and uses auth and named volume for data persistence even if container is stopped and removed

~ First Create a network
`docker network create goals-net`

~ Then run the mongo container with MONGO_INITDB_ROOT_USERNAME=saket and MONGO_INITDB_ROOT_PASSWORD=secret
uses named volume -v flag to persist the data even after container is removed and started
`docker run --name mongo-db --rm --network goals-net -v goals-data:/data/db -e MONGO_INITDB_ROOT_USERNAME=saket -e MONGO_INITDB_ROOT_PASSWORD=secret mongo `

# NODE BACKEND

Node backend app needs to expose port 80 for frontend app should also use goals-net network to communicate with mongodb container bind-mount created for /app folder for real time code changes

~ build the node app image
`docker build -t goals-node`

~ use .env file to pass mongo db user name and password with --env-file flag, mongo URI should also be updated
`mongodb://${process.env.MONGO_USERNAME}:${process.env.MONGO_PASSWORD}@mongo-db:27017/course-goals?authSource=admin`

~ run the node app container with bind mount for live reload
`docker run --name goals-backend -v \"%cd%\":/app -v logs:/app/logs -v /app/node_modules --env-file=./.env -p 80:80 --network goals-net --rm goals-node`

# REACT FRONTEND

~ build the image
`docker build -t goals-react .`

~ expose PORT 3000 and run the Container
`docker run --name goals-frontend --rm -p 3000:3000 goals-react`

~ create bind mount for src folder of react app for live load of code and use -e WATCHPACK_POLLING=true
`docker run -e WATCHPACK_POLLING=true -v C:\<pathtoapp>\src:/app/src --name goals-frontend --rm -p 3000:3000 goals-react`

~ for react-script > 5 use env -e WATCHPACK_POLLING=true else use -e CHOKIDAR_USEPOLLING=true
