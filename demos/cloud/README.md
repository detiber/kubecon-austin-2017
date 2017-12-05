## Bootstrapping the build hosts
- Provision the instances for bootstrapping
  - 1 AWS, 1 Packet, use one of the existing osuosl instances
  - Packet hostname: bootstrap-arm64 ip: 147.75.201.190
  - OSUOSL hostname: openshift-1 ip: 140.211.168.190
  - AWS hostname: bootstrap-amd64 ip: 52.204.174.156
- Configure bootstrap hosts
  - Installed using CentOS 7
```
sudo yum update -y
sudo yum install -7 epel-release docker git wget tito
cd /etc/yum.repos.d
sudo wget https://github.com/openshift/release/blob/master/projects/origin-release/base/cbs-paas7-openshift-multiarch-el7-build.repo
cd 
sudo yum install -y golang createrepo bsdtar krb5-devel
sudo systemctl enable docker
echo <<EOF > /etc/docker/daemon.json
{
    "live-restore": true,
    "group": "dockerroot"
}
EOF
sudo usermod -a -G dockerroot <user>
sudo reboot
```

## Build the binaries and images
```
export GOPATH=~/go
export PATH=${PATH}:${GOPATH}/bin
go get -u github.com/openshift/imagebuilder/cmd/imagebuilder
git clone https://github.com/openshift/origin.git
cd origin
git checkout release-3.7
sed -i s/amd64/<arch>/g Makefile
sed -i s/amd64/<arch>/g hack/build-rpm-release.sh
sed -i s/amd64/<arch>/g hack/build-images.sh
git config user.name <name>
git config user.email <email>
git commit -am test
hack/build-base-images.sh
make build-images
for image in `docker images | grep latest | awk '{print $1}' | cut -d '/' -f 2`; do docker tag "openshift/${image}:latest" "openshiftmultiarch/${image}-$(arch):v3.7.0-multiarch.0"; done
docker login
for image in `docker images | grep latest | awk '{print $1}' | cut -d '/' -f 2`; do docker push "openshiftmultiarch/${image}-$(arch):v3.7.0-multiarch.0"; done
```

## Create the manifest list image
```
for image in openvswitch hello-openshift node origin-haproxy-router origin-keepalived-ipfailover origin-gitserver origin-sti-builder origin-deployer origin-f5-router origin-docker-builder origin-recycler origin origin-federation origin-egress-http-proxy origin-docker-registry origin-cluster-capacity origin-template-service-broker origin-service-catalog origin-base origin-pod origin-source; do
  cat << EOF > manifest.yaml
image: openshiftmultiarch/${image}:v3.7.0-multiarch.0
manifests:
EOF
  for arch in x86_64 aarch64 ppc64le; do
    docker pull openshiftmultiarch/${image}-${arch}:v3.7.0-multiarch.0
    echo "- image: 'openshiftmultiarch/${image}-${arch}:v3.7.0-multiarch.0'" >> manifest.yaml
    echo "  platform:" >> manifest.yaml
    if [[ "${arch}" == "x86_64" ]]; then
      echo "    architecture: amd64" >> manifest.yaml
    elif [[ "${arch}" == "aarch64" ]]; then 
      echo "    architecture: arm64" >> manifest.yaml
    else; 
      echo "    architecture: ${arch}" >> manifest.yaml
    fi
    echo "    os: linux" >> manifest.yaml
  done
  manifest-tool push from-spec manifest.yaml
done
```

## DNS Configuration
- Delegated kubecon.paas.ninja to a Route53 Hosted Zone
- openshift.kubecon.paas.ninja -> AWS m4.xlarge instance for cluster master
- node1.aws.kubecon.paas.ninja -> AWS m4.xlarge instance for cluster node
- node2.aws.kubecon.paas.ninja -> AWS m4.xlarge instance for cluster node
- node1.packet.kubecon.paas.ninja -> Packet Type 2A instance for cluster node
- node2.packet.kubecon.paas.ninja -> Packet Type 2A instance for cluster node
- node1.osuosl.kubecon.paas.ninja -> OSUOSL m1.xlarge instance for cluster node
- node2.osuosl.kubecon.paas.ninja -> OSUOSL m1.xlarge instance for cluster node

### Cluster Build
- On all hosts:
```
cd /etc/yum.repos.d
wget http://rpms.kubecon.paas.ninja/kubecon-demo.repo
yum install -y origin-clients docker
echo 'STORAGE_DRIVER: overlay2' > /etc/sysconfig/docker-storage-setup
sudo systemctl start docker

```

### TODO
## Demo Build
- Use the previously generated binaries and images to start a cluster using oc cluster up/oc cluster join
- Create a CI workflow for building the multi-arch binaries and images
- Work on getting https://github.com/gnunn1/summit-game-ansible running on the multi-arch cluster
- Work on updating the game to show users the arch of the node they are playing on
- Work on getting pacman game running on cluster with mongo distributed across arches
- Update pacman game to show arch instead of cluster

## Demo Content
- Work up rough script
- Slides describing demo
- Architecture diagrams
- How to reproduce
