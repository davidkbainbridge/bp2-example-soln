# ![](bp_logo.png#top) BluePlanet2 - Go Micro-Service Solution [![go-bp2](https://img.shields.io/badge/go--bp2-example-blue.svg)]()
This is a sample solution project that contains a "fig" file for both
solution as well as a solution that contains a sample service and consumer of
that service, both written in Go.

### Preparing Solution
This
Preparing the solution is all about building docker containers

### Build the platform solution
./env/bin/solmaker build --tag=build platform.yml`

### Build the example solution
./env/bin/solmaker build --tag=build example.yml`

### Start the solution manager
`docker run -d -v /var/run/docker.sock:/tmp/docker.sock -v /bp2:/bp2 -p 8001:9999 -e NOAUTH=True -e DOCKER_SOCKET=/tmp/docker.sock -e DOCKER_HOST=unix:///tmp/docker.sock --name solutionmanager dockerreg.cyanoptics.com/cyan/solutionmanager:latest`

### Get to the solution managre shell
docker exec -it solutionmanager smcli
