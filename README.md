# CloudSimSDN

CloudSimSDN: SDN extension of CloudSim project. Version 2.0 is now availalbe.

## New Features:

* Resource provisioning for NFV in the edge computing environment;
* Simulation framework for NFV in edge and cloud computing (inter cloud data centers);
* Policy supports of Network link selection, VM allocation, Virtual Network Function(VNF) placement, and SFC auto-scaling algorithms;
* Performance evaluation of the framework with use case scenarios.

## Introduction

**CloudSimSDN** is to simulate utilization of hosts and networks, and response time of requests in SDN-enabled cloud data centers.
**CloudSimSDN** is an add-on package to [CloudSim](http://www.cloudbus.org/cloudsim/), thus it is highly recommended to learn how to use CloudSim before using CloudSimSDN.
CloudSimSDN supports calculating power consumption by both hosts and switches. For instance, network-aware VM placement policies can be evaluated using CloudSimSDN. As an example, we will present energy savings in SDN-enabled cloud data center via VM consolidation. If VMs are consolidated to the minimum number of hosts, the unused hosts and switches can be powered off to save more power. We will show two different VM placement policies: Best Fit (MFF, Most Full First) and Worst Fit (LFF, Least Full First).

## Program Dependencies📄
You can download cloudsim-4.0.jar here (https://github.com/Cloudslab/cloudsimsdn/releases/tag/v2.0.1-beta)
or clone [CloudSim (cloudsim-4.0)] src code (https://github.com/Cloudslab/cloudsim), either include all src code or export the jar of the newest version (bugs fixed to support cloudsimsdn-nfv) and
enter the project's root directory and execute `mvn clean install` to install the jar packages into your local maven repository.

Other dependencies are already included.

## Quick Start⚡️
After the mvn build, you could simply run the project's example in IDE's Run Configurations by adding commands in the Arguments:

* For example, to start the simulation example of SimpleExampleInterCloud:
````
LFF example-intercloud/intercloud.physical.json example-intercloud/intercloud.virtual.json example-intercloud/intercloud-example-workload.csv example-intercloud/intercloud-example-workload2.csv
````

* To run StartExperimentSFCEdge:
````
LFF 0 example-edge/edge.physical.json example-edge/edge.virtual.json example-edge/ edge.workload_host1.csv edge.workload_host2.csv
````
* To run StartExperimentSFC:
1 for enable SFC auto-scaling
````
LFF 1 example-sfc/sfc-example-physical.json example-sfc/sfc-example-scale-virtual.json example-sfc/ sfc-example-scale-workload.csv
````

## Package Components
1. org.cloudbus.cloudsim.sdn

  Main components of CloudSimSDN. Core functions are implemented in this package source codes.
  
2. org.cloudbus.cloudsim.example

  Example program. SimpleExample.java is the entry point of the example program. Please follow the code from SimpleExample.java
  This document is to describe the example program.
  Other Scenarios includes:
  Inter cloud data centers, Link Selection Policy, Overbooking Host Resources, QoS, Service Function Chaining, and
  Service Function Chaining in edge computing.
  
3. org.cloudbus.cloudsim.sdn.exmaple.topogenerators

  Example topology generators. Physical / Virtual topology files (inter-clouds, Edge computing, SFC, multi-tier web application, etc.) can be generated by using these generators with customizable parameters. Some distributions can be used within topology generators.
  
4. org.cloudbus.cloudsim.sdn.monitor

  Energy consumption and utilization monitor.
  
5. org.cloudbus.cloudsim.sdn.nos

 Main components of Networking Operation System includes flow channel manager, and extended version of nos for different scenarios. 
  
6. org.cloudbus.cloudsim.sdn.parsers
  
  Parsers for physical topology, virtual topology, and workload.
  
7. org.cloudbus.cloudsim.sdn.physicalcomponents

  SDN-enabled components includes node, link, physical topology, switches (Aggregation, core, edge, gateway, inter-cloud), routing table, extended datacenter, and host.
  
8. org.cloudbus.cloudsim.sdn.policies

  Policies (algorithms) for Host selection, Link selection, Vm allocation, Host overbooking.
  
 9. org.cloudbus.cloudsim.sdn.provisioners
 
  Bandwidth(bw) and CPU(Pe) overbooking provisioners.
  
 10. org.cloudbus.cloudsim.sdn.sfc
 
  Main components of Service Function Chaining (SFC) festures, including SFC Forwarder, SFC policy, auto scaling algorihtms (scaling up and out), etc.
  
 11. org.cloudbus.cloudsim.sdn.virtualcomponents
 
  Flows created in VMs (SDNVM.java) through channel(Channel.java) based on corresponding Flow configuration(FlowConfig.java) are forwarded according to rules(ForwardingRule.java) in SDN-enabled switches. VirtualNetworkMapper includes the main APIs for network traffic forwarding.
  
 12. org.cloudbus.cloudsim.sdn.workload
 
  Core components for workload processing and networking transmission. Request.java represents the message submitted to VM that includes a list of activities(Activity.java) that should be performed at the VM (Processing and Transmission). Furthermore, in some senarios, one request could include another request for the subsequent requests which will be performed at the other VMs.
  
## Input Data
We need to submit three input files to CloudSimSDN: data center configuration (physical topology), resource deployment request (virtual topology), and workloads for VMs.

### Physical topology (Data center configuration)
Configurations of physical hosts, switches and links that consist of SDN-enabled cloud data center. This can input as JSON file.  Please look at sdn-example-physical.json file. 
In this example, data center is configured to operate 100 hosts, 10 edge switches connecting 10 hosts each, and one core switch that connects all edge switches.

* Host nodes
  1. type: "host"
  2. name: name of the host
  3. pes, mips, ram, storage : the host specification
  4. bw: connection bandwidth with the edge switch

* Switch nodes
  1. type: either "core", "aggregate" or "edge"
  2. name: name of the switch
  3. bw: maximum bandwidth support by switch

* Links
  1. source: the name of source node
  2. destination: the name of destination node

###Virtual topology (Resource deployment request)
When customers send VM creation requests to the cloud data center, they provide virtual topology for their network QoS and SLA. Virtual topology consists of VM types and virtual links between VMs. This can input as JSON file. Please look at sdn-example-virtual.json file.

The resource deployment file includes 500 VM creation requests in which three to five VMs are grouped in a same virtual network to communicate with each other. 

* Nodes
  1. type: "vm"
  2. name: name of the vm
  3. pes, mips, ram, size: the VM specification
* Links
  1. name: the name of the link that can be used in workloads. For default link, use "default"
  2. source: the name of source VM
  3. destination: the name of destination VM
  4. bandwidth (optional): specifically requested bandwidth for the link

###Workloads (workload.csv)
After VMs are created in the data center, computation and network transmission workloads from end-users are passed to VMs to be processed. A workload consists of compute processing and network transmission. This can input as CSV file.
Please look at `sdn-example-workload-*.csv` files

Workload file has a long packet transmission between VMs in a same virtual network. Since we should measure power consumption of switches, data transmissions between VMs are necessary to let switches work for the experiment time. To make the experiment simple, we make VMs use network bandwidth in full during their lifetime, so that just one long packet transmission workload for each VM is given in the workload file.

* CSV file structure
  1. Submission time
  2. Submission VM (VM1)
  3. Packet size of the transmission to VM1 (use 0)
  4. Computational workload for VM1
  5. The name of virtual link to transfer packet to the next VM (VM2)
  6. The next VM (VM2)
  7. Packet size of the transmission to VM2
  8. Computational workload for VM2
  9. ... (repeat v ~ viii)

## Simulation Execution
You have to build the project using your IDE or typing `mvn clean install` at the project's root directory.
After that, to execute the example, enter the project's `target` directory and use the following command:

```
java -cp cloudsimsdn-1.0-with-dependencies.jar org.cloudbus.cloudsim.sdn.example.SDNExample <LFF|MFF> [physical.json] [virtual.json] [workload1.csv] [workload2.csv] [...]
```

* ```<LFF | MFF>```: Choose VM placement policy. LFF(Least Full First) or MFF(Most Full First)
* ```[physical.json]```: Filename of physical topology (data center configuration)
* ```[virtual.json]```: Filename of virtual topology (VM creation and network request)
* ```[workload1.csv] ...```: Filenames of workload files. Multiple files can be supplied.

### EXAMPLE:
```
java -cp cloudsimsdn-1.0-with-dependencies.jar org.cloudbus.cloudsim.sdn.example.SDNExample MFF ../dataset-energy/energy-physical.json ../dataset-energy/energy-virtual.json ../dataset-energy/energy-workload.csv > results.out
```

This command will run the simulation using MFF algorithm, and the output is redirected to results.out file.

## Simulation results
The results have five parts.

* Part 1) Detailed result of workloads: shows computational time and transmission time of each workload components. It also shows total response time of each workload.
* Part 2) Average result of workloads: shows the total number of workloads, average rate of all workload requests, and the average response time.
* Part 3) Host power consumption and detailed utilization: shows total power consumption and detailed utilization history (in MIPS) for each host
* Part 4) Switch power consumption and detailed utilization: shows total power consumption and detailed utilization history (in number of active ports) for each switch
* Part 5) Total power consumption: shows total power consumption over the data center with the maximum hosts utilized at the same time

