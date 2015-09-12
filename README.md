# ![](bp_logo.png#top) BluePlanet2 - Go Micro-Service Solution [![go-bp2](https://img.shields.io/badge/go--bp2-example-blue.svg)]()
This is a sample solution project that contains a "fig" file for both
solution as well as a solution that contains a sample service and consumer of
that service, both written in Go.

### Prerequisites and Assumptions
It is assumed you have docker 1.8.1+. This may work with other versions, but I don't promise.

It is assumed that you have already built images for the two non-standard microservices used in this example: `bp2-service` and `bp2-consumer`. If not they can be found in the GIT repositories `https://github.com/davidkbainbridge/bp2-template` and `https://github.com/davidkbainbridge/bp2-consumer` respectively. To buid/deploy the images you shuold be able to execute the `make` targets `prepare-venv` and `image` in for each project. You will need Go 1.5+ to build these microservices.

It is also assumed that you have read access to the Cyan docker registry at `dockerreg.cyanoptics.com`. If you don't have this access you will need to talk to someone at Cyan (now Ciena) that has the power to grant you access. Employees should have read access by default.

### Preparing Solution
A solution in BluePlanet is a collection of apps or microservices. A solution is defined in a `fig` file that is a YAML formated documented based on the original `fig` docker composition work that has now been moved into the `docker-compose` project. The syntax is a Cyan modified verion, but it is pretty easy to understand.

This project contains two solutions. The first represents teh BluePlanet platform, yes the platform is just like any other BluePlanet solution, which is a good thing. The BluePlanet platform solution consists of three services that provide the basics for everything else:

- **etcd** - distributed configuration and cluster support
- **haproxy** - load balancing and high availability support
- **discod** - interface discover and inter-microservice connectivity

The plafrom solution definition is contained in the YAML file `platform.yml`.

The second solution included in this project is the solution that utilizes the two sample microservices:


