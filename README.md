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
3. Export existing image config from phenix by running `phenix config get image/miniccc > /phenix/switchev-iso15118.yml`.
4. Add necessary packages and scripts to the newly exported image config file.
5. Change the name field in the image config file by giving it a unique name.
6. Remove the overlays field from the image config file.
7. Import the image config file by running `phenix config create /phenix/switchev-iso15118.yml`.
8. Finally, build the image by running `phenix image build -x -c -o /phenix/vmdb switchev-iso15118`.
