### Node Scenarios
This scenario disrupts the node(s) matching the label on a Kubernetes/OpenShift cluster. Actions/disruptions supported are listed [here](https://github.com/redhat-chaos/krkn/blob/master/docs/node_scenarios.md)

#### Run
If enabling [Cerberus](https://github.com/redhat-chaos/krkn#kraken-scenario-passfail-criteria-and-report) to monitor the cluster and pass/fail the scenario post chaos, refer [docs](https://github.com/redhat-chaos/krkn-hub/tree/main/docs/cerberus.md). Make sure to start it before injecting the chaos and set `CERBERUS_ENABLED` environment variable for the chaos injection container to autoconnect.

```
$ podman run --name=<container_name> --net=host --env-host=true -v <path-to-kube-config>:/root/.kube/config:Z -d quay.io/redhat-chaos/krkn-hub:node-scenarios
$ podman logs -f <container_name or container_id> # Streams Kraken logs
$ podman inspect <container-name or container-id> --format "{{.State.ExitCode}}" # Outputs exit code which can considered as pass/fail for the scenario
```

```
$ docker run $(./get_docker_params.sh) --name=<container_name> --net=host -v <path-to-kube-config>:/root/.kube/config:Z -d quay.io/redhat-chaos/krkn-hub:node-scenarios
OR 
$ docker run -e <VARIABLE>=<value> --net=host -v <path-to-kube-config>:/root/.kube/config:Z -d quay.io/redhat-chaos/krkn-hub:node-scenarios

$ docker logs -f <container_name or container_id> # Streams Kraken logs
$ docker inspect <container-name or container-id> --format "{{.State.ExitCode}}" # Outputs exit code which can considered as pass/fail for the scenario
```

#### Supported parameters

The following environment variables can be set on the host running the container to tweak the scenario/faults being injected:

ex.) 
`export <parameter_name>=<value>`

See list of variables that apply to all scenarios [here](all_scenarios_env.md) that can be used/set in addition to these scenario specific variables

Parameter               | Description                                                           | Default
----------------------- | -----------------------------------------------------------------     | ------------------------------------ |
ACTION                  | Action can be one of the [following](https://github.com/redhat-chaos/krkn/blob/master/docs/node_scenarios.md) | node_stop_start_scenario for aws and node_stop_scenario for vmware |
LABEL_SELECTOR          | Node label to target                                                  | node-role.kubernetes.io/worker       |
NODE_NAME               | Node name to inject faults in case of targeting a specific node; Can set multiple node names separated by a comma      | ""                                   |
INSTANCE_COUNT          | Targeted instance count matching the label selector                   | 1                                    |
RUNS                    | Iterations to perform action on a single node                         | 1                                    |
CLOUD_TYPE              | Cloud platform on top of which cluster is running, supported platforms - aws or vmware                     | aws |
TIMEOUT                 | Duration to wait for completion of node scenario injection             | 180                                |
VERIFY_SESSION          | Only needed for vmware - Set to True if you want to verify the vSphere client session using certificates    | False                               |
SKIP_OPENSHIFT_CHECKS   | Only needed for vmware - Set to True if you don't want to wait for the status of the nodes to change on OpenShift before passing the scenario  | False |

#### Demo
You can find a link to a demo of the scenario [here](https://asciinema.org/a/ANZY7HhPdWTNaWt4xMFanF6Q5)


The following environment variables need to be set for the scenarios that requires intereacting with the cloud platform API to perform the actions:

Amazon Web Services
```
$ export AWS_ACCESS_KEY_ID=<>
$ export AWS_SECRET_ACCESS_KEY=<>
$ export AWS_DEFAULT_REGION=<>
```

VMware Vsphere
```
$ export VSPHERE_IP=<vSphere_client_IP_address>

$ export VSPHERE_USERNAME=<vSphere_client_username>

$ export VSPHERE_PASSWORD=<vSphere_client_password>

```

Google Cloud Platform
```
TBD
```

Azure
```
export AZURE_TENANT_ID=<>
export AZURE_CLIENT_SECRET=<>
export AZURE_CLIENT_ID=<>

```

OpenStack

```
TBD
```

Baremetal
```
TBD
```

**NOTE** In case of using custom metrics profile or alerts profile when `CAPTURE_METRICS` or `ENABLE_ALERTS` is enabled, mount the metrics profile from the host on which the container is run using podman/docker under `/root/kraken/config/metrics-aggregated.yaml` and `/root/kraken/config/alerts`. For example:
```
$ podman run --name=<container_name> --net=host --env-host=true -v <path-to-custom-metrics-profile>:/root/kraken/config/metrics-aggregated.yaml -v <path-to-custom-alerts-profile>:/root/kraken/config/alerts -v <path-to-kube-config>:/root/.kube/config:Z -d quay.io/redhat-chaos/krkn-hub:container-scenarios
