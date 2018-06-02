# springboot-mesos-chronos

## Goal

The goal of this project is to create a simple java application and use `Chronos` to run it in specific intervals.
In order to simulate finishing status of the job (successfully or error), there is an environment variable `EXIT_CODE` that can be set as:
```
0 = success (default)
!0 = failure.
```
Besides, there is an environment variable called `SLEEP`, `default 5000`, to change the amount of time the thread sleeps (_in milliseconds_).
The idea of the `SLEEP` is to simulate the application processing something that takes time, for example. 

## Start Environment

1. Open one terminal.

2. Go to `/dev` folder inside `/springboot-mesos-chronos` root folder.

3. Export the machine ip address to `HOST_IP_ADDR` environment variable.
> It can be obtained by executing ifconfig command on Mac/Linux terminal or ipconfig on Windows;
```
export HOST_IP_ADDR=...
```

4. Inside `/springboot-mesos-chronos/dev` folder run
```
docker-compose up -d
```

5. In order to check if all applications are `UP` and running type
```
docker-compose ps
```

You should see something like
```
Name        Command                          State        Ports
-------------------------------------------------------------------
chronos     /chronos/bin/start.sh --zk ...   Up      0.0.0.0:4400->4400/tcp
master      mesos-master --registry=in ...   Up      0.0.0.0:5050->5050/tcp
slave       mesos-slave                      Up      0.0.0.0:5151->5151/tcp
zookeeper   /docker-entrypoint.sh zkSe ...   Up      0.0.0.0:2181->2181/tcp...
```

## Build Docker Image

1. Open a new terminal

2. Go to the `/springboot-mesos-chronos` root folder

3. To build the docker image run
```
mvn clean package docker:build -DskipTests
```

## Run Docker Container

In order to check if the image was built correctly, run the container.
```
docker run --rm \
  -e EXIT_CODE=0 \
  -e SLEEP=1000 \
  docker.mycompany.com/springboot-mesos-chronos:1.0.0; echo $?
```

## Running as Chronos Job

1. Go to the `/springboot-mesos-chronos` root folder.

2. You can edit some properties present in `/src/main/resources/job.json`.
For example, you can change `"schedule"` to the specific date/time (UTC) you want the job to start and the interval and change the `"container.image"` to the newest one.

3. Use curl command bellow to add jobs to Chronos. For more Chronos endpoints visit https://mesos.github.io/chronos/docs/api.html.
```
curl -i -X POST \
  -H "Content-Type: application/json" \
  -d@./dev/job.json \
  http://localhost:4400/v1/scheduler/iso8601
```
4. You can check and edit the schedule of the jobs on `Chronos` website: http://localhost:4400

5. In order to check the history of complete tasks, stderr and stdout of those tasks, etc you can visit `Mesos` website: http://localhost:5050