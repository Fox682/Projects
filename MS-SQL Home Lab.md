# Project: MS-SQL Home Lab  
* MS-SQL Home Lab (Debian)
  * Setup a place to learn MS-SQL
  * Requires: Official MS-SQL Docker & Azure Data Studio  

Learning SQL can take time, having a Home Lab to run makes learning easier and provides more opportunity to take the time to learn.  
  
Update the system:  
```bash
# apt update && apt upgrade
```  
Install Docker CE:  
Download the Docker .deb - https://download.docker.com/linux/debian/dists/  
Browse to the pool then stable, eg.
```bash
# wget https://download.docker.com/linux/debian/dists/stretch/pool/stable/amd64/docker-ce-cli_18.09.3~3-0~debian-stretch_amd64.deb
# dpkg -i /path/to/package.deb
```  
Functional Testing:  
```bash
# docker run hello-world
```  
Note: To update the Docker .deb package, download and install Docker again  
  
When installation is successful you will get output:  
