# Definition
An xApp is a software tool used by a RAN Intelligent Controller (RIC) to manage network functions in near-real time. xApps are part of a RIC — a central software component of the Open RAN architecture. It is responsible for controlling and optimizing RAN functions and resources. There are two types of RIC: non-real-time (non-RT) and near-real-time (near-RT).xApp are typically deployed on top of the Radio Intelligent Controller (RIC) and communicates with other components over E2 interface within the O-RAN architecture

# O-RAN
## Intro
O-RAN stands for Open Radio Access Network, which is a new paradigm for designing, deploying, and operating cellular networks. O-RAN networks are built with disaggregated components that are connected via open interfaces and optimized by intelligent controllers. This allows for multi-vendor, interoperable components and programmatically optimized networks through a centralized abstraction layer and data-driven closed-loop control. The O-RAN Alliance is defining a virtualization platform for the RAN and extending the definition of 3GPP and eCPRI interfaces to connect RAN nodes

## Architecture
![](https://hackmd.io/_uploads/r182-xBK3.png)

The main architecture of O-RAN is based on key principles that have been at the center of the Software-defined Networking (SDN) transformation in wired networks in the past 15 years, and have started moving into the wireless domain more recently . The main architectural building blocks of O-RAN include the near-RT and non-RT RICs and the SMO. The O-RAN interfaces include E2, O1, A1, the fronthaul interface, and O2. 

### Main Architectural Building Blocks
#### RIC
* RICs (Radio Intelligent Controllers) are essential components of the O-RAN architecture, facilitating control and optimization of the RAN (Radio Access Network).
* There are two types of RICs: near-RT RIC and non-RT RIC. The near-RT RIC handles real-time control and optimization, while the non-RT RIC handles non-real-time control and optimization.
* RICs connect to the Service Management and Orchestration (SMO) through different interfaces: A1 for both RIC types, E2 for near-RT RIC, and O1 for non-RT RIC.
* RICs support the execution of third-party applications called rApps/xApps, enabling value-added services such as policy guidance, enrichment information, configuration management, and data analytics for RAN optimization and operations. They contribute to the network's flexibility and programmability.
 
#### O-DU (O-RAN Distributed Unit)

* The O-DU is responsible for distributed baseband processing in the O-RAN architecture.
* It performs functions like digitization, modulation/demodulation, and low-level radio signal processing.
* Multiple O-DUs can be deployed in a distributed manner to cover a geographical area, improving scalability and reducing latency.

#### O-CU (O-RAN Centralized Unit)

* The O-CU is responsible for centralized baseband processing in the O-RAN architecture.
* It handles functions such as radio resource management, scheduling, and beamforming.
* The O-CU receives instructions from the O-RAN RIC and coordinates the distributed units (O-DUs) for radio transmission and reception.

#### O-RU (O-RAN Radio Unit)
* Open Interface: The o-RU follows a standardized protocol, allowing interoperability between different vendors' equipment in the radio access network.
* Distributed Architecture: The o-RU works alongside the distributed unit (DU) to provide flexible and efficient radio access capabilities.
* Radio Signal Processing: The o-RU handles functions like modulation, demodulation, encoding, decoding, beamforming, and filtering for effective wireless signal transmission.
* Conversion: The o-RU converts digital baseband signals to analog RF signals for transmission and vice versa, enabling communication between digital and analog domains.
* Remote Configuration and Management: The o-RU can be remotely configured, managed, and monitored, facilitating efficient operation, parameter adjustments, and software updates without physical access.
 
#### SMO
* The SMO (Service Management and Orchestration) is an important part of the O-RAN architecture that takes care of managing the RAN (Radio Access Network) and controlling different types of RICs (RAN Intelligent Controllers) that work in real-time and non-real-time.
* The SMO connects to the RICs through specific interfaces: A1 for both real-time and non-real-time RICs, E2 for real-time RICs, and O1 for non-real-time RICs.
* The SMO handles various management tasks like fixing faults, setting up configurations, keeping track of resources, monitoring performance, and ensuring security. It also manages services and resources, organizes the network's structure, and handles policies for network operations.
* Acting like a bridge, the SMO coordinates different functions across the entire network. This allows special apps within the O-RAN system to gather information about the RAN. The SMO uses this information to make smart decisions about services, computing at the network's edge, and how the network is divided into slices.
* The SMO is a crucial part of the O-RAN architecture that ensures everything runs smoothly and efficiently. It manages the RAN, controls different types of RICs, handles important management tasks, makes informed decisions based on gathered information, and optimizes network operations and services.


# xApp
An xApp is a plug-and-play component that implements custom logic for RAN data analysis and RAN control. It can receive data and telemetry from the RAN and send back control using the E2 interface. An xApp is defined by a descriptor and by the xApp software image, which includes the set of files needed to deploy the fully-functional xApp. The xApp descriptor includes information on parameters needed to manage the xApp, such as autoscaling policies, deployment, deletion, and upgrade information 

## xApp Architecture
![](https://hackmd.io/_uploads/rJ6Y8nQtn.png)

## Functions
* Radio Resource Management: xApps dynamically allocate and manage radio resources to optimize network capacity and quality of service based on real-time network conditions and user demands.
* Network Optimization: xApps analyze network performance data to identify areas for improvement and recommend or automatically configure network parameters for optimal coverage, capacity, and quality. 
* Interference Mitigation: xApps detect and mitigate interference issues by analyzing interference patterns and taking proactive measures such as adjusting transmission power and optimizing antenna configurations. 
* Dynamic Spectrum Management: xApps optimize spectrum usage by dynamically allocating spectrum resources based on traffic demands and interference conditions, maximizing spectral efficiency and capacity. 
* Network Intelligence and Analytics: xApps collect and analyze network data using machine learning and artificial intelligence techniques to extract actionable insights, enabling operators to make informed decisions for network optimization and service orchestration.

## Patent Presentation 
<iframe src="https://docs.google.com/presentation/d/e/2PACX-1vRtdKDKT2UeCzkXh2nxf62bum3DAAALZ9uuJJXhOHpA_vFN3PQeAr98WHkt5mCZ8uF6wjRY-nkJ1SBf/embed?start=true&loop=true&delayms=3000" frameborder="0" width="960" height="569" allowfullscreen="true" mozallowfullscreen="true" webkitallowfullscreen="true"></iframe>

