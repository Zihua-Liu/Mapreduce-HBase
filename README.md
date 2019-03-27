# Mapreduce-HBase

This project is for loading data to HBase using mapreduce on a Hadoop cluster. The following steps show how to run this project on a GCP Dataproc Cluster.

### How to run

- First, install GCP SDK and set up your project. Also remember to enable Dataproc API.

- Set the compute region and zone for Hadoop Cluster.

  ```bash
  gcloud config set compute/region us-east1
  gcloud config set compute/zone us-east1-b
  ```

- Launch the cluster, the instance type and the number of workers can be changed.

  ```
  gcloud dataproc clusters create hadoop --master-machine-type n1-standard-2 --master-boot-disk-size 64 --num-workers 3 --worker-machine-type n1-standard-2 --worker-boot-disk-size 512 --image-version 1.3-deb9
  ```

- Log into the master node of the cluster

  ```bash
  gcloud compute ssh hadoop-m 
  ```

- Get the code and install maven on the master node of the cluster

  ```
  git clone https://github.com/Zihua-Liu/Mapreduce-HBase.git
  sudo apt-get install -y maven
  ```

- Update the configuration in `YetAnotherImportTsv`

  ```java
  Configuration conf = HBaseConfiguration.create();
  // TODO: update the Zookeeper address(es)
  String zkAddr = "";
  conf.set("hbase.zookeeper.quorum", zkAddr);
  conf.set("hbase.zookeeper.property.clientport", "2181");
  conf.set("hbase.cluster.distributed", "true");
  // Linux-based HDInsight clusters use /hbase-unsecure as the znode parent
  conf.set("zookeeper.znode.parent","/hbase-unsecure");
  ```

- Download the dataset to the cluster and put the dataset to HDFS

  ```bash
  # Note that the first line of `yelp_academic_dataset_business.tsv`, is not a record but the header row. 
  # You MUST get rid of the header line first so that you will not import the header line into HBase. 
  # We recommend that you use `tail` to skip the header row
  
  wget https://cloudcourse.blob.core.windows.net/dataset/yelp-academic-dataset/2018/tsv/yelp_academic_dataset_business.tsv
  
  tail -n +2 yelp_academic_dataset_business.tsv > yelp_academic_dataset_business_no_header.tsv
  
  # Create a new directory in HDFS   
  hadoop fs -mkdir /input
  
  # Put the dataset to HDFS
  hadoop fs -put yelp_academic_dataset_business_no_header.tsv /input
  
  # To check if the file is successfully stored to HDFS, use the following command (or you may specify your own file path):
  hadoop fs -ls /input/yelp_academic_dataset_business_no_header.tsv
  ```

- Run the project

- ```bash
  mvn clean package
  yarn jar target/import_tsv.jar edu.cmu.cc.utils.YetAnotherImportTsv
  ```

It will load the data to table "business" into your HBase