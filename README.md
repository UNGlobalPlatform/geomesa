# Geomesa Demonstration

Geo Mesa / Accumalo / GeoServer / AWS EMR / GDELT

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
Connect to Geoserver:

http://your.dns.address:9090/geoserver/

Login: admin (default login)

Password: geoserver (default password)

Make sure the AWS security group is allocated to the EMR Master. This will allow external access from an external IP Addresses.
Ensure the elastic IP address is associated with the EMR Master to allow DNS access to be configured.

Find out the Zookeeper IP, you will need this below:

```
sudo docker exec accumulo-master cat /opt/accumulo/conf/accumulo-site.xml | grep -A2 instance.zoo | grep value | sed 's/.<value>(.)<\/value>/\1/'
```
Create new store in Geoserver:

Store name: ons

DataSourceName: gdelt

instanceId: gis

zookeepers: $zookeeperIPADDRESS

user: root

password: secret

tableName: geomesa.gdelt

## Authors

* **Mark Craddock** - *Initial work*
* **Neville de Mendonca** - *Initial work*

See also the list of [contributors](https://github.com/your/project/contributors) who participated in this project.

## License

TBD

## Acknowledgments

* Hat tip to anyone who's code was used
* Inspiration
* etc