### EXAMPLE:
* Part 1 / 2) In our example, part 1 and 2 (for workload results) is not useful; because the workload is generated solely to make switches work for the whole lifetime of communicating VMs. 
* Part 3 / 4)
```
Host #0: 29653.168930555563
0.0, 4000.0
0.0, 16000.0
0.0, 35200.0
0.0, 51200.0
2390.0, 55200.0
2423.0, 59200.0
...
Switch #103: 27511.461264316662
22660.21001, 2
90180.21001, 3
502117.0, 2
1458312.66651, 0
```
Part 3 and 4 shows the detailed power consumption and utilization level of each host or switch. 
For Host #0, it consumed 29,653 Wh which hosted 4 VMs at the time 0. From the time 0 until 2390, the host utilized 51200 MIPS. 
For Switch #103, it consumed 27,511 Wh for the whole experiment. No traffic was occurred until the time 22660, and 2 ports were active between 22660 and 90180 seconds.

* Part 5)
```
========== TOTAL POWER CONSUMPTION ===========
Host energy consumed: 1848038.3846250002
Switch energy consumed: 92493.37391543222
Total energy consumed: 1940531.7585404324
Simultanously used hosts:30
```
Part 5 is the main result of this example. Using MFF policy, total energy consumption of the data center was 1,940,531Wh and at most 30 hosts were used at the same time.
 
To compare with the result of LFF policy, run the same program with 'LFF' parameter instead of 'MFF'. The result shows that 2,508,871Wh was consumed with LFF policy.
 
## Generate different scenarios
1. Use topology generators (org.cloudbus.cloudsim.sdn.example.topogenerators) to create more complex scenario in larger scale.
2. Implement different VM allocation policy to test different VM placement algorithms
3. Implement different NetworkOperatingSystem to test different network policies.

## Publication
Please cite this paper:
* Jungmin Son, Amir Vahid Dastjerdi, Rodrigo N. Calheiros, Xiaohui Ji, Young Yoon, and Rajkumar Buyya, ["CloudSimSDN: Modeling and Simulation of Software-Defined Cloud Data Centers"](http://ieeexplore.ieee.org/document/7152513/), Proceedings of the 15th IEEE/ACM International Symposium on Cluster, Cloud and Grid Computing (CCGrid 2015), Shenzhen, China, May 4-7, 2015. doi:10.1109/CCGrid.2015.87
