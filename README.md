# Geomesa Demonstration

GeoMesa / Accumulo / Spark / GeoServer / AWS EMR / GDELT

## Getting Started

These instructions will get you a copy of the project up and running on AWS for development and testing purposes.

![Geomesa/Geoserver Demonstration](https://github.com/UNGlobalPlatform/geomesa/blob/master/docs/geomesa-example.png?raw=true)

![Geomesa/Geoserver Demonstration](https://github.com/UNGlobalPlatform/geomesa/blob/master/docs/uk-buildings-example.png?raw=true)

### Prerequisites

Access to an Amazon AWS admin account

### Installing

Create an Ubuntu image for administration, as the Amazon EMR instances will be within your VPC and not accessable from the internet.

SSH into your admin server and run the following to create the AWS EMR cluster.
```
aws emr create-cluster \
--name "GeoDocker GeoMesa Demonstration" \
--release-label emr-5.2.0 \
--output text \
--use-default-roles \
--ec2-attributes KeyName=YOURKEY \
--applications Name=Hadoop Name=Zookeeper Name=Spark \
--instance-groups \
Name=Master,InstanceCount=1,InstanceGroupType=MASTER,InstanceType=m3.xlarge \
Name=Workers,InstanceCount=3,InstanceGroupType=CORE,InstanceType=m3.xlarge \
--bootstrap-actions \
Name=BootstrapGeoMesa,Path=s3://geomesa-docker/bootstrap-geodocker-accumulo.sh,Args=\[-t=geomesa-1.3.2-accumulo-1.8.1,-n=gis,-p=secret,-e=TSERVER_XMX=10G,-e=TSERVER_CACHE_DATA_SIZE=6G,-e=TSERVER_CACHE_INDEX_SIZE=2G]
```
You will need to copy your key.pem onto your admin server to allow access to the EMR via SSH.
Find the IP address of your EMR Master and login using the following command on your admin server.

```
ssh -i YOURKEY.pem ec2-user@EMR Master IP Address
```
Check that all the docker containers are up and running:

```
sudo docker ps

CONTAINER ID IMAGE COMMAND CREATED STATUS PORTS NAMES
7ff58c9aa7b7 quay.io/geomesa/geomesa-jupyter:geomesa-1.3.2-accumulo-1.8.1 "tini -- start-notebo" 2 minutes ago Up 2 minutes jupyter
9e28369a3006 quay.io/geomesa/geoserver:geomesa-1.3.2-accumulo-1.8.1 "/opt/tomcat/bin/cata" 4 minutes ago Up 4 minutes geoserver
a3d23ca5e774 quay.io/geomesa/accumulo-geomesa:geomesa-1.3.2-accumulo-1.8.1 "/sbin/entrypoint.sh " 5 minutes ago Up 5 minutes accumulo-gc
159f8bf3221a quay.io/geomesa/accumulo-geomesa:geomesa-1.3.2-accumulo-1.8.1 "/sbin/entrypoint.sh " 5 minutes ago Up 5 minutes accumulo-tracer
44add09ec05f quay.io/geomesa/accumulo-geomesa:geomesa-1.3.2-accumulo-1.8.1 "/sbin/entrypoint.sh " 5 minutes ago Up 5 minutes accumulo-monitor
3711ed204d1b quay.io/geomesa/accumulo-geomesa:geomesa-1.3.2-accumulo-1.8.1 "/sbin/entrypoint.sh " 5 minutes ago Up 5 minutes accumulo-master
## Contributing

Please read [CONTRIBUTING.md](https://gist.github.com/PurpleBooth/b24679402957c63ec426) for details on our code of conduct, and the process for submitting pull requests to us.
```
#### Inject GDELT Data

Lets prepare to injest our data from GDELT hosted on AWS S3. This creates a list of CSV files that will be injested.
```
FILES=$(seq 80 -1 40 | xargs -n 1 -I{} sh -c "date -d'{} days ago' +%Y%m%d" | xargs -n 1 -I{} echo s3a://gdelt-open-data/events/{}.export.csv | tr '\n' ' ')
```
Now injest the data
```
sudo docker exec accumulo-master geomesa ingest -c geomesa.gdelt -C gdelt -f gdelt -s gdelt -u root -p secret $FILES
```
Test data ingest by exporting first 100 rows
```
sudo docker exec accumulo-master geomesa export -c geomesa.gdelt -f gdelt -u root -p secret -m 100
```
#### Configure Geoserver

Connect to Geoserver:

http://hostname:9090/geoserver/

Login: admin (default login)

Password: geoserver (default password)

Make sure the AWS security group is allocated to the EMR Master. This will allow external access from an external IP Addresses.
Ensure the elastic IP address is associated with the EMR Master to allow DNS access to be configured.

Find out the Zookeeper IP, you will need this below:

```
sudo docker exec accumulo-master cat /opt/accumulo/conf/accumulo-site.xml | grep -A2 instance.zoo | grep value | sed 's/.<value>(.)<\/value>/\1/'
```
Create new store in Geoserver:

```
Store name: ons
DataSourceName: gdelt
instanceId: gis
zookeepers: $zookeeperIPADDRESS
user: root
password: secret
tableName: geomesa.gdelt
```

#### Display data in Geoserver

First publish the store

Save the store and publish the gdelt layer.

Set the “Native Bounding Box” and the “Lat Lon Bounding Box” to -180,-90,180,90. Save the layer.

Then go to the following URL, it will take a long time to build the first map (several minutes).

http://hostname:9090/geoserver/wms?service=WMS&version=1.1.0&request=GetMap&layers=ons:gdelt&styles=&bbox=-180,-90,180.0,90&width=1350&height=600&srs=EPSG:4326&format=application/openlayers

#### Test Jupyter Service

The Jupyter service can also be tested by going to this URL:

http://hostname:8888/notebooks/GDELT%2BAnalysis.ipynb

#### Configure Tomcat

Edit the
```
/opt/tomcat/conf/server.xml
```
to move it over to port 80.

Change this section to port 80:

```Define a non-SSL/TLS HTTP/1.1 Connector on port 8080
-->
<Connector port="9090" protocol="HTTP/1.1"
connectionTimeout="20000"
```
## Adding UK Buildings Shapefiles

Login to the admin server.
Connect to the EMR cluster.
Find out the docker id of the geoserver 
```
sudo docker ps
```
Connect to the docker container
```
sudo docker exec -i -t 82ce493b07e5 /bin/bash
```
Change to the data directory
```
cd /opt/tomcat/webapps/geoserver/data/data/ 
mkdir ukshapefiles
cd ukshapefiles
```
Download all the shapefiles. Repeat the command below for each file.
```
wget https://www.dropbox.com/sh/kioja4ofr2azihn/AAD-K8Ze794fJv1h2tLEaErpa/midland_england_buildings_clipped.dbf?dl=0
```
![Geomesa/Geoserver Demonstration](https://github.com/UNGlobalPlatform/geomesa/blob/master/docs/geomesa-example.png?raw=true)

## Authors

* **Mark Craddock** - *Initial work*
* **Neville de Mendonca** - *Initial work*

See also the list of [contributors](https://github.com/your/project/contributors) who participated in this project.

## License

TBD

## Acknowledgments

* Alasdair Rae, for the UK building shapefiles, http://www.statsmapsnpix.com/2017/09/buildings-of-great-britain.html
* http://www.geomesa.org/documentation/tutorials/geodocker-geomesa-spark-on-aws.html



