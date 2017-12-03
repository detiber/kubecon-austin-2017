### TODO

## Troubleshooting
- Troubleshoot issue with multi-arch changeset

## Bootstrapping
- Provision the instances for bootstrapping
  - 1 AWS, 1 Packet, use one of the existing osuosl instances
  - Packet hostname: bootstrap-arm64 ip: 147.75.201.190
  - OSUOSL hostname: openshift-1 ip: 140.211.168.190
  - AWS hostname: bootstrap-amd64 ip: 52.204.174.156
- Configure bootstrap hosts
  - Installed using CentOS 7
```
  sudo yum update -y
  sudo yum install -y golang docker git epel-release
  sudo yum install -y tito createrepo
  sudo reboot
  git clone https://github.com/openshift/origin.git
  cd origin
  git checkout release-3.7
  sed -i s/amd64/<arch>/ Makefile
  git config user.name <name>
  git config user.email <email>
  make build-rpms
```


- Grab the Origin 3.7 source tarball from github
- Copy 3.7 source tarball to each bootstrapping host
- Build binaries and images on each bootstrapping host
- Copy generated binaries to this repo
- Push generated images to openshiftmultiarch dockerhub org
- Deprovision the bootstrapping instances

## Demo Build
- Configure delegated route53 dns zone for the cluster
  - kubecon.paas.ninja
- Provision the cluster instances
  - 1 AWS Master, 2 AWS Nodes, 2 Packet Nodes, 2 OSUOSL nodes
    - Reprovision the OSUOSL bootstrapping instance for one of the nodes
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
