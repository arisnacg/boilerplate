Create MongoDB keyfile
```sh
openssl rand -base64 756 > mongodb-keyfile
chmod 400 mongodb-keyfile
```

Start the MongoDB containers
```sh
docker compose up -d
```

Login to the mongo1 (primary)
```sh
docker exec -it mongo1 mongosh -u root -p root
```

Initiate the replica set with all nodes
```javascript
rs.initiate({
  _id: "rs0",
  members: [
    { _id: 0, host: "mongo1:27017", priority: 2 },
    { _id: 1, host: "mongo2:27017", priority: 1 },
    { _id: 2, host: "mongo3:27017", priority: 1 }
  ]
})
```

Or initiate the primary node first and add the secondary nodes one by one
```javascript
rs.initiate()
rs.add("mongo2:27017")
rs.add("mongo3:27017")
```

Verify the replica set
```javascript
rs.status()
```

Create MongoDB replica set cluster admin
```javascript
use admin
db.createUser({
  user: "clusterAdmin",
  pwd: "clusterPass",
  roles: ["clusterAdmin", "readWriteAnyDatabase"]
})
```

Create MongoDB replica set cluster user for read only
```javascript
use admin
db.createUser({
  user: "clusterReadAdmin",
  pwd: "clusterReadPass",
  roles: [ "clusterMonitor", "readAnyDatabase" ]
})
```

List all users
```javascript
use admin
db.getUsers()
```

The connection string for the replica set
```sh
mongosh "mongodb://clusterAdmin:clusterPass@localhost:27017,localhost:27018,localhost:27019/?replicaSet=rs0"

mongosh "mongodb://clusterReadAdmin:clusterReadPass@localhost:27017,localhost:27018,localhost:27019/?replicaSet=rs0&readPreference=secondary"
```
