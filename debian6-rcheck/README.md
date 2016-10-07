# R Check Dockerfile for running on Debian 6

Build the container image, pointing it to the directory that contains this Dockerfile:

docker build -t debian6-rcheck:1 /Users/test/Docker_Builds/debian6_R

Run the container on your target R tar bundles (order is important for dependencies):

docker run -it --rm --volume $(pwd):/srv debian6-rcheck:1 seismicRoll*.tar.gz IRISSeismic*.tar.gz IRISMustangMetrics*.tar.gz

Note that in this run example, we are in the current directory where the tar bundles are.  In this example, we are looking at the following files:

seismicRoll*.tar.gz
IRISSeismic*.tar.gz
IRISMustangMetrics*.tar.gz

The ENTRYPOINT routine will call up a sequence of an R CMD install, followed by an R CMD check --as-cran on each file in turn.  The install part
is done so that subsequent files that may depend on that package will see it present and installed in the R environment.

