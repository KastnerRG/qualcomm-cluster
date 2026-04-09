# Qualcomm Cluster v1: Scaling to 100–200 Nodes

## Overview

This document describes the next hardware iteration of the Qualcomm cluster. The goal is to scale from the current 14-node prototype to a 100–200 node system built around a repeatable tray-based architecture.

Qualcomm has confirmed that Rubik Pi 3 boards can be powered through the 5V GPIO pins, which enables centralized power delivery per tray. This project designs around that capability.

---

## Architecture

The system is organized into **trays of 24–36 nodes**. Each tray is the repeatable unit of the cluster. Multiple trays mount into a standard rack to form the full system.

Design requirements for each tray:

- Electrically stable power delivery to all nodes
- Clean physical organization with managed cabling
- Adequate airflow for sustained operation
- Rack-compatible form factor
- Straightforward to replicate across 4–6 trays

---

## Power Distribution

Two power strategies are under evaluation:

### Option A: Centralized DC Distribution

A custom PCB or bus distributes regulated 5V power directly to each node via the GPIO pins. This is the simpler approach and avoids per-node power adapters.

Design considerations:
- Total current capacity across 24–36 nodes
- Voltage drop along the distribution bus
- Connector reliability under sustained load
- Protection circuitry (fusing, overcurrent protection)

### Option B: PoE-Based Distribution

Power and data share the same ethernet cabling, with a custom HAT or adapter converting PoE power for each node.

**Important constraint:** Rubik Pi 3 boards do not support PoE natively. A custom HAT or adapter is required, and power and data will likely still run on separate paths internally.

The core tradeoff: PoE simplifies cabling but adds engineering complexity through the custom adapter requirement. A clear recommendation between the two approaches is a target outcome of this work.

---

## Validation Plan

Testing should cover full-tray scenarios, not just individual nodes:

- Voltage drop across the tray at full load
- Current draw under sustained compute workload
- Boot-time power spikes from simultaneous node startup
- Thermal behavior within the tray enclosure
- Projected failure modes when scaling to 4–6 trays

---

## Mechanical Design

The tray must:

- Hold 24–36 Rubik Pi nodes with secure mounting
- Support airflow through the node array
- Enable clean cable routing
- Insert and remove from a standard rack
- Be reproducible as the scaling unit of the cluster

---

## Deliverables

### Minimum Viable

- Working power distribution prototype for a 24–36 node tray
- Demonstrated stable GPIO-based power delivery at tray scale
- Basic tray with node mounting and cable organization
- Technical documentation covering design decisions and experimental results
- Scaling plan for replicating the tray design to 100–200 nodes

### Stretch Goals

- Custom PoE HAT or adapter prototype
- Clear recommendation between DC and PoE approaches with supporting data
- PCB design with protection and monitoring circuitry
- Multi-tray rack design
- Power and thermal model for a 100–200 node deployment
- Publication-quality documentation suitable for external distribution

---

## Documentation

Clear documentation is a primary deliverable. It should cover:

- Tray architecture and scaling assumptions
- Electrical design and operating limits
- Mechanical design decisions and tradeoffs
- Experimental results including failure cases
- Recommendations for scaling from one tray to a full 100–200 node cluster

Qualcomm is an interested external audience for this work.

---

## Relation to v0

The v0 cluster established the baseline: 14 Rubik Pi nodes, microk8s, and a working autograder pipeline. See [`../v0/README.md`](../v0/README.md) for the full v0 setup documentation.

v1 addresses the hardware scaling question: what tray design should serve as the repeatable building block for a production-scale Qualcomm cluster?