- **bp2-service** - This service is based on the [Go-Kit Example](https://github.com/go-kit/kit/tree/master/examples), for the purposes of this solution it can capitalize a string
- **bp2-consumer** - This service repeated (about a 5 second interval) fetches some "lorem ipsum" from a [public web service](http://loripsum.net) and capitalizes it via the bp2-service.

The whole point of this example is to show how two services can be interconnected as well as demonstrate how Go based microservices can be used with BluePlanet (as well as vehicle for me to better learn about the platform)

#### From `fig` to `docker`
Prepareing a solution in BluePlanet parlance means transformnig the `fig` file into a docker contanier that can be leveraged by the BluePlanet solution manager. This is done via the `solmaker` command. The basic steps to accomplish this are:

1. Install pythons `virtualenv` (if not already installed)
```
sudo apt-get install -y python-virtualenv
```

2. Start the virtual environment, which will build out the virtual environment in a local directory named `env`.
```
virtualenv env
```

3. Install `solmaker` in this local environment. _note: you may need to install `python-dev` to include the `Python.h` headers if the initial install fails_
```
./env/bin/pip install git+git://github.cyanoptics.com/bp2/solution-maker.git
```

4. Build the docker files for the solutions. The `--tag=build` option when building the solution dockers makes life easier as by default a commit value is pulled from the git repository and will be hard to rememebr / type.
```
./env/bin/solmaker build --tag=build platform.yml
./env/bin/solmaker build --tag=build example.yml
```

### Start the Solution Manager
The BluePlanet solution manager is essentially a software control plane that is utilized to `deploy` or `undeploy` solutions as well as perform other "sollution-type" actions. Below I am startnig the `latest` version of the solution manager, which is likely not a great idea, but for me it worked.
```
docker run -d -v /var/run/docker.sock:/tmp/docker.sock \
  -v /bp2:/bp2 -p 8001:9999 \
  -e NOAUTH=True \
  -e DOCKER_SOCKET=/tmp/docker.sock \
  -e DOCKER_HOST=unix:///tmp/docker.sock \
  --name solutionmanager \
  dockerreg.cyanoptics.com/cyan/solutionmanager:latest
```

### Deploy the Solutions
At this point everything _should_ be ready to go and all that is required is to connect to the solution manager shell and issue the command to deploy the solution.
```
$ docker exec -it solutionmanager smcli
(Cmd) show_solutions
cyan.example:build
    solution_downloaded: True
dockerreg.cyanoptics.com.cyan.platform:build
    solution_downloaded: True
(Cmd) solution_deploy dockerreg.cyanoptics.com.cyan.platform:build
Deploying dockerreg.cyanoptics.com.cyan.platform:build
    Installing dockerreg.cyanoptics.com.cyan.platform:build
        SDI Already Downloaded
        Installing SDC
        App Images Already Download
        Creating App Containers
    Starting dockerreg.cyanoptics.com.cyan.platform:build
        Starting App Containers
(Cmd) solution_deploy cyan.example:build
Deploying cyan.example:build
    Installing cyan.example:build
        SDI Already Downloaded
        Installing SDC
        App Images Already Download
        Creating App Containers
    Starting cyan.example:build
        Starting App Containers
```
You can use `?<return>` at the `(Cmd)` prompt to get a list of available commnads from the solution manager. Play around, have fun.

### Verify the Solutoin
There are various ways to verify that things are working, from checking log files to using `docker exec` to issue commands to the various contianers. You can use `docker ps` to see the contianers that are runnnig. The containers will have a naming pattern that is `cyan.<solution>-<tag>_<service>_#`, e.g., `example-build_bp2-service_0`.

For this specific solution you can verify things are connected and working properly by viewing the logs from the `bp2-consumer` microservice and you should see a bunch of capitalized LOREM IPSUM log messages. if you use `-f` as opposed to `--tail=10` in the command below you will see the log messages as they are generated.
```
$ docker logs --tail=10 cyan.example-build_bp2-consumer_0
2015/09/11 23:55:41 Converted string to 'LOREM IPSUM DOLOR SIT AMET, CONSECTETUR ADIPISCING ELIT. EAEDEM RES MANEANT ALIO MODO. TIBI HOC INCREDIBILE, QUOD BEATISSIMUM. SEDULO, INQUAM, FACIAM. SINE EA IGITUR IUCUNDE NEGAT POSSE SE VIVERE? AGE SANE, INQUAM. DUO REGES: CONSTRUCTIO INTERRETE. COMPENSABATUR, INQUIT, CUM SUMMIS DOLORIBUS LAETITIA. HIC AMBIGUO LUDIMUR. '
2015/09/11 23:55:46 Converted string to 'LOREM IPSUM DOLOR SIT AMET, CONSECTETUR ADIPISCING ELIT. QUARE CONARE, QUAESO. GRAECE DONAN, LATINE VOLUPTATEM VOCANT. DUO REGES: CONSTRUCTIO INTERRETE. QUOD IAM A ME EXPECTARE NOLI. QUOD QUIDEM IAM FIT ETIAM IN ACADEMIA. '
2015/09/11 23:55:51 Converted string to 'LOREM IPSUM DOLOR SIT AMET, CONSECTETUR ADIPISCING ELIT. RECTE, INQUIT, INTELLEGIS. NAM QUID POSSUMUS FACERE MELIUS? AN NISI POPULARI FAMA? GLORIOSA OSTENTATIO IN CONSTITUENDO SUMMO BONO. NEMO IGITUR ESSE BEATUS POTEST. DUO REGES: CONSTRUCTIO INTERRETE. '
2015/09/11 23:55:56 Converted string to 'LOREM IPSUM DOLOR SIT AMET, CONSECTETUR ADIPISCING ELIT. EASDEMNE RES? REFERT TAMEN, QUO MODO. PROCLIVI CURRIT ORATIO. SI ENIM AD POPULUM ME VOCAS, EUM. '
2015/09/11 23:56:01 Converted string to 'LOREM IPSUM DOLOR SIT AMET, CONSECTETUR ADIPISCING ELIT. CUR ID NON ITA FIT? DAT ENIM INTERVALLA ET RELAXAT. SCRUPULUM, INQUAM, ABEUNTI; ET NEMO NIMIUM BEATUS EST; DUO REGES: CONSTRUCTIO INTERRETE. QUOD TOTUM CONTRA EST. '
2015/09/11 23:56:06 Converted string to 'LOREM IPSUM DOLOR SIT AMET, CONSECTETUR ADIPISCING ELIT. IDEM ISTE, INQUAM, DE VOLUPTATE QUID SENTIT? SED AD ILLUM REDEO. INQUIT, DASNE ADOLESCENTI VENIAM? SI LONGUS, LEVIS. '
2015/09/11 23:56:11 Converted string to 'LOREM IPSUM DOLOR SIT AMET, CONSECTETUR ADIPISCING ELIT. AT HOC IN EO M. QUARE ATTENDE, QUAESO. SI ENIM AD POPULUM ME VOCAS, EUM. INDE IGITUR, INQUIT, ORDIENDUM EST. QUO MODO? DUO REGES: CONSTRUCTIO INTERRETE. EASDEMNE RES? NIHIL SANE. '
2015/09/11 23:56:16 Converted string to 'LOREM IPSUM DOLOR SIT AMET, CONSECTETUR ADIPISCING ELIT. FACERES TU QUIDEM, TORQUATE, HAEC OMNIA; DUO REGES: CONSTRUCTIO INTERRETE. NON IGITUR BENE. '
2015/09/11 23:56:21 Converted string to 'LOREM IPSUM DOLOR SIT AMET, CONSECTETUR ADIPISCING ELIT. EAEDEM RES MANEANT ALIO MODO. MURENAM TE ACCUSANTE DEFENDEREM. QUID CENSES IN LATINO FORE? QUAM NEMO UMQUAM VOLUPTATEM APPELLAVIT, APPELLAT; DUO REGES: CONSTRUCTIO INTERRETE. SUMMAE MIHI VIDETUR INSCITIAE. '
2015/09/11 23:56:26 Converted string to 'LOREM IPSUM DOLOR SIT AMET, CONSECTETUR ADIPISCING ELIT. SINT MODO PARTES VITAE BEATAE. SUMMUM A VOBIS BONUM VOLUPTAS DICITUR. '
```
### Troubleshooting
While troubleshooting my solution, in particular why my NBIs were not being connectioned to my SBIs, several tools were helpful.

- **RESTful API to discod** - `https://dk2.cyanoptics.com/discod/api/v1/spec`, particularly the `/relations`, `/containers/<name>/nbis`, `/containers/<name>/sbis` resources. With these resources you can determine what the platform things is connected.

- **discod log** - When debugging microservice connectivity the following command will be your friend:
```
docker exec dockerreg.cyanoptics.com.cyan.platform-build_discod_0 \
  cat /bp2/log/discod.log | less
```
This command, modified to `exec` on your `discod` container, will allow you to view the log that contains information about what has been discovered and what has been connected. Searching for `soutthbound-update` + `<serivce>` in some form will likely get you want you need.

### Lessons Learned
- **BluePlanet requires bash on the container or does it** - The BluePlanet platform invokdes hooks on the microservice containers by executing commands. When doing this the platform assumes `bash` is on the target container. This may not be the case if you are using a "tiny" container such as BusyBox or Alpine. You can revert to Ubuntu if you want to bloat your image, but also you can fake it as demonstrated in the `bp2-comsumer` project, which is based on [work by tmost](https://github.cyanoptics.com/tmost/chatterbox/blob/master/bin/bash).

- **hook commands must be in the default shell path** - 'Nuf said. Symlink your hooks into `/usr/bin` in your `Dockerfile`; again see the `bp2-consumer` project.

- **hook overrides via the environement are interface specific** - Originally I thought I would be able to tell the platform where the `southbound-update` resides via a docker environment variable as opposed to a symlink. While this can be done it is interface specific. Thus if I have two southbound interface, `this` and `that` I can't use a single environment variable such as `SBI_southbound-update` which covers both. Instead I would have to specify `SBI_this_southbound-update` and `SBI_that_southbound-update`

- **haproxy assumes that the interface name is part of the URL path** - If I have a NBI that my microservice is providing named `something` the haproxy will proxy from itself using the NBI name and thus servers a URL similar to `http://haproxy:80/something` and proxy that to a URL on the target microservice also using the NBI name, `http://bp2-service:8901/something`. When you write a microservice that exposes a NBI your microservice must serve that NBI on a URL based on the NBI name.

### Thanks
I wanted to add a quick thank to those people on `#slack` that helped me thourgh this by answering questions as well as providing advise and suggestions. So special thanks (in no particular order) to `@jmiguel @rthille @chrisr @pwarner @tmost`.
