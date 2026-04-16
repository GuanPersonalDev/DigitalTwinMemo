---
title: "Digital twin driven smart factories: real time physics based co-simulation using edge a.i. and federated learning"
source: "https://www.nature.com/articles/s41598-025-28466-9"
author:
  - "[[V. Padmavathi]]"
  - "[[R. Kanimozhi]]"
  - "[[R. Saminathan]]"
published: 2025-12-09
created: 2026-04-16
description: "Thanks to Industry 4.0, digital twin technology now leads the way in smart manufacturing, enabling the real-time replication of devices to facilitate maintenance, process improvements, and smooth operation. Still, many conventional Digital Twin approaches heavily rely on cloud solutions, which result in high latency, limited scaling options, and increased risks to data privacy. However, real-time, physics-enabled simulation remains very challenging for making fast and accurate choices in busy factories due to difficulties with computer speed and network capacity. We introduce a new method that combines Digital Twin, Edge AI, and FL to make co-simulation in smart factories quick, efficient, and secure. Moving the learning and simulation steps to edge devices near the factory floor enables real-time data processing and inference. Federated Learning enables these edge devices to share and learn from one another, ensuring that important production data remains within the factories. This engine, based on FMI, enables the real-time co-simulation of physical events with their virtual counterparts, allowing the system’s behavior to be tested under various operating modes. The framework achieved a latency reduction of up to 35%, a 28% decrease in cloud usage, and a 13.2% throughput gain compared to cloud-only architectures, highlighting its scalability and industrial applicability. Additional ways to enhance this area include the use of blockchain, automated learning updates, and allowing machines to transfer knowledge from plant to plant."
tags:
  - "clippings"
---
## Abstract

Thanks to Industry 4.0, digital twin technology now leads the way in smart manufacturing, enabling the real-time replication of devices to facilitate maintenance, process improvements, and smooth operation. Still, many conventional Digital Twin approaches heavily rely on cloud solutions, which result in high latency, limited scaling options, and increased risks to data privacy. However, real-time, physics-enabled simulation remains very challenging for making fast and accurate choices in busy factories due to difficulties with computer speed and network capacity. We introduce a new method that combines Digital Twin, Edge AI, and FL to make co-simulation in smart factories quick, efficient, and secure. Moving the learning and simulation steps to edge devices near the factory floor enables real-time data processing and inference. Federated Learning enables these edge devices to share and learn from one another, ensuring that important production data remains within the factories. This engine, based on FMI, enables the real-time co-simulation of physical events with their virtual counterparts, allowing the system’s behavior to be tested under various operating modes. The framework achieved a latency reduction of up to 35%, a 28% decrease in cloud usage, and a 13.2% throughput gain compared to cloud-only architectures, highlighting its scalability and industrial applicability. Additional ways to enhance this area include the use of blockchain, automated learning updates, and allowing machines to transfer knowledge from plant to plant.

## Introduction

Manufacturing operations have undergone significant changes due to Industry 4.0, which combines cyber-physical systems, the Internet of Things (IoT), artificial intelligence (AI), and real-time analysis. The Digital Twin (DT) is at the heart of this transformation, enabling continuous supervision, simulation, and optimization of manufacturing systems. Digital Twins have been found to reduce instances of equipment downtime, increase productivity, enable future-based repairs, and enhance decision-making accuracy. At the same time, several technical and operational difficulties delay the complete development of smart factories that work independently and intelligently. A main obstacle is found in relying on large, central cloud services for storing data and running models [^1]. Due to bandwidth limitations, long message latency, and an increased risk of security attacks, these architectures are not well-suited for use in industrial settings that require real-time operation. Co-simulation, which merges physical and virtual results in real time, is another major challenge. The majorities of data transformation frameworks currently uses straightforward data models or rely on simulations because they are unable to handle larger datasets, resulting in less accurate and less beneficial insights when quick decisions are required.

