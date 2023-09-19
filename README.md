# 🤏💋 MicroShift

MicroShift ([4.13](https://access.redhat.com/documentation/en-us/red_hat_build_of_microshift/4.13)) setup for my small form factor PC using [this blog as the base of my setup](https://medium.com/@ben.swinney_ce/microshift-homelab-ddf57864c1d0)

## ⛰️ Base Setup

* using a Lenovo / ThinkCentre M930 or something like that as the base. It's an i5 4th gen with 16 gb of RAM
* used a RHEL 9.2 instance with a user called microshift and set the hostname to microshift
* tried using a Kickstart file but failed massively 🤦‍♂️
* some post OS install setup:
```bash
sudo subscription-manager register --username USER --password PASSWD` to register the host
sudo systemctl enable --now cockpit.socket`
sudo dnf install git jq 

sudo subscription-manager repos --enable rhocp-4.12-for-rhel-9-x86_64-rpms   --enable fast-datapath-for-rhel-9-x86_64-rpms
sudo subscription-manager repos --enable rhocp-4.13-for-rhel-9-x86_64-rpms
sudo dnf install -y microshift openshift-clients

sudo dnf -y update
sudo reboot
```

## 🪂 Setup MicroShift

0. setup the DNS entries (A Record for the domain name and CN for *.) to point to the homeserver. Allow the comms for `80`, `443` & `6443` throught the router to the box

1. Setup firewall on the machine
```bash
sudo firewall-cmd --permanent --zone=trusted --add-source=10.42.0.0/16
sudo firewall-cmd --permanent --zone=public --add-port=6443/tcp
sudo firewall-cmd --permanent --zone=public --add-port=443/tcp
sudo firewall-cmd --permanent --zone=public --add-port=80/tcp
sudo firewall-cmd --reload
```

2. Place config in `/etc/microshift/config.yaml` - see `microshift-config.yaml` file in root of this folder

3. Get an [OpenShift Pull Secret](https://cloud.redhat.com/openshift/install/pull-secret) and set CRIO to be able to read it
```bash
sudo cp $HOME/openshift-pull-secret /etc/crio/openshift-pull-secret
sudo chown root:root /etc/crio/openshift-pull-secret
sudo chmod 600 /etc/crio/openshift-pull-secret
```

4. Start microshift `sudo systemctl enable --now microshift.service`

5. Grab Kubeconfig
```bash
mkdir -p ~/.kube
sudo cat /var/lib/microshift/resources/kubeadmin/kubeconfig > ~/.kube/config
chmod go-r ~/.kube/config
```

6. Verify things are working `oc get pods -A`

7. Do something interesting :)

## 🗃️ Storage
I used an external USB3 flash drive (240GB !!)....
change to root:
* `pvdisplay` to show what volumegroups are avail - probs just the `rhel` one
* `fdisk -l` to find the external disk map (in my case `/dev/sdb`)
* `fdisk /dev/sdb` to prepare the disk - used `d` for delete `w` to write
* `pvcreate /dev/sdb` to create a new pv on the device
* `vgcreate microstorage /dev/sdb` create new storage volume group
* `pvdisplay` to verify its been created :)
* place the config file for the storage (see root of this project) in `/etc/microshift/lvmd.yaml` and restart microshift.
* Test the storage with `oc apply -f pvc-test.yaml`

## 🐙 THIS IS GITOPS 🐙

TODO - add ARGOCD STUFF

## Debuggles
* `journalctl -u microshift  -n 100 --no-pager`
* `sudo systemctl restart microshift`

## Next to come 
* adding my flower sensors
* proper SSL Certs :)