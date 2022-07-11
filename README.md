# sandia-iso15118
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
Instruction is given in the repository [readme file](https://github.com/sandia-minimega/phenix) and also in the [project website](https://phenix.sceptre.dev/). But a more straight forward instructionis given below.
