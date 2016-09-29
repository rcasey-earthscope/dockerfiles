FROM debian:6
##CREDIT rocker/r-base "Carl Boettiger and Dirk Eddelbuettel" rocker-maintainers@eddelbuettel.com
##CREDIT shabbychef/crancheck "Steven E. Pav" shabbychef@gmail.com
MAINTAINER "Robert Casey, IRIS" <rob@iris.washington.edu>

# debian 6 - squeeze now relegated to old archive repos
RUN echo 'Acquire::Check-Valid-Until "false";' >/etc/apt/apt.conf.d/90ignore-release-date \
&& echo "deb http://archive.debian.org/debian squeeze main" > /etc/apt/sources.list \
&& echo "deb http://archive.debian.org/debian squeeze-lts main" >> /etc/apt/sources.list \
&& echo "deb http://archive.debian.org/debian-archive/debian squeeze main" >> /etc/apt/sources.list \
&& echo "deb http://archive.debian.org/debian-archive/debian squeeze-proposed-updates main" >> /etc/apt/sources.list \
&& echo "deb http://security.debian.org squeeze/updates main" >> /etc/apt/sources.list \
&& echo "deb http://archive.debian.org/debian-archive/debian squeeze-lts main contrib non-free" >> /etc/apt/sources.list \
&& echo "deb http://cran.r-project.org/bin/linux/debian squeeze-cran3/" >> /etc/apt/sources.list

RUN apt-get update \ 
&& apt-get install -y --no-install-recommends \
apt-utils \
ed \
less \
locales \
vim-tiny \
wget \
ca-certificates \
&& rm -rf /var/lib/apt/lists/*

## Configure default locale, see https://github.com/rocker-org/rocker/issues/19
RUN echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen \
&& locale-gen en_US.utf8 \
&& /usr/sbin/update-locale LANG=en_US.UTF-8
ENV LC_ALL en_US.UTF-8
ENV LANG en_US.UTF-8

# gpg public key for CRAN
RUN apt-key adv --keyserver hkp://pgp.mit.edu:80 --recv-keys 381BA480

ENV R_BASE_VERSION 3.2

## Now install R and littler, and create a link for littler in /usr/local/bin
## install additional Debian packages that R check will need
## Also set a default CRAN repo, and make sure littler knows about it too
RUN apt-get update \
&& apt-get install -t unstable -y --no-install-recommends \
littler \
r-cran-littler \
r-base=${R_BASE_VERSION}* \
r-base-dev=${R_BASE_VERSION}* \
r-recommended=${R_BASE_VERSION}* \
&& apt-get install -y libcurl4-openssl-dev libxml2-dev texlive texinfo qpdf texlive-fonts-extra \
&& echo 'options(repos = c(CRAN = "https://cran.rstudio.com/"), download.file.method = "wget")' >> /etc/R/Rprofile.site \
&& echo 'source("/etc/R/Rprofile.site")' >> /etc/littler.r \
&& ln -s /usr/share/doc/littler/examples/install.r /usr/local/bin/install.r \
&& ln -s /usr/share/doc/littler/examples/install2.r /usr/local/bin/install2.r \
&& ln -s /usr/share/doc/littler/examples/installGithub.r /usr/local/bin/installGithub.r \
&& ln -s /usr/share/doc/littler/examples/testInstalled.r /usr/local/bin/testInstalled.r \
&& install.r docopt drat devtools\
&& rm -rf /tmp/downloaded_packages/ /tmp/*.rds \
&& rm -rf /var/lib/apt/lists/*

# install additional CRAN packages needed for our test
RUN install.r signal Rcpp XML tidyr dplyr pracma RCurl microbenchmark

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