This paper proposes a framework to address these problems by combining Edge AI and FL for real-time, secure, and efficient co-simulation in smart factories equipped with digital twins [^2]. With Edge AI, it is possible to perform inference directly on edge devices located near the assets that need to be controlled, providing fast results. Federated Learning enhances this approach by training models at decentralized sites rather than transferring private data to a central server, thereby maintaining both privacy and regulatory compliance. Figure [1](https://www.nature.com/articles/s41598-025-28466-9#Fig1) shows the Digital Twin overview.

**Fig. 1**

![Fig. 1](https://media.springernature.com/lw685/springer-static/image/art%3A10.1038%2Fs41598-025-28466-9/MediaObjects/41598_2025_28466_Fig1_HTML.png)

[Full size image](https://www.nature.com/articles/s41598-025-28466-9/figures/1)

Overview of digital twin.

Using this common method, accurate physics-based simulations can work in cooperation and be synchronized across various nodes using standard protocols, such as the Functional Mock-up Interface. Thanks to FL, smart factories can identify problems and make adjustments on the go at a large scale, all while utilizing a minimal amount of cloud resources.

**Main contributions**.

This paper makes four main contributions, which are highlighted below.

- A new digital twin framework utilizing edge technology for real-time co-simulation with physics-based models, applicable across multiple sectors.
- Using Federated Learning to allow training of a model on decentralized smart factory units, ensuring privacy protection for the data.
- Building a real-time simulation engine using FMI guidelines so that real and simulated systems stay in perfect sync.
- By applying it in situations such as predictive maintenance and improving production, we can confirm that latency, scalability, and model accuracy have improved.

Next, we discuss related studies (Sect. 2), explain the system architecture and methods (Sect. 3), demonstrate how to use the system (Sects. 4 and 5), and present performance tests (Sect. 5). Conclusions and suggestions for future development follow this.

## Related work

Over the past few years, experts have become increasingly interested in the combination of digital twins, edge computing, and machine learning, particularly in the context of Industry 4.0 and smart manufacturing. Here, we discuss important progress made in four fields: (i) Digital Twins in Smart Manufacturing, (ii) Real-Time Co-Simulation and Physics-Based Modeling, (iii) Edge AI in Industrial Systems, and (iv) Federated Learning for Industrial AI.

### Using digital twins for manufacturing

Digital Twins have advanced from being simple 3D images to virtual representations that continuously match what is happening in real life. The work [^3] laid the groundwork for DT-based smart manufacturing by introducing ideas for integrating different phases and predicting outcomes. However, as most of these methods are cloud-based, they do not respond quickly enough in a factory environment. Furthermore, many DT approaches focus on a single application, such as condition monitoring, and therefore do not provide a comprehensive real-time view of the workspace.

### Real time co-simulation as well as modeling with physics

Thanks to physics-based modeling, crucial real-time decisions in industrial environments become much easier. Modelers can mix and match different systems and areas by using Modelica, Simulink, and OpenModelica through the FMI standard. Co-simulation can be successfully applied in cyber-physical energy systems [^4]. However, many of these techniques assume access to vast amounts of processing power and do not address how they might be utilized with resource-limited edge devices or in situations where tasks must be performed in real time.

### Using edge AI in the industry

With Edge AI, computing occurs on the device where the data is collected, enabling the system to respond much more quickly and alleviate pressure on the main cloud. Such algorithms are effectively utilized in machine vision, robot control, and the detection of abnormalities in smart manufacturing [^5]. Systems, including NVIDIA’s Jetson ecosystem and Google’s Edge TPU, now enable lightweight inference on power-saving devices. However, the majority of edge AI is used in isolation, resulting in little communication or coordination between devices, which is a drawback for making smart factories as efficient as possible.

### Using federated learning in industrial artificial intelligence

In 2017, Google introduced Federated Learning, a technique that enables multiple groups to collaboratively train a model without sharing their data at a central location. This method is particularly crucial in industrial settings, where concerns about data privacy, ownership, and compliance are paramount. To investigate federated learning to understand its applications in the mobile and healthcare sectors [^6]. FL has found applications in manufacturing for detecting product defects, forecasting energy needs, and conducting predictive maintenance [^7]. As in recent reviews, the shift from Industry 4.0 to Industry 5.0 is not only technological but also involves a redesign of new manufacturing paradigms that are human-centric, sustainable, and resilient. In a comparative study, “Review of Manufacturing System Design in the Interplay of Industry 4.0” (2024) describes how cyber-physical integration and AI-driven self-direction of Industry 4.0 evolve into collaborative, adaptive, and value-focused systems of Industry 5.0. The suggested Digital Twin-Edge AI-Federated Learning architecture aligns directly with this evolution, as it connects the concept of decentralized intelligence with the principles of socially responsible production objectives, allowing production factories not only to be efficient but also to be adaptive and sustainable.

**Fig. 2**

![Fig. 2](https://media.springernature.com/lw685/springer-static/image/art%3A10.1038%2Fs41598-025-28466-9/MediaObjects/41598_2025_28466_Fig2_HTML.png)

[Full size image](https://www.nature.com/articles/s41598-025-28466-9/figures/2)

Integration of digital twin with industry 4.0.

In Fig. [2](https://www.nature.com/articles/s41598-025-28466-9#Fig2), the use of FL in conjunction with physics-based digital twins and real-time co-simulation is rarely explored.

Despite achievements in many of these areas, three important gaps remain.

- No integrated systems are available that allow computers at the edge to run both DTs and real-time simulations at once.
- There is no coordination of intelligence at the edge, despite FL, which is needed for adjusting the products together.
- There is limited research on how physical systems can communicate with virtual simulations that run on different hardware platforms.

This work aims to address these gaps by presenting a flexible edge system that incorporates Digital Twins, Edge AI, and Federated Learning for up-to-date physics-based virtual simulations in smart factories.

## System architecture and methodologies

It includes illustrations of the proposed framework and explanations of the methods used to achieve real-time simulation for digital twin-driven smart factories. The system is designed to be flexible, support growth, and protect users’ privacy through the integration of Edge AI and federated learning [^8]. In the following subsections, we will outline both the primary features and the operational processes of the architecture.

### Overview of the platform

The proposed architecture consists of three layers. Physical layer is the term used for Shop Floor Systems. At this level, sensors, actuators, PLCs, and industrial machines constantly provide current data on temperature, vibration, torque, and throughput [^9]. Together, these nodes on the edge make up the Edge Intelligence Layer. By physical assets, edge nodes such as the Jetson Nano, Raspberry Pi, or industrial IoT gateways can host AI models and simulators that are lightweight enough to run on them. These machines operate within their environments, perform computations, and collaborate to learn from each other through federated learning. Functional Mock-up Units (FMUs) for real-time physics modeling are also implemented in this space using engines that comply with the Functional Mock-up Interface (FMI) standard [^10]. Central to the model is a cloud server that coordinates model training, shares global updates, and organizes the system. It also stores a digital version of the factory and provides a dashboard for viewing and analyzing data.

### A physics-based engine for performing co-simulation in real-time

The action of particular factory equipment is accurately mimicked through a co-simulation engine developed using the FMI standard. Every subsystem in a factory (such as robotic arms, conveyors, and milling machines) is modeled using Functional Mock-up Units, which enable multi-domain simulation (mechanical, thermal, electrical) on many edge devices.

The following are especially important aspects:

- Ensures that there is no timing difference between the physical and virtual parts of the system.
- Hybrid modeling uses both continuous equations and logical, discrete control.
- Managing latency: Improves the accuracy of the prediction, depending on the task severity, by adjusting the simulation resolution as needed.

**Integration of FMUs with edge AI models**:

Every Functional Mock-up Unit (FMU) is deployed together with a lightweight Edge AI module on the same edge device (e.g., Jetson Nano or Raspberry Pi). It is integrated according to the Functional Mock-up Interface (FMI) standard. It provides the AI module with real-time sensor data, as well as simulation states, via an event bus. This two-way streamlines the physical measurements, as well as simulations, at every time step. The AI models are usually with low-latency MobileNet or Tiny-YOLO models (3–5 convolutional layers, ReLU activation) and are trained with the Adam optimizer (0.0010.005 learning rate, batch size 32). These edge models are used to provide local updates through Federated Learning, which enables global optimization without exchanging raw data.

### Edge AI converter

The Edge AI module is utilized at every edge node to detect issues, make predictions, and optimize workflows. These types of applications use lightweight neural networks (for example, MobileNet and Tiny-YOLO) to either classify or measure sensory data [^11]. Millisecond response times are achievable through inference done right on the device. Adaptive Low Power Method: Enables the model to adjust for both accuracy and power consumption.

### Federated learning module

We employ a Federated Learning approach, enabling distributed edge nodes to collaborate without sharing any data. How training Works: Every edge node learns with its data and uploads only gradients to the central node for aggregation [^12]. Aggregation Strategy: The cloud controls the model’s updates by aggregating updates and then redistributing the merged global model. Data privacy is increased using differential privacy and homomorphic encryption during Gradient exchange. Nodes join the training process when they can and utilize asynchronous FL versions when other nodes are unavailable. In addition to distributed training, the suggested federated learning model can be expanded to organize product lifecycle processes across multiple factories. FL enables the ongoing refinement of production strategies by safely accumulating lifecycle data, including design changes and maintenance results, without compromising valuable intellectual property. This idea aligns with recent research in federated learning-enabled smart manufacturing and product lifecycle management, where model updates capture product evolution, design intent, and performance feedback to ensure the global optimization of distributed plants and the protection of proprietary data.

### Middleware for communication

The communication is easy between systems that use different hardware and software, the system integrates:

- **OPC unified architecture**: To allow different pieces of industrial equipment and digital twins to talk to each other routinely.
- **MQTT/CoAP**: So that edge devices may quickly and reliably communicate messages with the cloud aggregator.
- **Simulation event bus**: This custom middleware ensures that all FMUs start and end their tests simultaneously.

**Fig. 3**

![Fig. 3](https://media.springernature.com/lw685/springer-static/image/art%3A10.1038%2Fs41598-025-28466-9/MediaObjects/41598_2025_28466_Fig3_HTML.png)

[Full size image](https://www.nature.com/articles/s41598-025-28466-9/figures/3)

A simplified view of the system model.

A simplified view of the system model is shown in Fig. [3](https://www.nature.com/articles/s41598-025-28466-9#Fig3).

### Workflow summary

1. In real-time, information from the factory machines’ sensors is received.
2. Edge AI performs quick actions on information (such as detecting faults) at the point of data capture.
3. FMUs model the performance of necessary parts using continuous sensor data in real.

> \-world applications.

1. From time to time, federated training applies updates to models at all edge nodes.
2. Both these results and insights are available on the cloud dashboard, allowing everyone to view them, plan for the long term, and provide feedback.

**Fig. 4**

![Fig. 4](https://media.springernature.com/lw685/springer-static/image/art%3A10.1038%2Fs41598-025-28466-9/MediaObjects/41598_2025_28466_Fig4_HTML.png?as=webp)

[Full size image](https://www.nature.com/articles/s41598-025-28466-9/figures/4)

System architecture.

In Fig. [4](https://www.nature.com/articles/s41598-025-28466-9#Fig4), the system’s modular and hierarchical structure enables it to perform highly, operate very quickly, and preserve data sovereignty, scalability, and the ability to handle tight constraints—all of which are necessary for today’s smart factories [^13]. The system designed for digital twin-driven smart factories consists of three layers that support instant, physics-based joint simulation using Edge AI and Federated Learning. To start, the Physical Layer is comprised of industrial robots, belt systems, and sensors that generate information in real time. Edge Layer devices use AI and physics to access this data and perform simulations by using Functional Mock-up Units. Because they are automated, these sensors can detect issues, predict system faults, and streamline processes with low-latency results [^14]. This layer aggregates scattered AI models through Federated Learning, ensuring that data remains private and the system’s intelligence continually improves [^15]. The main dashboard allows operators to view the system’s performance in real-time. By incorporating distributed intelligence and high-quality simulation, this architecture enhances manufacturing security, scalability, and responsiveness. In the proposed system, the Physical, Edge, and Cloud Layers all contribute to making smart factories capable of acting in real-time with privacy-preserving, controlled, and intelligent functions.

**Physical Layer**:

At this layer, you have machines, sensors, actuators, and programmable logic controllers (PLCs) that continuously collect real-time data for the entire network. They are key to AI insights and also to physics-based computer modeling.

**Edge Layer**:

Edge nodes (such as Jetson Nano or Raspberry Pi) are placed near the machines to enable local data processing. They provide real-time inference for AI models, such as fault detection and anomaly detection, as well as Functional Mock-up Units that enable the simulation of systems using the Functional Mock-up Interface standard [^16]. It supports fast decisions and keeps the data private.

**Cloud Layer**:

An orchestration server receives new model updates from edge nodes that are working together with Federated Learning. To create a global model, it operates using FedAvg or similar methods and then distributes the model to the edge devices. In the cloud, system metrics are displayed through dashboards, the factory is modeled digitally, and long-term planning and analysis can be conducted [^17]. Due to the architecture, there is continuous synchronization between the factory’s equipment and its software twin, enabling quick responses and collaborative learning across factory units with no impact on scalability or personal data privacy.

**Pseudo-code: Federated Edge-CoSim Optimizer (FECO)**.

**Input**:

D <sub>i</sub> → Sensor signals are sent in real-time by edge node i. At node i, there is a Functional Mock-up Unit that simulates physics. The model at node i is called the M <sub>i</sub> model.

T <sub>sync</sub> → How often the simulation is synchronized.

T <sub>FL</sub> → How often a new global model is generated through updating.

θ <sub>i</sub> → represents the weights used at node i.

A → Function for combining different models (e.g., FedAvg).

**Initialize**:

θ <sub>i</sub> is set with pre-trained weights.

t = 0.

Loop: When the factory is working.

Run for each edge node individually.

1. Record the values D <sub>i(t)</sub> from physical sensors.
2. Predict: y <sub>i(t)</sub>  = M <sub>i</sub> (D <sub>i(t)</sub>).
3. Simulate: S <sub>i(t)</sub>  = f (S <sub>i(t)</sub> D <sub>i(t)</sub>).
4. The foregoing states that R <sub>i(t)</sub>  = α \* y <sub>i(t)</sub> + (1 - α) \* S <sub>i(t)</sub>.
5. If the modulus of t divided by T <sub>FL</sub> is 0:
1. Local training should be done for M <sub>i</sub> on D <sub>i</sub>.
2. The new compute model output is θ <sub>i</sub> <sup>new − θi</sup>.

Send the measurement from the instrument, denoted as Δθ <sub>i</sub>, to the cloud aggregator.

**As part of the cloud aggregator**:

If the time in milliseconds (t) is a multiple of 10:

1. Aggregate: θ\* = A({Δθ <sub>i</sub> })
2. Return the updated angles θ\* to all the nodes.

**Synchronize simulations**:

If the current time (t) is divisible by the sync interval (T <sub>sync</sub>):

1. Have each node send its delta state δS <sub>i</sub> over the network.
2. Update: S <sub>i(t+1)</sub>  = S <sub>i(t)</sub>  + β \* (average supply - S <sub>i(t)</sub>).

Update time: t = t + 1.

**Key Features**:

Sets known intervals for T <sub>sync</sub>, the synchronization in the simulation space, and for T <sub>FL</sub>, the update rate on the FL side.

- Hybrid method: Uses both information from AI and physical principles found in FMU to decide on actions. Ensuring distributed.
- Simulation accuracy: This ensures that a multi-node co-simulation runs precisely by periodically synchronizing the virtual states.

## Results and discussion

A digital twin-empowered smart factory was examined using real-world scenarios in two industrial settings: (i) maintaining rotating equipment and (ii) managing output in a multi-line assembly room. The experiments were conducted using Jetson Nano, Raspberry Pi, and a central cloud server to simulate live factory settings. Edge computing was utilized in predictive maintenance by deploying equipment near rotary tools to collect and analyze vibration and temperature data in real time. By relying on AI models embedded in the system, it was possible to predict almost all imminent failures and respond in under 20 milliseconds on average. With the addition of physics-based FMUs, system operators can gain a deeper understanding of the results of co-simulations and identify any anomalies that may arise [^18] [^19]. To check the throughput in production, edge AI analyzed where the bottlenecks would occur, and the FMUs modeled the assembly line operation at different production rates. Models were trained on different assembly lines using Federated Learning, so the data did not need to be transferred, and model convergence was achieved after approximately 25 iterations between the teams. As a result, throughput increased by 18%, and process delays decreased by 22% compared to a standard cloud-only network.

Furthermore, privacy is maintained, and convergence is stable in the proposed Federated Learning framework. At training time, every edge node performs local updates based on its own dataset and sends encrypted gradients only to the central server. To prevent data leakage, the collaborative model’s accuracy is ensured by using differential privacy and homomorphic encryption to safeguard the gradient exchanges. Convergence behavior was examined over several rounds, and the ability of the global models to improve in terms of accuracy and stability was stable beyond 25 rounds of communication. The model loss stabilized with minimal variance, demonstrating the robustness of the FL process and indicating the effectiveness of communication, which provided excellent privacy assurances.

To assess the system’s effectiveness, the experts evaluated four key factors: response time, additional cloud communication required, model accuracy, and throughput improvement [^20]. The outcome showed a 35% decrease in latency, a 28% decrease in cloud usage, and a 20% increase in error detection. A study comparing the new architecture with centralized and older digital twins revealed that the innovative format is more scalable, protects privacy, and responds more quickly to changes. Whenever there was an elevated number of devices and complexity in the simulation, the system brought synchronized and accurate results, proving that the FECO algorithm was effective. Four performance metrics are considered: latency, bandwidth usage, fault detection accuracy, and production throughput gain.

The experiment was used to simulate a factory environment, utilizing two simulated rotating equipment setups in a maintenance environment and a multi-line assembly environment. The emulation of edge, device, and cloud layers was executed using a Jetson Nano, a Raspberry Pi, and a central cloud server with the same hardware and network setups. Industrial Wi-Fi parameters were adjusted to create conditions that closely simulated reality in the factory, allowing for variations in network latency and bandwidth. The Cloud-Centric and Edge AI Only systems were run on the same hardware and network conditions, ensuring all configurations are consistent. Such a configuration provides a consistent basis for comparing all architectures in terms of latency, bandwidth consumption, and fault detection accuracy.

### Latency comparison

The Table [1](https://www.nature.com/articles/s41598-025-28466-9#Tab1) shows the response latency (in milliseconds) for three types of architectures commonly found in smart factories: Cloud-Centric, Edge AI Only, and the proposed architecture (Edge AI + Federated Learning + Co-Simulation) [^21]. The cloud-centric approach has the longest delays due to network issues and the requirement to run everything in a single location. A benefit of Edge AI Only is that it reduces time delays by handling data directly on the device. By combining edge computing on IoT devices and nearby co-simulation, the proposed method achieves the best latency (42 ms) with minimal reliance on the cloud. Examining the Fig. [5](https://www.nature.com/articles/s41598-025-28466-9#Fig5), latency decreases significantly as intelligence is moved to the edge, validating the method’s usefulness for industrial applications that require short delays.

**Table 1 Latency Comparison.**

**Fig. 5**

![Fig. 5](https://media.springernature.com/lw685/springer-static/image/art%3A10.1038%2Fs41598-025-28466-9/MediaObjects/41598_2025_28466_Fig5_HTML.png?as=webp)

[Full size image](https://www.nature.com/articles/s41598-025-28466-9/figures/5)

Latency comparison.

### Bandwidth

The Table [2](https://www.nature.com/articles/s41598-025-28466-9#Tab2) shows the bandwidth used (in MB/hr) by each type of architectural approach. Because it moves a large amount of data over the internet, the Cloud-Centric approach requires the most bandwidth. Edge AI only saves bandwidth, but still shares a portion of the raw data [^22]. The Proposed system takes up the least amount of bandwidth (1.7 MB/hr) by performing training on each device and only sharing updates with a central server. Figure [6](https://www.nature.com/articles/s41598-025-28466-9#Fig6) shows that bandwidth usage is significantly reduced, which is crucial for scaling, particularly in environments where bandwidth or privacy concerns are a priority.

**Table 2 Bandwidth usage comparison.**

**Fig. 6**

![Fig. 6](https://media.springernature.com/lw685/springer-static/image/art%3A10.1038%2Fs41598-025-28466-9/MediaObjects/41598_2025_28466_Fig6_HTML.png?as=webp)

[Full size image](https://www.nature.com/articles/s41598-025-28466-9/figures/6)

Bandwidth usage comparison.

### Quality and precision of fault detection

Percentage indicators in Table [3](https://www.nature.com/articles/s41598-025-28466-9#Tab3) indicate the model’s effectiveness in identifying faults in industrial machines. Cloud-centric models perform well, although they struggle to respond promptly and provide the necessary details. Edge AI performs better because it leverages data learned from its surroundings [^23]. The highest accuracy (94.2%) is possible in this approach because it uses both physical simulations and AI across multiple edge nodes. Figure [7](https://www.nature.com/articles/s41598-025-28466-9#Fig7) Incorporating simulation models and hybrid learning into models enhances accuracy, which is crucial for mission-critical applications such as predictive maintenance.

**Table 3 Fault detection accuracy.**

**Fig. 7**

![Fig. 7](https://media.springernature.com/lw685/springer-static/image/art%3A10.1038%2Fs41598-025-28466-9/MediaObjects/41598_2025_28466_Fig7_HTML.png?as=webp)

[Full size image](https://www.nature.com/articles/s41598-025-28466-9/figures/7)

Fault Detection Accuracy.

### Throughput gain

Table [4](https://www.nature.com/articles/s41598-025-28466-9#Tab4); Fig. [8](https://www.nature.com/articles/s41598-025-28466-9#Fig8) quantify the improvement in production throughput across different architectures.

- The Baseline system (now corrected to 5%) represents a static factory without real-time optimization.
- Edge AI offers moderate improvement (7.5%) through local decision-making.
- The Proposed framework achieves the highest throughput gain (13.2%) by enabling dynamic optimization via co-simulation and collective learning.

**Table 4 Throughput Gain.**

**Fig. 8**

![Fig. 8](https://media.springernature.com/lw685/springer-static/image/art%3A10.1038%2Fs41598-025-28466-9/MediaObjects/41598_2025_28466_Fig8_HTML.png?as=webp)

[Full size image](https://www.nature.com/articles/s41598-025-28466-9/figures/8)

Throughput gain.

### Comparative analysis

This section compares the proposed Digital Twin-Driven Smart Factory framework with two baseline approaches, Cloud-Centric Architecture and Edge AI-only Systems, across four major dimensions: latency, Bandwidth Usage, Fault Detection Accuracy, and Throughput Gain. Figure [9](https://www.nature.com/articles/s41598-025-28466-9#Fig9) summarizes these comparative metrics, aligning visually with the data reported in Table [5](https://www.nature.com/articles/s41598-025-28466-9#Tab5).

**Table 5 Comparative analysis.**

The data highlights the change in production we see by using different hardware architectures. Baseline (with correction) refers to a factory that is not optimized in real time. Edge AI achieves a notable improvement (7.5%) thanks to its local computing. It achieves the largest throughput improvement (13.2%) by utilizing joint simulation and learning across different models [^24] [^25]. The following figure clearly shows that, compared to others, the new system performs significantly better, supporting its positive impact on operations. It compares the suggested model for a Digital Twin-Driven Smart Factory with Cloud-Centric Architecture and Edge AI-only Systems by examining four key aspects—response time, bandwidth expenditure, accuracy in fault detection, and performance gain—and draws specific conclusions.

### What the comparison highlighted

- Latency and responsiveness are improved since inference and simulations are handled close to where they are needed.
- Going federated reduces the amount of data transmission compared to statistical averages.
	- Better accuracy and intelligence: Applying co-simulation and distributed training enhances accuracy and the system’s intelligence.
		- Operational efficiency: A 13.2% increase demonstrates that real-time adaptive decision-making enhances operational efficiency.
- Scaling to many devices and users is possible with Federated Learning while maintaining privacy.

**Fig. 9**

![Fig. 9](https://media.springernature.com/lw685/springer-static/image/art%3A10.1038%2Fs41598-025-28466-9/MediaObjects/41598_2025_28466_Fig9_HTML.png?as=webp)

[Full size image](https://www.nature.com/articles/s41598-025-28466-9/figures/9)

Performance metric.

The framework that connects Digital Twins, Edge AI, and Federated Learning outperforms single applications of Centralized Digital Twin, Edge AI only, and Federated Learning only in every measured aspect. This approach can decrease latency by 35% and lower cloud communication overhead by 28%, which is much more impressive than the small gains from the baseline solutions. Additionally, it boosts the model’s accuracy by 20% and increases production throughput by 18%, demonstrating its ability to provide real-time, scalable, and secure optimization. They show that the selected hybrid model is the leading option for responsive, intelligent, and efficient smart manufacturing. To consider the relevance of the framework to Industry 5.0 objectives, sustainability and resilience indicators were integrated into the performance assessment. The metric of sustainability used was energy savings (a 12.8% decrease in per-unit energy consumption) and the efficient use of resources, facilitated by federated coordination. System recovery time during simulated node failures and production disturbances was used to assess resilience, with recovery times being 17 times faster than those of cloud-centric ones. These metrics are used to complement classic metrics, such as latency and throughput, and demonstrate that the suggested architecture promotes high performance while also being environmentally friendly and resilient to manufacturing.

### Generative AI and AIGC-driven smart manufacturing

In addition to predictive and reactive intelligence, the future of smart manufacturing aligns with Industry 5.0, where human creativity is combined with the generative capacity of AI. Including Generative AI (GenAI) and AI-Generated Content (AIGC) into the Digital Twin system enables the creation of an ecosystem for federated learning, allowing for the generation of optimal design configurations, process parameters, and workflow structures. Co-simulation structures can also be made more flexible and creative by utilizing diffusion-based generative models, which can generate and simulate thousands of design alternatives based on various physical and operational constraints. Articles like “Diffusion model-driven smart design and manufacturing” and “AIGC-empowered smart manufacturing” prove that the combination of diffusion models and digital twins enables the development of generative co-simulation, in which various system setups can be dynamically updated using AI-enhanced design feedback. This is a direct supplement of the suggested FECO architecture, which offers a flexible framework to future self-optimizing and creative factories.

## Conclusion

This research work introduces a new approach to integrating Digital Twin technology with Edge AI and Federated Learning to address significant challenges in smart manufacturing, including slow response times, data privacy concerns, and lack of efficient scalability Because simulation and learning are handled at the edge, the proposed system can run physics-based co-simulation in real-time and help each factory unit react quickly and work more efficiently. The framework consistently outperforms cloud-centric and edge-only baselines, as detailed in Sect. 4.5. These results support how the framework can produce safe, flexible, and growing creative solutions for today’s digital twin-driven smart factories. FMI integration, along with lightweight edge inference and learning across multiple devices, supports a system that can easily adapt to future changes. By doing this, it advances efforts to establish Industry 4.0 by improving manufacturing environments that are adaptable, automated, and strong. Through alignment with the broader vision of Industry 5.0, the framework supports sustainable, human-centered, and resilient manufacturing systems —key dimensions of the next industrial paradigm.

### Future improvement

The other area that can be developed is to extend federated learning beyond model training to include product lifecycle management (PLM) and system reconfiguration. With this type of long architecture, FL can be used to synchronize product design models, maintenance history, and operational insights across factories, allowing digital twins to co-evolve in real-time. Researchers, including those in federated learning-powered manufacturing and product life cycle management, as well as digital twin-powered rapid reconfiguration of automated manufacturing systems via an open architecture model, illustrate approaches to scalable, privacy-conscious PLM, which enable the flexible reconfiguration of production systems. These principles will help develop the proposed framework in support of the Industry 5.0 vision, which envisions adaptive, collaborative, and human-centric manufacturing ecosystems.

Future updates may integrate blockchain technology to enhance the security and transparency of records, facilitate autonomous learning, and facilitate the seamless transfer of important models between locations. Additionally, new AR/VR interfaces could facilitate improved interaction with digital twins, and by utilizing energy-aware scheduling, operational activities can become more environmentally friendly. The implementation of interoperability through standards such as DTDL and OPC UA over TSN will ensure that various industrial systems integrate coherently. Future extensions will include generative design pipelines and diffusion-based AIGC models to automatically suggest and confirm system reconfigurations, production plans, and material optimization. This will establish the framework for AI-driven creative production, contributing to the principles of human-AI collaboration that Industry 5.0 will focus on. Moreover, the framework can be further extended by Physics-Informed Machine Learning (PIML) to integrate physical laws (e.g., the conservation of energy or relationships between stress and strain) or thermodynamic constraints into the training process of the neural network. When such principles are combined with data-driven optimization, every edge node can learn models that are consistent with known physics, even in unseen working conditions. This would ensure that data dependency is mitigated, model interpretability is enhanced, and the robustness of Federated Edge-CoSim Optimizer (FECO) to act as a robust decision-maker in real-time is improved. Augmented digital twins of this kind are consistent with the Industry 5.0 vision of human-centered, resilient, and sustainable manufacturing.

Moreover, the suggested system can be applied to the industrial metaverse to make immersive human-machine cooperation and remote control developments. Remote validation and optimization of industrial processes would be made possible by integrating digital twin-based semi-physical commissioning, as demonstrated in the article titled “Digital Twins-Based Remote Semi-Physical Commissioning of Flow-Type Smart Manufacturing Systems.” With the integration of AR/VR interfaces, operators can visualize, interact with, and control the digital twin environment in real-time, thereby enhancing situational awareness and operational efficiency. This orientation is associated with the vision presented in the article ‘Industrial metaverse towards Industry 5.0,’ which builds a cyberphysical-social workspace where humans, AI agents, and digital twins can work in harmony within a connected and experiential manufacturing ecosystem.

## Data availability

The datasets used and/or analyzed during the current study are available from the corresponding author on reasonable request.

## References

## Ethics declarations

### Competing interests

The authors declare no competing interests.

## Additional information

### Publisher’s note

Springer Nature remains neutral with regard to jurisdictional claims in published maps and institutional affiliations.

## Rights and permissions

**Open Access** This article is licensed under a Creative Commons Attribution-NonCommercial-NoDerivatives 4.0 International License, which permits any non-commercial use, sharing, distribution and reproduction in any medium or format, as long as you give appropriate credit to the original author(s) and the source, provide a link to the Creative Commons licence, and indicate if you modified the licensed material. You do not have permission under this licence to share adapted material derived from this article or parts of it. The images or other third party material in this article are included in the article’s Creative Commons licence, unless indicated otherwise in a credit line to the material. If material is not included in the article’s Creative Commons licence and your intended use is not permitted by statutory regulation or exceeds the permitted use, you will need to obtain permission directly from the copyright holder. To view a copy of this licence, visit [http://creativecommons.org/licenses/by-nc-nd/4.0/](http://creativecommons.org/licenses/by-nc-nd/4.0/).

[^1]: Ambedkar, K. & Prakash, G. Evolving dynamic capabilities: exploring implications of industry 4.0 digital technologies on manufacturing. *IEEE Trans. Eng. Manage.* **71**, 5455–5469. [https://doi.org/10.1109/TEM.2024.3361308](https://doi.org/10.1109/TEM.2024.3361308) (2024).

[Article](https://doi.org/10.1109%2FTEM.2024.3361308) [Google Scholar](http://scholar.google.com/scholar_lookup?&title=Evolving%20dynamic%20capabilities%3A%20exploring%20implications%20of%20industry%204.0%20digital%20technologies%20on%20manufacturing&journal=IEEE%20Trans.%20Eng.%20Manage.&doi=10.1109%2FTEM.2024.3361308&volume=71&pages=5455-5469&publication_year=2024&author=Ambedkar%2CK&author=Prakash%2CG)

[^2]: Bae, C., Choi, E. & Lee, S. Technologies, Applications, and challenges of digital twin across industries: A systematic review of the State-of-the-Art literature. *IEEE Access.* **13**, 152843–152869. [https://doi.org/10.1109/ACCESS.2025.3601615](https://doi.org/10.1109/ACCESS.2025.3601615) (2025).

[Article](https://doi.org/10.1109%2FACCESS.2025.3601615) [Google Scholar](http://scholar.google.com/scholar_lookup?&title=Technologies%2C%20Applications%2C%20and%20challenges%20of%20digital%20twin%20across%20industries%3A%20A%20systematic%20review%20of%20the%20State-of-the-Art%20literature&journal=IEEE%20Access.&doi=10.1109%2FACCESS.2025.3601615&volume=13&pages=152843-152869&publication_year=2025&author=Bae%2CC&author=Choi%2CE&author=Lee%2CS)

[^3]: Balta, E. C., Pease, M., Moyne, J., Barton, K. & Tilbury, D. M. Digital Twin-Based Cyber-Attack Detection Framework for Cyber-Physical Manufacturing Systems, *IEEE Transactions on Automation Science and Engineering.* **21** (2) 1695–1712, Apr. (2024). [https://doi.org/10.1109/TASE.2023.3243147](https://doi.org/10.1109/TASE.2023.3243147)

[^4]: Dietz, M., Reichvilser, T. & Pernul, G. A Data-Driven framework for digital twin creation in industrial environments. *IEEE Access.* **12**, 93294–93304. [https://doi.org/10.1109/ACCESS.2024.3423372](https://doi.org/10.1109/ACCESS.2024.3423372) (2024).

[Article](https://doi.org/10.1109%2FACCESS.2024.3423372) [Google Scholar](http://scholar.google.com/scholar_lookup?&title=A%20Data-Driven%20framework%20for%20digital%20twin%20creation%20in%20industrial%20environments&journal=IEEE%20Access.&doi=10.1109%2FACCESS.2024.3423372&volume=12&pages=93294-93304&publication_year=2024&author=Dietz%2CM&author=Reichvilser%2CT&author=Pernul%2CG)

[^5]: Din, I. U., Taj, I., Awan, K. A., Almogren, A. & Altameem, A. Quantum and GAN-Driven Digital Twin Approach for IoT-Based Consumer Electronics Manufacturing. *IEEE Internet of Things Journal* **12** (4), 3734–3741, https://doi.org/10.1109/JIOT.2024.3499737 (2025).

[^6]: Dong Choi, J. et al. AI and digital twin Federation-Based flexible safety control for Human–Robot collaborative work cell. *IEEE Access.* **13**, 124037–124050. [https://doi.org/10.1109/ACCESS.2025.3586121](https://doi.org/10.1109/ACCESS.2025.3586121) (2025).

[Article](https://doi.org/10.1109%2FACCESS.2025.3586121) [Google Scholar](http://scholar.google.com/scholar_lookup?&title=AI%20and%20digital%20twin%20Federation-Based%20flexible%20safety%20control%20for%20Human%E2%80%93Robot%20collaborative%20work%20cell&journal=IEEE%20Access.&doi=10.1109%2FACCESS.2025.3586121&volume=13&pages=124037-124050&publication_year=2025&author=Dong%20Choi%2CJ)

[^7]: Ferko, E., Bucaioni, A. & Behnam, M. Architecting digital twins. *IEEE Access.* **10**, 50335–50350. [https://doi.org/10.1109/ACCESS.2022.3172964](https://doi.org/10.1109/ACCESS.2022.3172964) (2022).

[Article](https://doi.org/10.1109%2FACCESS.2022.3172964) [Google Scholar](http://scholar.google.com/scholar_lookup?&title=Architecting%20digital%20twins&journal=IEEE%20Access.&doi=10.1109%2FACCESS.2022.3172964&volume=10&pages=50335-50350&publication_year=2022&author=Ferko%2CE&author=Bucaioni%2CA&author=Behnam%2CM)

[^8]: Gadekallu, T. R. et al. XAI for industry 5.0—Concepts, Opportunities, Challenges, and future directions. *IEEE Open. J. Commun. Soc.* **6**, 2706–2729. [https://doi.org/10.1109/OJCOMS.2024.3473891](https://doi.org/10.1109/OJCOMS.2024.3473891) (2025).

[Article](https://doi.org/10.1109%2FOJCOMS.2024.3473891) [Google Scholar](http://scholar.google.com/scholar_lookup?&title=XAI%20for%20industry%205.0%E2%80%94Concepts%2C%20Opportunities%2C%20Challenges%2C%20and%20future%20directions&journal=IEEE%20Open.%20J.%20Commun.%20Soc.&doi=10.1109%2FOJCOMS.2024.3473891&volume=6&pages=2706-2729&publication_year=2025&author=Gadekallu%2CTR)

[^9]: Hu, Y. et al. Industrial internet of things intelligence empowering smart manufacturing: A literature review. *IEEE Internet Things J.* **11** (11), 19143–19167. [https://doi.org/10.1109/JIOT.2024.3367692](https://doi.org/10.1109/JIOT.2024.3367692) (June 2024).

[^10]: Huang, J. et al. Deep Reinforcement Learning-Based Dynamic Reconfiguration Planning for Digital Twin-Driven Smart Manufacturing Systems With Reconfigurable Machine Tools. *IEEE Transactions on Industrial Informatics* **20** (11), 13135–13146 https://doi.org/10.1109/TII.2024.3431095 (2024).

[^11]: Kahveci, E., Akgul, A. K., Daim, T. & Meissner, D. The critical mediator role of process management for effective industry 4.0 implementation in SMEs. *IEEE Trans. Eng. Manage.* **72**, 4–17. [https://doi.org/10.1109/TEM.2024.3494535](https://doi.org/10.1109/TEM.2024.3494535) (2025).

[Article](https://doi.org/10.1109%2FTEM.2024.3494535) [Google Scholar](http://scholar.google.com/scholar_lookup?&title=The%20critical%20mediator%20role%20of%20process%20management%20for%20effective%20industry%204.0%20implementation%20in%20SMEs&journal=IEEE%20Trans.%20Eng.%20Manage.&doi=10.1109%2FTEM.2024.3494535&volume=72&pages=4-17&publication_year=2025&author=Kahveci%2CE&author=Akgul%2CAK&author=Daim%2CT&author=Meissner%2CD)

[^12]: Kaur, H. & Bhatia, M. Scientometric Analysis of Digital Twin in Industry 4.0. *IEEE Internet Things J.* **12** (2), 1200–1221. [https://doi.org/10.1109/JIOT.2024.3459965](https://doi.org/10.1109/JIOT.2024.3459965)

[^13]: Kim, H., Ben-Othman, J. & Kim, D. Harmonized All-Ways Security Surveillance and Disaster Prevention in Smart Eco-Cities. *IEEE Internet of Things Journal* **11** (8), 14206–14215. https://doi.org/10.1109/JIOT.2023.3340169 (2024).

[^14]: Li, L., Aslam, S., Wileman, A. & Perinpanayagam, S. Digital twin in aerospace industry: A gentle introduction. *IEEE Access.* **10**, 9543–9562. [https://doi.org/10.1109/ACCESS.2021.3136458](https://doi.org/10.1109/ACCESS.2021.3136458) (2022).

[Article](https://doi.org/10.1109%2FACCESS.2021.3136458) [Google Scholar](http://scholar.google.com/scholar_lookup?&title=Digital%20twin%20in%20aerospace%20industry%3A%20A%20gentle%20introduction&journal=IEEE%20Access.&doi=10.1109%2FACCESS.2021.3136458&volume=10&pages=9543-9562&publication_year=2022&author=Li%2CL&author=Aslam%2CS&author=Wileman%2CA&author=Perinpanayagam%2CS)

[^15]: Liu, X. et al. Design of an optimal scheduling control system for smart manufacturing processes in tobacco industry. *IEEE Access.* **11**, 33027–33036. [https://doi.org/10.1109/ACCESS.2023.3261883](https://doi.org/10.1109/ACCESS.2023.3261883) (2023).

[Article](https://doi.org/10.1109%2FACCESS.2023.3261883) [Google Scholar](http://scholar.google.com/scholar_lookup?&title=Design%20of%20an%20optimal%20scheduling%20control%20system%20for%20smart%20manufacturing%20processes%20in%20tobacco%20industry&journal=IEEE%20Access.&doi=10.1109%2FACCESS.2023.3261883&volume=11&pages=33027-33036&publication_year=2023&author=Liu%2CX)

[^16]: Maier, M., Ebrahimzadeh, A., Beniiche, A. & Rostami, S. The Art of 6G (TAO 6G): how to wire society 5.0 \[Invited\]. *J. Opt. Commun. Netw.* **14** (2), A101–A112. [https://doi.org/10.1364/JOCN.438522](https://doi.org/10.1364/JOCN.438522) (Feb. 2022).

[^17]: Nwoke, J., Milanesi, M., Viola, J., Chen, Y. & Visioli, A. A Reduced-Order digital twin FPGA-Based implementation with Self-Awareness capabilities for power electronics applications. *IEEE J. Radio Freq. Identif.* **8**, 493–505. [https://doi.org/10.1109/JRFID.2024.3404563](https://doi.org/10.1109/JRFID.2024.3404563) (2024).

[Article](https://doi.org/10.1109%2FJRFID.2024.3404563) [ADS](http://adsabs.harvard.edu/cgi-bin/nph-data_query?link_type=ABSTRACT&bibcode=2024IJRFI...8..493N) [Google Scholar](http://scholar.google.com/scholar_lookup?&title=A%20Reduced-Order%20digital%20twin%20FPGA-Based%20implementation%20with%20Self-Awareness%20capabilities%20for%20power%20electronics%20applications&journal=IEEE%20J.%20Radio%20Freq.%20Identif.&doi=10.1109%2FJRFID.2024.3404563&volume=8&pages=493-505&publication_year=2024&author=Nwoke%2CJ&author=Milanesi%2CM&author=Viola%2CJ&author=Chen%2CY&author=Visioli%2CA)

[^18]: Ooi, K. B. et al. The metaverse in engineering management: Overview, Opportunities, Challenges, and future research agenda. *IEEE Trans. Eng. Manage.* **71**, 13882–13889. [https://doi.org/10.1109/TEM.2023.3307562](https://doi.org/10.1109/TEM.2023.3307562) (2024).

[Article](https://doi.org/10.1109%2FTEM.2023.3307562) [Google Scholar](http://scholar.google.com/scholar_lookup?&title=The%20metaverse%20in%20engineering%20management%3A%20Overview%2C%20Opportunities%2C%20Challenges%2C%20and%20future%20research%20agenda&journal=IEEE%20Trans.%20Eng.%20Manage.&doi=10.1109%2FTEM.2023.3307562&volume=71&pages=13882-13889&publication_year=2024&author=Ooi%2CKB)

[^19]: Qian, F., Tang, Y. & Yu, X. The future of process industry: A Cyber–Physical–Social system perspective. *IEEE Trans. Cybernetics*. **54**, 3878–3889. [https://doi.org/10.1109/TCYB.2023.3298838](https://doi.org/10.1109/TCYB.2023.3298838) (July 2024).

[^20]: Seok, M. G., Tan, W. J., Cai, W. & Park, D. Digital-Twin Consistency Checking Based on Observed Timed Events With Unobservable Transitions in Smart Manufacturing. *IEEE Transactions on Industrial Informatics* **19** (4), https://doi.org/10.1109/TII.2022.3200598 (2023).

[^21]: Shi, F., Zhou, F., Liu, H., Chen, L. & Ning, H. Survey and tutorial on hybrid Human-Artificial intelligence. *Tsinghua Sci. Technol.* **28** (3), 486–499. [https://doi.org/10.26599/TST.2022.9010022](https://doi.org/10.26599/TST.2022.9010022) (June 2023).

[^22]: Tang, Q. et al. Internet of intelligence: A survey on the enabling Technologies, Applications, and challenges. *IEEE Commun. Surv. Tutorials*. **24** (3), 1394–1434. [https://doi.org/10.1109/COMST.2022.3175453](https://doi.org/10.1109/COMST.2022.3175453) (2022).

[Article](https://doi.org/10.1109%2FCOMST.2022.3175453) [Google Scholar](http://scholar.google.com/scholar_lookup?&title=Internet%20of%20intelligence%3A%20A%20survey%20on%20the%20enabling%20Technologies%2C%20Applications%2C%20and%20challenges&journal=IEEE%20Commun.%20Surv.%20Tutorials&doi=10.1109%2FCOMST.2022.3175453&volume=24&issue=3&pages=1394-1434&publication_year=2022&author=Tang%2CQ)

[^23]: Teng, S. et al. The ParallelWorkforce: A Framework for Synergistic Collaboration in Digital, Robotic, and Biological Workers of Industry 5.0. *IEEE Trans. Comput. Social Syst.* **12** (4), 1627–1636. [https://doi.org/10.1109/TCSS.2024.3512576](https://doi.org/10.1109/TCSS.2024.3512576). (2024).

[^24]: Wen, J., Kang, J., Niyato, D., Zhang, Y. & Mao, S. Sustainable Diffusion-Based incentive mechanism for generative AI-Driven digital twins in industrial Cyber-Physical systems. *IEEE Trans. Industrial Cyber-Physical Syst.* **3**, 139–149. [https://doi.org/10.1109/TICPS.2024.3524483](https://doi.org/10.1109/TICPS.2024.3524483) (2025).

[Article](https://doi.org/10.1109%2FTICPS.2024.3524483) [Google Scholar](http://scholar.google.com/scholar_lookup?&title=Sustainable%20Diffusion-Based%20incentive%20mechanism%20for%20generative%20AI-Driven%20digital%20twins%20in%20industrial%20Cyber-Physical%20systems&journal=IEEE%20Trans.%20Industrial%20Cyber-Physical%20Syst.&doi=10.1109%2FTICPS.2024.3524483&volume=3&pages=139-149&publication_year=2025&author=Wen%2CJ&author=Kang%2CJ&author=Niyato%2CD&author=Zhang%2CY&author=Mao%2CS)

[^25]: Padmavathi, V., Saminathan, R. A federated edge intelligence framework with trust based access control for secure and privacy preserving IoT systems. *Sci Rep* **15**, 35832. [https://doi.org/10.1038/s41598-025-19712-1](https://doi.org/10.1038/s41598-025-19712-1) (2025).