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

## Contributing

Please read [CONTRIBUTING.md](https://gist.github.com/PurpleBooth/b24679402957c63ec426) for details on our code of conduct, and the process for submitting pull requests to us.

## Versioning

TBD

## Authors

* **Mark Craddock** - *Initial work*
* **Neville de Mendonca** - *Initial work*

See also the list of [contributors](https://github.com/your/project/contributors) who participated in this project.

## License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details

## Acknowledgments

* Hat tip to anyone who's code was used
* Inspiration
* etc
