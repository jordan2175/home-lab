# Cluster Notes

Here are some questions you may need to ask yourself when building your own
Raspberry Pi cluster. 

 1. What is the purpose of this cluster?
   - To learn about various technologies
     - Docker
     - Docker Compose
     - Docker Swarm
     - Kubernetes
     - Ansible
     - Terraform
     - etc..
   - To do some sort of parallel compute
   - To have a bunch of individual computers for different tasks

 2. How many nodes will you need in the cluster to do what you need?

 3. How much ram and flash storage do you need?
   - For flash, what do you need to store on the node itself versus a mounted volume
   - For ram, what does each job need and how many per node will you run

 4. Are you going to get boards with eMMC Flash storage or use an SSD?
   - You cannot use both an SSD and the eMMC flash built into the board at the same time
   - SSDs can be slow and cost money they are also less forgiving on power off
     - However, they are often much easier and faster to provision
   - Some carrier boards for Compute Modules will have a slot for an SSD on the back
     - But you cannot use that SSD slot if the module (CM4/CM5) has eMMC flash

 5. How are you going to power each node?
   - Inidvidaul power supply
   - POE (power over ethernet)
     - This will require a POE+ hat for each node and a POE+ switch
   - Backplane for compute modules (turingpi or similar)

 6. What OS are you going to use for the individual nodes in the cluster?
   - A prebuilt official Raspberry Pi image
   - A prebuilt official image from your favorite distro (e.g., Ubuntu)
   - Roll your own

 7. How are you going to get the initial OS on to each node?
   - Flash the eMMC memory 
   - Flash a MicroSD Card
   - Flash an SSD/SATA/nVME
   - Boot from the network

 8. How are you going to do the initial configuration of each node?
   - Cloud Init
   - Cloud Init + Ansible
   - Manually
   - Bootstrapping problems to consider
     - Initial connection to the node
     - Get right users on each node
     - Set the password for each user for local privilege access (sudo)
     - How to secure / lock it down after everything is setup

 9. How are you going to do networking? 
   - Each node will be directly and individually addressable
   - You will need to go through a gateway machine to see them
     - Addressable via a route through the gateway machine
     - Gate must proxy the connections, meaning connect to gateway and then to nodes

 10. Where are you going to store common files that each node needs?
   - On a single node?
   - On a mounted volume to a shared external drive?

 11. How are you going to remotely connect to each node?
   - SSH with username and passwords
     - This is not as secure as using key pairs
     - This is really only viable for a small number of devices
     - Once you have many, it becomes a pain always typing in a password
   - SSH with public/private key pairs
     - Are you going to assign a password to protect the private key
     - Assigning a password is more secure 
     - But quickly becomes painful with large numbers of nodes
       - But you can use the ssh-agent to cache that password

 12. Setup the /etc/hosts file to find all of your nodes with short names
 
