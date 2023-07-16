# Install in Linux(Ubuntu)

### Install using curl
* First install curl if already not installed
```sh
sudo apt install curl
```
* Now, get the docker file from the docker website. This will fetch docker from docker website to the get-docker.sh file in tmp folder
```sh
curl -o /tmp/get-docker.sh https://get.docker.com
```
* Now, run the get-docker.sh file using sh
```sh
sh /tmp/get-docker.sh
```
* Check docker is working by running below command
```sh
sudo docker run hello-world 
```
* Note that we used `sudo` to run docker. To avoid this we need add user to the docker group. we can add user to the group by using `usermod` command
```sh
sudo usermod -aG docker $USER
```
* You need to restart to effect this addition of user group

#### Additional Info on usermod command
-a -> specifies that we need to add a user 
G -> specifies that that it is group
-aG -> specifies that we are adding the user to the user-group. 
docker -> is the user-group that can run the docker
$USER -> current system user
