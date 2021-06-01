# Project deployment

To deploy this project in namespace <5GCORENS>

```bash
git clone https://github.com/clustership/ocpopen5gs.git
ce ocpopen5gs/helm-chart

export NS=<5GCORENS>

oc new-project ${NS}
oc create sa 5gcore
oc adm policy add-scc-to-user privileged -z 5gcore

cat <<EOF > values-overrides.yaml
mongodb:
  adminPassword: bikinibottom
  password: patrickstar

open5gs:
  logger:
    level: debug

amf:
  nodePort: 31412 
EOF

helm install -f values-overrides.yaml open5gs .
watch oc get pods

```
