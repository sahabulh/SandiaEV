# SandiaEV
## Introduction
This project is a research collaboration between UNM, SNL and few other organizations. The goals of the project are to:
- virtualize the EV charging PKI environment using phenix-minimega
- answer some research questions:
  1. What’s the scalability of certificate generation/distribution?
  2. What’s the anticipated network load?
  3. Are there issues with key generation, distribution, and registration?
  4. How will revocation mechanisms scale?
  5. Will failback operating modes work when, e.g., the Root-CA is un-reachable?
  6. Are there interoperability issues with mixed ISO 15118-2, ISO 15118-20, SAE PKI Vehicles and EVSE?

## Useful repositories
- [ISO 15118](https://github.com/SwitchEV/iso15118). Python Implementation of the ISO 15118 -2 and -20 protocols.
- [phenix](https://github.com/sandia-minimega/phenix). Orchestration tool for [minimega](https://github.com/sandia-minimega/minimega) which can virtualize the PKI environment.

## Installing required softwares
### ISO 15118
Clear instruction is given in the repository readme file. Please follow [this](https://github.com/SwitchEV/iso15118#readme).
### phenix
Instruction is given in the repository [readme file](https://github.com/sandia-minimega/phenix) and also in the [project website](https://phenix.sceptre.dev/). But a more straight forward instruction is given below:
1. Install [Docker](https://docs.docker.com/engine/install/ubuntu/) and [Docker Compose](https://docs.docker.com/compose/install/).
2. Clone the [phenix git repository](https://github.com/sandia-minimega/phenix.git).
3. From a terminal, change into the phenix/docker directory.
4. Run `sudo docker-compose build` (this can take a while depending on your system).
5. Run `sudo mkdir /phenix` to create the phenix installation folder.
6. Run `sudo docker-compose up -d phenix`.
7. Browse to http://localhost:3000 to access the phenix UI.

To restart phenix:
1. From a terminal, change into the phenix/docker directory.
2. Run `sudo docker-compose down`.
3. Run `sudo rm -rf /etc/phenix`.
4. Run `sudo docker-compose up -d phenix`.

## Building an image using phenix
1. Enble root user by running `sudo su`.
2. Add an alias to the system for phenix by running `alias phenix="docker exec -it phenix phenix"`.
3. Export existing image config from phenix by running `phenix config get image/miniccc > /phenix/switchev-iso15118.yml`. Here 'switchev-iso15118.yml' is the name of the downloaded image config file.
4. Add necessary packages and scripts to the newly exported image config file.
5. Change the name field in the image config file by giving it a unique name.
6. Remove the overlays field from the image config file. See the [Image Config](https://github.com/sahabulh/SandiaEV/tree/main/Image%20Config) folder for a prepared file with all the necessary packages.
7. Import the image config file by running `phenix config create /phenix/switchev-iso15118.yml`.
8. Finally, build the image by running `phenix image build -x -c -o /phenix/vmdb switchev-iso15118`.

## Running VMs
1. Browse to http://localhost:3000 to access the phenix UI.
2. Create or import a network topology using the [Configs](http://localhost:3000/configs) tab. Visit the [phenix website](https://phenix.sceptre.dev/configuration/#topology) to learn how to create topologies. Check the [Topology](https://github.com/sahabulh/SandiaEV/tree/main/Topology) folder for some prepared examples.
3. Go to the [Experiments](http://localhost:3000/experiments) tab to create a new experiment. Select the correct topology.
4. Run the experiment by clicking the red 'stopped' button. If everything is okay, the experiment will run and the button will turn to a green 'started' button. Stopping the experiment follows the similar process.
5. Click on the name of the experiment to open the experiment panel.
6. Click on the screenshot of a VM to access it. A new tab will be created for each VM.

## Accessing the VMs from the host machine
If you want to ping the VMs from the host machine or vice versa, it will not work. The VMs do get virtual interfaces created but they are isolated on the OVS bridge they're connected to. The easiest way to access the VMs is to "tap" the bridge with an addressable interface on the host. Here are the steps:
1. Clone the [minimega](https://github.com/sandia-minimega/minimega) git repository.
2. Open the terminal. Enble root user by running `sudo su`.
3. Add the minimega bin folder to the PATH variable.
4. Check the vlan name from the phenix experiment panel (not the experiment tab). It should be under the network column. Ignore the number in the parenthesis. For exmaple, if the entry under the network tab is "lan (101)", the vlan name is "lan".
5. Check the bridge name using `ifconfig` in the host machine. It should be "phenix".
6. Check the ipv4 subnet of the VMs. Choose an ip address within the subnet excluding the already used ones. It will be used as the ip address for the tap interface. For example, if the ip of the VMs are 10.1.2.101-110, you can choose 10.1.2.1/24 or 10.1.2.254/24 as the ip address for the tap.
7. Execute this to create the tap: `minimega -e tap create "vlan name" bridge "bridge name" ip "ip address" "tap name"`. Choose a tap name of your choice. For example, the complete command can be: `minimega -e tap create lan bridge phenix ip 10.1.2.254/24 nat0`
8. The VMs and the host will be able to communicate with each other now. Test by pinging.

## Accesing the internet from the VMs
By default, the VMs will not be able to access the internet. Test by pinging www.google.com. To enable access to the internet, on the host side, we will have to enable ip forwarding and configure the iptable. On the VM side, we need to add the host ip as the default gateway and configure the DNS. The steps are as follows:
### Configuring the host machine
1. Create a tap interface following the instruction in the previous section.
2. Execute this to enable ip forwarding in the host machine: `sysctl -w net.ipv4.ip_forward=1`.
3. Execute the following 5 commands in order. You may need to change `eth0` to match the interface on the host machine with Internet access. Also, You may need to change `nat0` to match the name of the tap interface you have just created.</br>
   `iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE`</br>
   `iptables -A INPUT -i nat0 -j ACCEPT`</br>
   `iptables -A INPUT -i eth0 -m state --state ESTABLISHED,RELATED -j ACCEPT`</br>
   `iptables -A OUTPUT -j ACCEPT`</br>
   `iptables --policy FORWARD ACCEPT`
### Configuring the virtual machines
1. Login as root.
2. Check if the defalut gateway is set to the tap interface ip address of the host machine.
3. If not, use `ip route delete default` to delete the wrong default gateway.
4. Now execute `ip route add default via 10.1.2.254` to set the correct default gateway. Here, 10.1.2.254 is the tap interface ip address of the host machine.
5. Check if you can ping google.
6. If not, edit `/etc/resolv.conf` by adding the address of google public DNS (8.8.8.8 and/or 8.8.4.4) or any other DNS. The entry should look like this:</br>
   `nameserver 8.8.8.8`</br>
   `nameserver 8.8.4.4`
7. Test by pinging google.
