# openebs

https://docs.openebs.io/docs/overview.html


```bash
# build manifests to verify
kubectl kustomize .

# apply to cluster
kubectl apply -k .


# create storagepool: edit cstor-storage.yaml
# StoragePoolClaim/cstor-disk-pool > spec.blockDevices.blockDeviceList
# fill in values from `kubectl get -A blockdevices.openebs.io`
kubectl apply -f ./cstor-storage.yaml
```


## upgrading

See https://github.com/openebs/openebs/tree/master/k8s/upgrades


## metrics and logging

https://github.com/openebs/openebs/tree/master/k8s#optional-enable-monitoring-using-prometheus-and-grafana
https://github.com/openebs/openebs/tree/master/k8s#optional-enable-monitoring-using-prometheus-operator

matrix prefixes:
  openebs* or latest_openebs*.

**maya-exporter**
https://grafana.com/dashboards/9065

https://github.com/coreos/kube-prometheus/blob/master/manifests/0prometheus-operator-0podmonitorCustomResourceDefinition.yaml


**loki**


## debug

```bash
kubectl api-resources --api-group openebs.io

kubectl -n openebs logs -f -l openebs.io/component-name=ndm-operator
kubectl -n openebs logs -f -l openebs.io/component-name=maya-apiserver

kubectl get --all-namespaces blockdevices

kubectl api-resources --api-group openebs.io -o name \
  | xargs -n 1 kubectl get --all-namespaces --show-kind --ignore-not-found

kubectl get storageclasses
kubectl get storagepoolclaim.openebs.io
kubectl get cstorpools
```


## deleting

https://docs.openebs.io/docs/next/uninstall.html

```bash
# (!) before deleting uncomment namespace in manifest
kubectl delete -f ../../manifests/openebs

kubectl get storageclasses -o name | fgrep "openebs-" \
  | xargs -n 1 kubectl delete

kubectl api-resources --api-group openebs.io -o name \
  | xargs -n 1 kubectl delete --all-namespaces --all

kubectl api-resources --api-group openebs.io -o name \
  | xargs -n 1 kubectl delete crd

kubectl delete ValidatingWebhookConfiguration validation-webhook-cfg
kubectl delete namespace openebs

# when stuck
kubectl proxy

kubectl get namespace openebs -o json \
  | jq 'del(.spec.finalizers[] | select("kubernetes"))' \
  | curl --request PUT --header "Content-Type: application/json" \
    --data-binary @- http://localhost:8001/api/v1/namespaces/openebs/finalize
```


## Test

```bash
cat << EOF | kubectl apply -f -
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: test1
spec:
  # storageClassName: cstor-localssd-3x-nowait
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi

---
apiVersion: v1
kind: Pod
metadata:
  name: test1
spec:
  containers:
  - name: alpine-noop
    image: alpine:3.10
    command: ["/bin/sh", "-c"]
    args: [ "tail -f /dev/null" ]
    volumeMounts:
    - name: data
      mountPath: "/srv/data"
  volumes:
  - name: data
    persistentVolumeClaim:
      claimName: test1
EOF
```
