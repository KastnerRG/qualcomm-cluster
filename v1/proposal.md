Scaling the Qualcomm Cluster: Power, Hardware, and Rack Design
Why This Matters
What does it take to turn a pile of embedded devices into a real compute cluster?
Software is only part of the story. Large-scale systems depend on power delivery, physical design, and hardware reliability. If we want junkyard clusters to scale beyond prototypes, we need to answer practical questions: how do we power many devices safely, how do we package them cleanly, and what design choices make the system reproducible?
This project focuses on building the next-generation hardware platform for a Qualcomm-based cluster—with an emphasis on scalable tray-level design and high-quality documentation for external partners like Qualcomm.
Project Overview
In this project, you will design and prototype the next hardware iteration of the Qualcomm cluster using Rubik Pi devices. Qualcomm has confirmed that these boards can be powered through the 5V GPIO pins, enabling centralized power delivery.
The system will be designed around a tray-based architecture, where each tray supports approximately 24–36 nodes. These trays are intended to serve as the repeatable building blocks of a larger cluster with an eventual scale of roughly 100–200 total nodes. Your goal is to design a tray and supporting hardware approach that is:
electrically stable
physically organized
easy to replicate
suitable for scaling across multiple trays in a rack
You will:
design a power distribution system per tray
evaluate power strategies such as DC distribution versus a custom PoE-based design
design a rack-compatible tray
validate stability across many nodes powered simultaneously
document the system clearly enough that future teams and external partners can build on it
A key outcome is clear, professional documentation, as Qualcomm is interested in the results of this work.
What You’ll Do
Design and prototype a power distribution PCB for a tray of 24–36 Rubik Pi nodes
Test powering boards through the 5V GPIO pins at scale
Evaluate two power strategies:
centralized DC-to-DC power distribution per tray
custom PoE HAT / adapter approach
Analyze the implications of PoE given that:
Rubik Pis do not support PoE natively
a custom HAT or adapter is required
power and data will likely use separate paths
Design and prototype a tray system that:
holds 24–36 nodes
supports airflow and cooling
enables clean cable management
can be replicated across enough trays to support a 100–200 node cluster
Measure and evaluate:
voltage drop across the tray
current draw under full load
boot-time power spikes from many nodes starting simultaneously
thermal behavior within the tray
likely failure modes as the system scales to multiple trays
Produce clear technical documentation of the system, tradeoffs, and recommendations
How You Might Approach It
This project combines hardware design, systems thinking, and engineering communication.
Tray-level power design
 Design a PCB or bus system that distributes stable 5V power to 24–36 nodes. Consider total current capacity, voltage drop, connector reliability, and protection circuitry.
Scaling beyond one tray
 Even if the prototype only covers one tray, the design should explicitly consider how it would behave when scaled to 4–6 trays, enough to support a 100–200 node cluster. That means thinking about upstream power distribution, rack layout, heat, serviceability, and wiring complexity.
Tradeoff study: DC vs PoE
 Compare:
a simpler DC distribution per tray
a more complex PoE HAT-based system
Because PoE is not built into the board, your team should explicitly evaluate whether its deployment advantages justify the extra engineering effort.
Scale-aware validation
 Test not just one node, but full-tray scenarios such as simultaneous boot, sustained compute load, and partial failures.
Mechanical design (tray + rack)
 Design a tray that can be inserted into a rack, supports airflow, and can be replicated across multiple trays to form the full cluster.
Documentation (core deliverable)
 Produce documentation that clearly explains:
tray architecture and scaling assumptions
electrical design and operating limits
mechanical design decisions
experimental results and failure cases
recommendations for scaling from one tray to a 100–200 node system
What Success Looks Like
Minimum Viable Product
A working prototype power system for a 24–36 node tray
Demonstration of stable GPIO-based power delivery at tray scale
A basic tray design with mounting and cable organization
Initial documentation describing the design and experimental results
A scaling plan explaining how the tray design could be replicated into a 100–200 node cluster
Stretch Goals
A custom PoE HAT or adapter prototype
A clear recommendation between DC and PoE approaches
Improved PCB design with protections and monitoring
Multi-tray rack design considerations
Power and thermal modeling for a 100–200 node deployment
Publication-quality documentation suitable for Qualcomm and future teams
Why This Project is Exciting
You will design something that operates at real system scale
You will work at the intersection of EE, embedded systems, and distributed computing
You will build infrastructure that is meant to be replicated into a much larger cluster
You will produce work that is useful beyond the classroom
How This Connects to Ongoing Research
This project supports a broader effort to build scalable compute platforms from embedded and repurposed hardware. While other projects focus on orchestration and software, this project focuses on the physical scaling unit of the Qualcomm cluster.
Your work will help answer a key question: what is the right 24–36 node tray design to serve as the building block for a reliable 100–200 node Qualcomm cluster?

