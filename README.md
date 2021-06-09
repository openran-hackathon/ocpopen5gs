# Open5gs deployment for OpenShift

## Purpose

This project contains the helm chart to deploy open5gs (https://open5gs.org/) on OpenShift taking care of using Red Hat base images or certified containers and reducing the privileged requirements.
Red Hat UBI (Universal Based Image) are used to build the open5gs containers.


## Project deployment

To deploy this project in namespace <5GCORENS>

```bash
git clone https://github.com/clustership/ocpopen5gs.git
ce ocpopen5gs/helm-chart

export NS=<5GCORENS>

oc new-project ${NS}
```

Unfortunately the upf container required privileged SCC (Security Context Constraint) to configure the tun device used for communication with user equipment (UE).
While waiting for the development of a Kubernetes device plugin to manage tun device allocation to containers to reduce this privleged requierments, you must ask you cluster administrator to issue the following command in your new project.


```bash
oc adm policy add-scc-to-user privileged -z 5gcore
```

Then create a values-overrides.yaml file to customize your helm deployment and install the chart in you namespace.

```bash
cd helm-chart
cat <<EOF > values-overrides.yaml
mongodb:
  adminPassword: bikinibottom
  password: patrickstar

open5gs:
  logger:
    level: info

#
# Change nodePort to a free available
#
amf:
  nodePort: 31412 
EOF

helm install -f values-overrides.yaml open5gs .
watch oc get pods

```


After installation, check that upf pods is running. Check that Replicasets current state match the desired state.
If not, check replicaset status by using:

```bash
oc describe rs <rs-name>
```

## TODOS

* Write a device plugin to manage tun device allocation
* Use initContainer and readiness probes to control start order of each containers (mongo and upf before smf before other CNFs).
* Write a open5gs scc to reduce unused privleges
* Change helm chart to support conditional déployment of CNFs (using PCF or UPF from other namespaces and/or vendors)
* Use multus to create all segregated networks used in 5G Core
