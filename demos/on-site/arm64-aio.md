## Bootstrapping
- Install Fedora Atomic Host
- Download the arm64 oc binary
```
mkdir -p ~/bin
curl -s http://binaries.kubecon.paas.ninja/linux/arm64/oc -o bin/oc
chmod +x bin/oc
```
- Start the cluster
```
sudo bin/oc cluster up --image="openshiftmultiarch/origin" --version="v3.7.0-multiarch.0" --skip-registry-check
```
