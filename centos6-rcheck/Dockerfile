# Produces a CentOS 6 container for performing an R CHECK as cran
FROM centos:6
MAINTAINER "Robert Casey, IRIS" <rob@iris.washington.edu>

# install latest R-devel available for CentOS 6
RUN yum update -y \
&& rpm -Uvh http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm \
&& yum install -y texinfo-tex libcur* libxml* openssl* texlive texinfo \
&& yum install -y tcl \
&& yum clean all \
&& yum install -y R-devel qpdf

# install additional CRAN packages needed for our test
RUN echo 'options(repos = c(CRAN = "https://cran.rstudio.com/"))' > /tmp/install.packages.R \ 
&& echo 'install.packages(c("signal","Rcpp","XML","tidyr","dplyr","pracma","RCurl","microbenchmark"))' >> /tmp/install.packages.R \
&& R CMD BATCH /tmp/install.packages.R

# taking cues from shabbychef/crancheck Dockerfile
# mount a shared filesystem from the host server as /srv from docker run
# e.g.  docker run -it --rm --volume $(pwd):/srv:rw image_name
# on OS X, Moby Linux VM allows /Users, /Volumes, /private, and /tmp automatically
# so pwd should be something like /Users/test/mytest, or /tmp/mytest -mount as-> /srv
#
# set this mount point to our current work directory
WORKDIR /srv

# create a script one-liner that will allow us to perform ordered test operations from ENTRYPOINT
RUN echo 'for f in $*;do R CMD INSTALL $f;R CMD check --as-cran --output=/tmp $f;done' > /tmp/Rcheck.bash

# Run the check script at invocation.
# we are looking for files in WORKDIR
# ENTRYPOINT means we will run this command and any args
# will override what is in CMD as additional args (filenames)
ENTRYPOINT ["bash","/tmp/Rcheck.bash"]
CMD ["/srv/*.tar.gz"]
