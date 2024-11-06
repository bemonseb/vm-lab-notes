# vm-lab-notes
This is documentation for some VM labs I'm working on.

## Lab Index

- [Lab 00 - Setup KVM/QEMU/libvirt](./lab-00-setup-kvm-qemu-libvirt/_index.md): Install the virtualization software on the host machine
- [Lab 01 - Create VM Templates](./lab-01-create-vm-templates/_index.md): Create template VMs for Fedora workstation and Fedora server editions
- [Lab 02 - Virtual Networking](./lab-02-virtual-networking/_index.md): Create some isolated networks
- [Lab 03 - pfSense Firewall](./lab-03-pfsense-firewall/_index.md): Use a pfSense firewall VM to manage and secure virtual networks
- [Lab 04 - OWASP Juice Shop](./lab-04-owasp-juice-shop/_index.md): Create a web server VM that runs the vulnerable web application from OWASP called Juice Shop
- [Lab 05 - Splunk](./lab-05-splunk/_index.md): Add a Splunk SIEM server to monitor the Juice Shop web server
- [Lab 06 - Incident Response](./lab-06-incident-response/_index.md): Exploit vulnerabilities in the Juice Shop web server and use the Splunk server to investigate the security events

## Lab Resources
I'm using my laptop to work on the labs. It's a Dell XPS 7590, with 32 GB RAM and an i9 processor. I'll use the QEMU emulator and libvirt API software to manage the virtual machines and virtual networks.