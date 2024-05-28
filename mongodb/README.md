Create MongoDB keyfile
```
openssl rand -base64 756 > mongodb-keyfile
chmod 400 mongodb-keyfile
```

Start the MongoDB containers
```
docker compose up -d
```

Login to the mongo1 (primary)
```
docker exec -it mongo1 mongosh -u root -p root
```

Initiate the replica set with all nodes
```
rs.initiate({
  _id: "rs0",
  members: [
    { _id: 0, host: "mongo1:27017" },
    { _id: 1, host: "mongo2:27017" },
    { _id: 2, host: "mongo3:27017" }
  ]
})
```

Or initiate the primary node first and add the secondary nodes one by one
```
rs.initiate()
rs.add("mongo2:27017")
rs.add("mongo3:27017")
```

Verify the replica set
```
rs.status()
```

Create MongoDB replica set cluster admin
```
use admin
db.createUser({
  user: "clusterAdmin",
  pwd: "clusterPass",
  roles: ["clusterAdmin", "readWriteAnyDatabase"]
})
```

Create MongoDB replica set cluster user for read only
```
use admin
db.createUser({
  user: "clusterReadAdmin",
  pwd: "clusterReadPass",
  roles: [ "clusterMonitor", "readAnyDatabase" ]
})
```

List all users
```
use admin
db.getUsers()
```

The connection string for the replica set
```
mongodb://clusterAdmin:clusterPass@mongo1:27017,mongo2:27017,mongo3:27017/?replicaSet=rs0
mongodb://clusterReadAdmin:clusterReadPass@mongo1:27017,mongo2:27017,mongo3:27017/?replicaSet=rs0
```
