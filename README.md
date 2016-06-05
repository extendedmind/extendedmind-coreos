# Extended Mind CoreOS Installation Instructions on GCE

## Create a three node CoreOS cluster on GCE

First get a fresh token from:

```
https://discovery.etcd.io/new?size=3
```

where the random last 32 characters of the generated URL is YOUR_TOKEN you can use in the build:

```
mvn clean generate-resources -Detcd.token=YOUR_TOKEN -Dstorage.id.prefix="google-node" -Dstorage.id.postfix="-data" -Dbackup.bucket.path="YOUR_BACKUP_BUCKET" -Dfiles.bucket.path="YOUR_FILES_BUCKET" -Dmariadb.root.pwd="YOUR_PREFERRED_MARIADB_ROOT_PASSWORD"
```

For node X (where X is either 1, 2 or 3), create a GCE instance with (using pre-created persistent disks "nodeX-data" for storing Docker data):

```
gcloud compute instances create YOUR_NODE_NAME_X --image CURRENT_CORE_OS_IMAGE --machine-type YOUR_MACHINE_TYPE --disk 
name=nodeX-data,device-name=nodeX-data --metadata-from-file user-data=target/cloud-config/nodeX-cloud-config.yaml
```

## Optional: setup Icinga2 monitoring

If you have an Icinga2 server running, initialize an Icinga2 remote execution bridge:

```
gcloud ssh YOUR_NODE_NAME_X
docker create --name DATA-ICINGA2 extendedmind/icinga2-ceb:latest /bin/true
docker run -d -p 5665:5665 -t --name icinga2-ceb --volumes-from DATA-ICINGA2 extendedmind/icinga2-ceb:latest
docker exec -i -t icinga2-ceb bash
icinga2 node wizard
exit
docker kill icinga2-ceb
docker rm icinga2-ceb
docker run -d -p 5665:5665 -t --name icinga2-ceb --volumes-from DATA-ICINGA2 extendedmind/icinga2-ceb:latest
```
