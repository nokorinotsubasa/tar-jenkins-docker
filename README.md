# Introduction
Here we will see how to persist Jenkins data with .tar on a containerized jenkins.

![](https://github.com/nokorinotsubasa/tar-jenkins-docker/blob/cd00d664ae4d7a6d2af87b3c41689126076d0262/images/archtecture.png)

## Notes about terraform

- Don't forget to change `terraform.tfvars` to set vm admin username, storage account key etc;

- The Virtual machines password will NOT be on the output, instead they can be securely found in the `terraform.tfstate` file.

## Steps

>`Note that in this example we will backup to an Azure Storage Account`

- On the Vm running the Jenkins Docker container, run the below command to tar and zip the volume which Jenkins uses to store its data, in our case `/var/jenkins_home`:

`docker run --cidfile=id.tmp --volumes-from <container_id> ubuntu tar -cO /var/jenkins_home | gzip -c > volume.tgz`

![](https://github.com/nokorinotsubasa/tar-jenkins-docker/blob/131136d52a2b0dfd5ffa2cdeaa8dbcb00dd3772a/images/tarcommand.png)

>`this can take some time to complete`

>`you need to have docker installed`

>`this will also create a log file`

>`it runs on ubuntu docker image`

>`volume.tgz is the name that of the final .tar file`

- Now you can upload the file wherever you want, in our case, we will upload to an azure storage account. To do this, with azure cli installed, run:

`az storage blob upload --account-name storage-account-name --container-name container-name --name volume.tgz --file volume.tgz --auth-mode key`

>`you can also use --auth-mode login if you will`

- Now, on another machine, pull the Jenkins docker image:

`docker pull jenkins/jenkins`

>`this will pull the latest jenkins docker image`

- Run the docker image:

`docker run --name jenkins -p 8080:8080 -p 50000:50000 -d -v jenkins_home:/var/jenkins_home jenkins/jenkins`

>`this will run a container named jenkins, based on jenkins/jenkins image, on detached mode, with ports 8080 and 50000 open. Also, we are defining the volume, this volume will be where the jenkins data will be stored.`

- Access the container on root:

`docker exec -u 0 -it <container_id> bash`

- With azure cli installed, download the tar file with:

`az storage blob download --account-name storage-account-name --container-name container-name --name volume.tgz --file volume.tgz --auth-mode key`

>`you can also use --auth-mode login if you will`

- Now, extract it:

`tar -xzvf volume.tgz`

![](https://github.com/nokorinotsubasa/tar-jenkins-docker/blob/cd00d664ae4d7a6d2af87b3c41689126076d0262/images/extractfile.png)

>`this will extract the tar file`

- We already specified the jenkins_home PATH on the .tar command, so it will replace all jenkins data with the archived jenkins data on the .tar file.

- Restart the container:

`docker restart <container_id>`

## Final Result

Now when you access Jenkins, it will have all the data at backup run time and prompt you to log in:

![](https://github.com/nokorinotsubasa/tar-jenkins-docker/blob/cd00d664ae4d7a6d2af87b3c41689126076d0262/images/jenkinsloginpage.png)
