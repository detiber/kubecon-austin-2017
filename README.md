## Bootstrapping
- Install Fedora Atomic Host
- Download the arm64 oc binary
```
mkdir -p ~/bin
curl -s http://binaries.kubecon.paas.ninja/linux/arm64/oc -o bin/oc
chmod +x bin/oc
```
- Add 172.30.0.0/16 to registries.block/registries in /etc/containers/registries.conf
- Restart docker
- Start the cluster
```
sudo bin/oc cluster up --image="openshiftmultiarch/origin" --version="v3.7.0-multiarch.0" --public-hostname=<ip address>
```
- Deploy the guestbook app
```
curl -s https://raw.githubusercontent.com/detiber/kubecon-austin-2017/master/guestbook-all-in-one.yaml -o guestbook-all-in-one.yaml
oc create -f guestbook-all-in-one.yaml
```
