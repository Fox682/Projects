# Project: MS-SQL Home Lab  
* MS-SQL Home Lab (Debian)
  * Setup a place to learn MS-SQL
  * Requires: Docker, Official MS-SQL Docker Image & Azure Data Studio  

Learning SQL can take time, having a Home Lab to run makes learning easier and provides more opportunity to take the time to learn.  
**Note:** All commands run as root unless otherwise specified. *(Use sudo if installed)*
  
Update the system:  
```bash
# apt update && apt upgrade
```  
Install Docker CE:  
Download the Docker .deb - https://download.docker.com/linux/debian/dists/  
Browse to the pool then stable, eg. https://download.docker.com/linux/debian/dists/stretch/pool/stable/amd64/
```bash
# wget https://download.docker.com/linux/debian/dists/stretch/pool/stable/amd64/docker-ce-cli_18.09.3~3-0~debian-stretch_amd64.deb
# dpkg -i /path/to/package.deb
```  
Download all the \*.deb files  

Install the CLI:  
```
dkpg -i docker-ce-cli_18.09.3~3-0~debian-stretch_amd64.deb
```  
Install containerd.io:  
```
dpkg -i containerd.io_1.2.4-1_amd64.deb  
```

Install the docker files:  
```
docker-ce_18.09.3~3-0~debian-stretch_amd64.deb  
```  

Run and Test:  
```
docker run hello-world  
```

**Note:**  To update the Docker .deb package, download and install Docker again  
  
When the installation is successful you will get output:  

```
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
1b930d010525: Pull complete 
Digest: sha256:2557e3c07ed1e38f26e389462d03ed943586f744621577a99efb77324b0fe535
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
 ```

Docker should now be installed and working properly. Next Step is to download the MS-SQL Docker container from Microsoft. We can use Docker for this.  

Install MS-SQL Docker image:  

```
docker pull mcr.microsoft.com/mssql/server:2017-latest
```  

Build the docker image:  
```
docker run --name sqldev -e 'ACCEPT_EULA=Y' -e 'MSSQL_SA_PASSWORD=Database007!' -e 'MSSQL_PID=Developer' -p 1433:1433 -d mcr.microsoft.com/mssql/server:2017-latest
```  
We should have a running docker instance of MS-SQL, we should be able to connect to it manually and/or with Azure Data Studio.

**Optional** - Add user to docker group to allow users to run docker containers instead of just root.  
```
usermod -aG docker $USER
```

If it does not start shortly after, OR the computer needs to be rebooted:  

```
docker start sqldev
```  

Verify the instance is running with:

```
#docker ps

a7cfaeeb0c53        mcr.microsoft.com/mssql/server:2017-latest   "/opt/mssql/bin/sqls…"   4 minutes ago       Up About a minute   0.0.0.0:1433->1433/tcp   sqldev

```
If this step fails check for errors with docker log followed by the first few (unique) characters of the container name. If running from a virtual machine, be sure it has 2GBs (2048MBs) allocated RAM.  

```
docker log a7cf
```

**Note:** The docker images have password requirements be sure to have a good password, for this test we're using "Database007!". You will need this to log into the SQL Server console, or connecting to it from Azure Data Studio.  

To verify the MS-SQL server is running and accessable log in to the docker image:  

```
docker exec -it sqldev "bash"
```  

Once inside the container (the prompt will change to a root prompt with the name of containter eg. root@a7cfaeeb0c53:/#)  we can run:  

```
/opt/mssql-tools/bin/sqlcmd -S localhost -U SA -P 'Database007!'
```
If we receive SQL prompt we're good to go!
```
1>
```  
**Note:** If you want Docker to start Automatically with the system, or disable this feature the following commands will do that respectively:  
```
systemctl enable docker
systemctl disable docker
```

**To Do: Install Azure Data Studio**  


