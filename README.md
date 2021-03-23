# 1. Create a development machine in Compute Engine

1. In the Cloud Console, go to Compute Engine > VM Instances > Create.
2. Configure the following fields, leave the others at their default value.

	-Name: devhost
	
	-Series: N1
	
	-Machine Type: 2 vCPUs (n1-standard-2 instance)
	
	-Identity and API Access: Allow full access to all Cloud APIs.
<br>
<br>
# 2. Install software

1. Set up Scala and sbt

```
sudo apt-get install -y dirmngr unzip
```
```
sudo apt-get update
```
```
sudo apt-get install -y apt-transport-https
```
```
echo "deb https://dl.bintray.com/sbt/debian /" | \
sudo tee -a /etc/apt/sources.list.d/sbt.list
```
```
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 642AC823
```
```
sudo apt-get update
```
```
sudo apt-get install -y bc scala sbt
```

2. Set up the Feature Detector Files

```
sudo apt-get update
```
```
gsutil cp gs://spls/gsp124/cloud-dataproc.zip .
unzip cloud-dataproc.zip
```
```
cd cloud-dataproc/codelabs/opencv-haarcascade
```

3. Launch build

```
sbt assembly
```
<br>
<br>
# 3 Create a Cloud Storage bucket and collect images

1. Fetch project ID
```
GCP_PROJECT=$(gcloud config get-value core/project)
```
Name your bucket and set a shell variable to your bucket name. The shell variable will be used in commands to refer to your bucket.
```
MYBUCKET="${USER//google}-image-${RANDOM}"
```
```
echo MYBUCKET=${MYBUCKET}
```
2. Use the gsutil program, which comes with gcloud in the Cloud SDK, to create the bucket to hold your sample images:
```
gsutil mb gs://${MYBUCKET}
```
3. Download images into bucket:
```
curl https://www.publicdomainpictures.net/pictures/20000/velka/family-of-three-871290963799xUk.jpg | gsutil cp - gs://${MYBUCKET}/imgs/family-of-three.jpg
```
```
curl https://www.publicdomainpictures.net/pictures/10000/velka/african-woman-331287912508yqXc.jpg | gsutil cp - gs://${MYBUCKET}/imgs/african-woman.jpg
```
```
curl https://www.publicdomainpictures.net/pictures/10000/velka/296-1246658839vCW7.jpg | gsutil cp - gs://${MYBUCKET}/imgs/classroom.jpg
```
<br>
<br>
# 4 Create Dataproc Cluster
1. Run the following commands in the SSH window:
```
MYCLUSTER="${USER/_/-}-qwiklab"
```
```
echo MYCLUSTER=${MYCLUSTER}
```
2. Set a global Compute Engine region to use and create a new cluster:
```
gcloud config set dataproc/region us-central1
```
```
gcloud dataproc clusters create ${MYCLUSTER} --bucket=${MYBUCKET} --worker-machine-type=n1-standard-2 --master-machine-type=n1-standard-2 --initialization-actions=gs://spls/gsp010/install-libgtk.sh --image-version=2.0  
```
<br>
<br>
# 5 Submit job to Cloud Dataproc
1. Run command in SSH window:
```
curl https://raw.githubusercontent.com/opencv/opencv/master/data/haarcascades/haarcascade_frontalface_default.xml | gsutil cp - gs://${MYBUCKET}/haarcascade_frontalface_default.xml
```
2. Submit job to Dataproc:
```
cd ~/cloud-dataproc/codelabs/opencv-haarcascade
```
```
gcloud dataproc jobs submit spark \
--cluster ${MYCLUSTER} \
--jar target/scala-2.12/feature_detector-assembly-1.0.jar -- \
gs://${MYBUCKET}/haarcascade_frontalface_default.xml \
gs://${MYBUCKET}/imgs/ \
gs://${MYBUCKET}/out/
```
3. When the job is complete, go to Navigation menu > Storage and find the bucket you created (it will have your username followed by student-image followed by a random number) and click on it. Then click on an image in the Out directory. Click on Download icon, the image will download to your computer.
