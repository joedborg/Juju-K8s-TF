# Tensorflow on Kubernetes on AWS with GPU support
## Prerequisites
* Installed `juju`.
* Installed `kubectl`.
* Installed `helm`.
* Access to `p2.xlarge` and `m4.xlarge` instances in `eu-central-1`.
  * This requires contacting AWS support.
  * At the time of writing, `eu-central-1` was the only region I could get these
instances.

## Method
### Juju
1) Bootstrap `juju bootstrap aws/eu-central-1`.
2) Deploy `juju deploy ./k8s-tensorflow.yaml`.
### EFS
3) Setup an EFS drive in `eu-central-1` and assign the correct security groups.  (See https://medium.com/intuitionmachine/kubernetes-gpus-tensorflow-8696232862ca#c678).
4)
  a) `cd ./charts`.
  b) `cp efs/values.yaml ./efs.yaml`.
  c) Update `efs.yaml` to the correct id.
  d) Deploy with `helm install efs --name efs --values ./efs.yaml`.
  e) `cd ../`.
### CNN
5)
  a) `cd ./charts`.
  b) `cp distributed-cnn/values.yaml ./dcnn.yaml`.
  c) Get ingress IP of the k8s cluster and append XIP DNS with `echo $(juju status kubernetes-worker-cpu --format json | jq -r '.machines[]."dns-name"').xip.io`.
  d) Update `dcnn.yaml` to correct the DNS field with the output of the
  previous command.
  e) Deploy with `helm install distributed-cnn --name dcnn --values ./dcnn.yaml`.
  f) Browse to XIP DNS address to see TensorBoard.

## Notes
* The CNN example does not facilitate the GPUs.  Roll you own for this (See https://medium.com/intuitionmachine/kubernetes-gpus-tensorflow-8696232862ca#eb05).
* Renting these instances is mega expensive (around $100 a day), be prepared for
  this!
* If you have issues with the pods at step `5`, the likely issue is that the
  EFS volumes doesn't have the correct security groups assigned.
