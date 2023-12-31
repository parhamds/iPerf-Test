<!--
SPDX-License-Identifier: Apache-2.0
-->

# iPerf Test
## Preparation Phase
### Setup VirtualBox
![UPF overview](./topo.png)
* IP range of HostOnly1 = 192.168.251.0/24
* IP range of HostOnly2 = 192.168.200.0/24
* IP address of enp0s8 of UERANSIM = 192.168.251.5
* IP address of enp0s8 of iPerf-Server = 192.168.200.10
* IP address of enp0s8 of SD-Core = 192.168.251.1
* IP address of enp0s9 of SD-Core = 192.168.200.1

### Setup SD-Core

```
sudo su
sudo apt update
sudo apt upgrade
sudo apt install make 
sudo apt-get remove unattended-upgrades
sudo systemctl stop apt-daily.timer
sudo systemctl disable apt-daily.timer
sudo systemctl disable apt-daily.service
sudo systemctl daemon-reload
sudo sysctl fs.inotify.max_user_instances=2147483647
sudo sysctl fs.inotify.max_user_watches=2147483647
git clone "https://gerrit.opencord.org/aether-in-a-box"
git clone https://github.com/parhamds/iPerf-Test.git
cp -r -f iPerf-Test/systemd/ aether-in-a-box/
cp -f iPerf-Test/Makefile aether-in-a-box/Makefile
rm aether-in-a-box/systemd/macvlan.conf
cd aether-in-a-box
```
DATA_IFACE is the interface connected to UERANSIM, and INT_IFACE is connected to iPerf-Server
```
ENABLE_GNBSIM=false DATA_IFACE=ens4 INT_IFACE=ens3 CHARTS=release-2.0 make roc-5g-models
ENABLE_GNBSIM=false DATA_IFACE=ens4 INT_IFACE=ens3 CHARTS=release-2.0 make router-host
kubectl create namespace omec
sysctl -w net.ipv4.ip_forward=1
iptables -P INPUT ACCEPT
systemctl stop ufw
iptables -I FORWARD -j ACCEPT
```
Then go to the already downloaded resources folder of this repository and change the config of UPF.yaml to the values that suits your test, Then run
```
kubectl apply -n omec -f ../iPerf-Test/resources/
```
Remove every limit from aether config using aether portal
* Go to http://192.168.251.1:31194/#/slice
* Edit AiaB Slice, set Maximum bitrate for both uplink and downlink to maximum value (500Mbps)
* Press update
* Next go to http://192.168.251.1:31194/#/device-group
* Edit AiaB Users and Unknown Inventory and set Maximum bitrate for both uplink and downlink to maximum value (500Mbps)
* http://192.168.251.1:31194/#/basket
* Press commit
#### UPF configs of the tests of this repo:
* For Increasing_UE.sh : 
```
upf.json: "{\"max_sessions_threshold\":2,\"min_sessions_threshold\":2,\"max_sessions_tolerance\":0.4,\"min_sessions_tolerance\":0.1,\"max_cpu_threshold\":1000,\"min_cpu_threshold\":600,\"max_bitrate_threshold\":10625000,\"min_bitrate_threshold\":12500,\"reconciliation_interval\":1,\"auto_scale_out\":true,\"auto_scale_in\":false,\"scalebycpu\":false,\"scalebysession\":false,\"scalebybitrate\":true,\"max_upfs\":3,\"min_upfs\":1,\"init_upfs\":1,\"ueransim\":true,\"access\":{\"ifname\":\"access\"},\"core\":{\"ifname\":\"core\"},\"cpiface\":{\"dnn\":\"internet\",\"hostname\":\"upf\",\"http_port\":\"8080\"},\"enable_notify_bess\":true,\"gtppsc\":true,\"hwcksum\":true,\"log_level\":\"trace\",\"max_sessions\":50000,\"measure_flow\":false,\"measure_upf\":true,\"mode\":\"af_packet\",\"notify_sockaddr\":\"/pod-share/notifycp\",\"qci_qos_config\":[{\"burst_duration_ms\":10,\"cbs\":50000,\"ebs\":50000,\"pbs\":50000,\"priority\":7,\"qci\":0}],\"slice_rate_limit_config\":{\"n3_bps\":100000000,\"n3_burst_bytes\":125000000,\"n6_bps\":100000000,\"n6_burst_bytes\":125000000},\"table_sizes\":{\"appQERLookup\":200000,\"farLookup\":150000,\"pdrLookup\":50000,\"sessionQERLookup\":100000},\"workers\":1}"
```
* For Decreasing_UE.sh : 
```
upf.json: "{\"max_sessions_threshold\":2,\"min_sessions_threshold\":2,\"max_sessions_tolerance\":0.4,\"min_sessions_tolerance\":0.1,\"max_cpu_threshold\":1000,\"min_cpu_threshold\":600,\"max_bitrate_threshold\":10625000,\"min_bitrate_threshold\":1875000,\"reconciliation_interval\":1,\"auto_scale_out\":false,\"auto_scale_in\":true,\"scalebycpu\":false,\"scalebysession\":false,\"scalebybitrate\":true,\"max_upfs\":3,\"min_upfs\":1,\"init_upfs\":3,\"ueransim\":true,\"access\":{\"ifname\":\"access\"},\"core\":{\"ifname\":\"core\"},\"cpiface\":{\"dnn\":\"internet\",\"hostname\":\"upf\",\"http_port\":\"8080\"},\"enable_notify_bess\":true,\"gtppsc\":true,\"hwcksum\":true,\"log_level\":\"trace\",\"max_sessions\":50000,\"measure_flow\":false,\"measure_upf\":true,\"mode\":\"af_packet\",\"notify_sockaddr\":\"/pod-share/notifycp\",\"qci_qos_config\":[{\"burst_duration_ms\":10,\"cbs\":50000,\"ebs\":50000,\"pbs\":50000,\"priority\":7,\"qci\":0}],\"slice_rate_limit_config\":{\"n3_bps\":100000000,\"n3_burst_bytes\":125000000,\"n6_bps\":100000000,\"n6_burst_bytes\":125000000},\"table_sizes\":{\"appQERLookup\":200000,\"farLookup\":150000,\"pdrLookup\":50000,\"sessionQERLookup\":100000},\"workers\":1}"
```
* For Increasing_Traffic.sh : 
```
upf.json: "{\"max_sessions_threshold\":2,\"min_sessions_threshold\":2,\"max_sessions_tolerance\":0.4,\"min_sessions_tolerance\":0.1,\"max_cpu_threshold\":1000,\"min_cpu_threshold\":600,\"max_bitrate_threshold\":10625000,\"min_bitrate_threshold\":12500,\"reconciliation_interval\":3,\"auto_scale_out\":true,\"auto_scale_in\":false,\"scalebycpu\":false,\"scalebysession\":false,\"scalebybitrate\":true,\"max_upfs\":3,\"min_upfs\":1,\"init_upfs\":1,\"ueransim\":true,\"access\":{\"ifname\":\"access\"},\"core\":{\"ifname\":\"core\"},\"cpiface\":{\"dnn\":\"internet\",\"hostname\":\"upf\",\"http_port\":\"8080\"},\"enable_notify_bess\":true,\"gtppsc\":true,\"hwcksum\":true,\"log_level\":\"trace\",\"max_sessions\":50000,\"measure_flow\":false,\"measure_upf\":true,\"mode\":\"af_packet\",\"notify_sockaddr\":\"/pod-share/notifycp\",\"qci_qos_config\":[{\"burst_duration_ms\":10,\"cbs\":50000,\"ebs\":50000,\"pbs\":50000,\"priority\":7,\"qci\":0}],\"slice_rate_limit_config\":{\"n3_bps\":100000000,\"n3_burst_bytes\":125000000,\"n6_bps\":100000000,\"n6_burst_bytes\":125000000},\"table_sizes\":{\"appQERLookup\":200000,\"farLookup\":150000,\"pdrLookup\":50000,\"sessionQERLookup\":100000},\"workers\":1}"
```
* For Decreasing_Traffic.sh : 
```
upf.json: "{\"max_sessions_threshold\":2,\"min_sessions_threshold\":2,\"max_sessions_tolerance\":0.4,\"min_sessions_tolerance\":0.1,\"max_cpu_threshold\":1000,\"min_cpu_threshold\":600,\"max_bitrate_threshold\":10625000,\"min_bitrate_threshold\":5000000,\"reconciliation_interval\":3,\"auto_scale_out\":false,\"auto_scale_in\":true,\"scalebycpu\":false,\"scalebysession\":false,\"scalebybitrate\":true,\"max_upfs\":3,\"min_upfs\":1,\"init_upfs\":3,\"ueransim\":true,\"access\":{\"ifname\":\"access\"},\"core\":{\"ifname\":\"core\"},\"cpiface\":{\"dnn\":\"internet\",\"hostname\":\"upf\",\"http_port\":\"8080\"},\"enable_notify_bess\":true,\"gtppsc\":true,\"hwcksum\":true,\"log_level\":\"trace\",\"max_sessions\":50000,\"measure_flow\":false,\"measure_upf\":true,\"mode\":\"af_packet\",\"notify_sockaddr\":\"/pod-share/notifycp\",\"qci_qos_config\":[{\"burst_duration_ms\":10,\"cbs\":50000,\"ebs\":50000,\"pbs\":50000,\"priority\":7,\"qci\":0}],\"slice_rate_limit_config\":{\"n3_bps\":100000000,\"n3_burst_bytes\":125000000,\"n6_bps\":100000000,\"n6_burst_bytes\":125000000},\"table_sizes\":{\"appQERLookup\":200000,\"farLookup\":150000,\"pdrLookup\":50000,\"sessionQERLookup\":100000},\"workers\":1}"
```
Set slice_rate_limit_config parameters as it suits your test infrustructure. In these examples 100Mbps is allocated to each upf to prevent unwanted behaviour

