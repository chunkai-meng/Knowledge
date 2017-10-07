Install RStudio Server in Debian

apt-get update
apt-get install r-base r-base-dev
apt-get install libatlas3-base
apt-get install gdebi-core
apt-get install wget; \
wget https://download2.rstudio.org/rstudio-server-1.0.136-amd64.deb; \
gdebi rstudio-server-1.0.136-amd64.deb


install.packages("http://cran.r-project.org/src/contrib/Archive/RNetLogo/RNetLogo_0.9-6.tar.gz", repo=NULL, type="source")


docker run -d -p 8787:8787 rocker/rstudio
Use the docker-machine ip to determine the ip address for your local or remote machine command, then visit that address appended with the port :8787. You can now log in to the session with the default username and password.
* username: rstudio
* password: rstudio
docker run -d -p 8787:8787 -e USER=<username> -e PASSWORD=<password> rocker/rstudio


R -e 'install.packages(c("package1", "package2"))' # install to default location.

sudo R -e 'install.packages(c("package1", "package2"), lib="/usr/local/lib/R/site-library")' # install to location that requires root.

R -e 'install.packages(c(“s20x”))’

docker run name d -p 8787:8787 ck/rstudio
