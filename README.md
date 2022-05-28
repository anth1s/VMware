# vmware prometheus project
uptime check monitoring service to be deployed on Kubernetes cluster.

This service can be accessed via any endpoint address with HTTP GET request

When the endpoint is accessed, this service checks the following two external URLs and return 4 metrics in Prometheus format.
* https://httpstat.us/503
* https://httpstat.us/200

There are two metrics for each URL:

* Whether the URL is up and running
* The response time in milliseconds


__Below is an example of response__
```
# HELP sample_external_url_up URL Up
# TYPE sample_external_url_up gauge
sample_external_url_up{url="https://httpstat.us/503"} 0.0
sample_external_url_up{url="https://httpstat.us/200"} 1.0
# HELP sample_external_url_response_ms Response Time
# TYPE sample_external_url_response_ms gauge
sample_external_url_response_ms{url="https://httpstat.us/503"} 298.56300354003906
sample_external_url_response_ms{url="https://httpstat.us/200"} 306.61916732788086
```

## Requirements
This service is developed in Python 3.7. 

The following packages are required:
* requests
* prometheus_client

The exact version of each package is specified in the `requirements.txt` file. To install the package, run:
```
pip install -r requirements.txt
```

## Manually Running the Service
This service can be started by the following command:
```
python app.py
```
By default, there will be a 1 second delay between two subsequent uptime check.

Optionally, you can specify the time interval between two checks by passing a number with the command:
```
python app.py 5
```

This command will perform the uptime check 5 seconds after the previous check.
Note that this is the time interval between two subsequent checks. This does not mean the checks are performed every 5 seconds since the uptime check itself may take about 1 second.
Metrics will always return the most recent uptime check results.

uptime time service will timeout if the URL fail to return any data in 5 seconds. Without timeout, the service may hang indefinitely if external URL does not response. 
Note that the timeout applies to data chunks instead of the whole connection. 
When timeout occurs, the URL is considered as down and the response time will be set to "inf".

## Docker Image
This service is available as a docker image: [anth1s/vmware2](https://hub.docker.com/r/anth1s/vmware2)

Docker pull command:
```
docker pull anth1s/vmware2
```
feel free to try it locally/standalone via cmd
```
docker run -dit -p 8080:80 anth1s/vmware2
```
This command maps the service to port 8080 on `localhost`. Once the docker image is running, the metrics will be available at http://localhost:8080

The docker image has a default entry point of `python` and default command/argument of `app.py`, which will start the service and perform the uptime check every one second. The repository also uses Github action to automatically build and push the docker image to Docker Hub. 

## Kubernetes Deployment Specifications!
The `deployment.yaml` helm file contains specifications for deploying this service to a kubernetes cluster using the docker image (`anth1s/vmware2`)

The following command will deploy the service on the kubernetes cluster authenticated with your account.
```
kubectl apply -f deployment.yaml
```
## Functional testing:
This repository uses travis-ci to perform automated testing whenever a commit is pushed to the master branch 

This service is built on 3 functions in app.py:
* `check_url()`, Checks if a URL is up and track the response time by sending HTTP GET request.
* `uptime_check()`, Performs uptime checks to two URLs.
* `parse_arguments()`, Parses the command line arguments.

The return values of check_url() and uptime_check() are mainly designed for testing purpose. The tests.py contains unit tests for these three functions. Tests can be executed with the following command:
```
python -m unittest discover -v -s . -p "test*.py"
```
## From the Prometheus Server
The `prometheus.yml` file is a config file for Prometheus server to monitor this service running locally on port 80.

Figure shows a simple query on the Prometheus server:
![Prometheus Server Query](artifacts/prometheus_server_query.png)

Figure visualizes the metrics in Grafana:
![Grafana Graphs](artifacts/Grafana.png)