The unit of bitrate thresholds are Byte per Second
### Setup UERANSIM
Download and install UERANSIM by its [official instruction](https://github.com/aligungr/UERANSIM/wiki/Installation)

Replace ~/UERANSIM/config/open5gs-gnb.yaml and ~/UERANSIM/config/free5gc-gnb.yaml with the ones from this repository on the UERANSIM-config folder.

Then run
```
ip r replace 192.168.252.0/24 via 192.168.251.1
ip r replace 192.168.200.0/24 via 192.168.251.1
apt install iperf3
```

### Setup iPerf-Server
```
ip r replace 192.168.251.0/24 via 192.168.200.1
ip r replace 172.250.0.0/16 via 192.168.200.1
apt install iperf3
```

## Action phase
### iPerf-server
Start 20 iperf server with ports from 5000 to 5019 in 20 screens (recommended, since it allows you to check re resault on iPerf3 server side) or simply run 
```
https://github.com/parhamds/iPerf-Test.git
cd scripts
chmod +x iPerf_Server.sh
./iPerf_Server.sh
pkill iperf3
```
### UERANSIM
Start screen
```
screen
```
In first screen
```
cd ~/UERANSIM/config/
../build/nr-gnb -c free5gc-gnb.yaml
```
Next screen (ctrl+c)
```
cd ~/UERANSIM/config/
../build/nr-ue -c free5gc-ue.yaml -n 20
```
Next screen (ctrl+c)

First check if all UEs are up and running by running ping script from script folder of this Repo
```
./ping.sh
```
It should be finished immediately and all commands should receive their responses.

If any UE had problems, go to the second screen (ctrl+a then 1), disconnect UEs with ctrl+c, then re-run the last command of that screen to re-initiate the UEs.

Then run the script of interest
```
chmod +x Decreasing_Traffic.sh Decreasing_UE.sh Increasing_Traffic.sh Increasing_UE.sh
./Decreasing_Traffic.sh
./Decreasing_UE.sh
./Increasing_Traffic.sh
./Increasing_UE.sh
```
## How does each test work
### Increasing_UE.sh
It starts with 1 UE with 10Mbps, and increase the Number to 20 of UEs keeping the bitrate 10Mbps
### Decreasing_UE.sh
It starts with 20 UE with 10Mbps, and decrease the Number of UEs to 1 keeping the bitrate 10Mbps
### Increasing_Traffic.sh
It starts with 20 UE with 1Mbps, and increase the bitrate of UEs to 10Mbps keeping the Number of UEs 20
### Decreasing_Traffic.sh
It starts with 20 UE with 10Mbps, and increase the bitrate of UEs to 1Mbps keeping the Number of UEs 20

## Change the test
If you ran a test and you want to change the test do these steps:

### On SD-Core
```
kubectl delete namespace omec
```
* change the config of UPF.yaml
* start virtual UPF again:
```
kubectl create namespace omec
kubectl apply -n omec -f .
```
### On UERANSIM
* Stop the gnb and UEs in first and second screen
* Start gnb and UEs again
* Check the functionality of all UEs with ping.sh script
* Run the desired test script

### On iPerf-server
* There is nothing to do with iPerf-server

## Results
To see the Results  go to [here](./Results.md)