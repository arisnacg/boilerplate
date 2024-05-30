# MongoDB

## Setup MongoDB Replica Set

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

## Problems
### Warning: vm.max_map_count is too low

Check current setting on the host
```sh
cat /proc/sys/vm/max_map_count
```
Update the setting on the host ( `262144` is the recommended value )
```sh
sudo nano /etc/sysctl.conf
# Add or update: vm.max_map_count= 262144
sudo sysctl -p
```
Restart the docker service
```sh
sudo systemctl restart docker
```
Check the setting on the container
```sh
cat /proc/sys/vm/max_map_count
```
If the warning still persists, update the soft and hard limti of nofile
```yaml
services:
  mongo:
    ulimits:
      nofile:
        soft: 65536
        hard: 1048576 
    # ...
```
Reference: https://www.mongodb.com/community/forums/t/mongodb-docker-vm-max-map-count-is-too-low-even-if-set-to-524288/280470/1