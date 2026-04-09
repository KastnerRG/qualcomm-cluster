# Qualcomm Cluster v0

A 14-node Kubernetes cluster built from Rubik Pi 3 single-board computers, housed in a donated rack and managed via microk8s.

![Qualcomm Cluster v0](cluster.png)

---

## Hardware

| Component | Model / Details | Qty |
|---|---|---|
| Compute nodes | Rubik Pi 3 | 14 |
| Switch | MikroTik CRS328-24P-4S+RM | 1 |
| Power hubs | Acroname USBHub3c (USB-C PD Hub) | 2 |
| Router | pfSense on a desktop PC | 1 |
| Rack | Donated custom rack (~2U, 19" × 24" × 3.5") | 1 |

**Switch product page:** https://mikrotik.com/product/crs328_24p_4s_rm  
**Power hub product page:** https://acroname.com/store/programmable-industrial-power-delivery-hub-s99-usbhub-3c-pro

### Notes on hardware choices

- The **Acroname hubs** are used solely for USB-C power delivery to the Rubik Pi boards. The monitoring and programmability features of these hubs are not currently used.
- The **MikroTik switch** is configured in **switch mode only** — its router features are disabled. All routing is handled by pfSense.
- **pfSense** runs on a separate desktop machine and serves as the NAT router for the cluster network.

---

## Network Architecture

```
Internet
   |
pfSense (desktop)  ← port forwards inbound traffic to nginx
   |
nginx reverse proxy (x86 machine)  ← routes /gradescope → autograder service
   |
MikroTik CRS328 (switch mode)
   |-- Node 01 (192.168.1.2)
   |-- Node 02 (192.168.1.3)
   ...
   |-- Node 14 (192.168.1.15)
```

All 14 nodes are on a private subnet behind pfSense. pfSense provides NAT for outbound internet access and port-forwards inbound traffic to the nginx reverse proxy.

### Reverse Proxy

An nginx instance running on a separate x86 machine handles inbound requests. pfSense is configured to port-forward external traffic to this machine. nginx proxies requests at the `/gradescope` path to the autograder service running in the cluster.

Example nginx location block:

```nginx
location /gradescope/ {
    proxy_pass http://<autograder-service-ip>:5000/;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
}
```

This allows Gradescope (or any external client) to reach the autograder at `http://<hostname>/gradescope/submit`, `http://<hostname>/gradescope/status/<id>`, etc.

---

## Step 1: Flash Ubuntu 24.04 onto each Rubik Pi 3

Follow the official Thundercomm documentation for flashing:

> https://www.thundercomm.com/rubik-pi-3/en/docs/about-rubikpi/

Use the **Qualcomm Launcher** tool as described in those docs to flash Ubuntu 24.04 to each board. Repeat this process for all 14 nodes.

---

## Step 2: Physical Setup

1. Mount the Rubik Pi boards in the rack.
2. Connect each board to the Acroname USBHub3c for power via USB-C.
3. Run an ethernet cable from each board to a port on the MikroTik switch.
4. Connect the MikroTik switch uplink to the pfSense machine.

> The rack provides approximately 2U of vertical space. The MikroTik switch occupies 1U.

---

## Step 3: Configure the MikroTik Switch

The MikroTik CRS328 is used in **switch mode** with its **default configuration** — no routing, VLANs, or custom port settings are applied. All traffic is bridged to the pfSense router.

No additional switch configuration is required beyond the factory defaults.

---

## Step 4: Configure pfSense

1. Install pfSense on a desktop machine.
2. Assign one NIC as the WAN interface (upstream internet) and one as the LAN interface (connected to the MikroTik switch).
3. Configure the LAN interface with a static IP that will serve as the default gateway for all cluster nodes.
4. Enable NAT on the WAN interface so nodes can reach the internet.

pfSense handles all routing. The MikroTik switch is purely layer 2.

---

## Step 5: Assign Static IPs to Nodes via DHCP Reservations

Static IPs are managed in **pfSense via DHCP static mappings** — the nodes themselves use DHCP. pfSense maps each node's MAC address to a fixed IP, ensuring they always receive the same address.

Node IPs are assigned as follows:

| Node | IP Address |
|---|---|
| Node 1 | 192.168.1.2 |
| Node 2 | 192.168.1.3 |
| ... | ... |
| Node 14 | 192.168.1.15 |

To configure a static mapping in pfSense:

1. Go to **Services → DHCP Server → LAN**
2. Scroll to **DHCP Static Mappings** and click **Add**
3. Enter the node's MAC address, assign the desired IP, and optionally set a hostname
4. Save and apply

Repeat for each of the 14 nodes. No netplan changes are needed on the nodes themselves — leave them configured for DHCP.

---

## Step 6: Install microk8s on All Nodes

On **each node**, install microk8s via snap:

```bash
sudo snap install microk8s --classic
sudo usermod -aG microk8s $USER
newgrp microk8s
```

---

## Step 7: Bootstrap the Control Plane

Choose one node to be the **control plane**. On that node:

```bash
microk8s status --wait-ready
```

Confirm microk8s is running before proceeding.

---

## Step 8: Join Worker Nodes to the Cluster

On the **control plane node**, generate a join token for each worker:

```bash
microk8s add-node
```

This outputs a `microk8s join` command. Copy it and run it on the worker node:

```bash
# On the worker node:
microk8s join <control-plane-ip>:<port>/<token>
```

Repeat for each of the remaining 13 nodes.

---

## Step 9: Verify the Cluster

On the control plane node:

```bash
microk8s kubectl get nodes
```

All 14 nodes should appear with a `Ready` status.

---

## Autograder Service

Student submissions are run via a Go HTTP service from the [junkyard-autograder](https://github.com/junkyard-computing/junkyard-autograder) project. The service accepts a student's code as a zip archive, creates a Kubernetes Job to run it, and returns the results.

The server listens on port **5000**.

### Endpoints

| Method | Path | Description |
|---|---|---|
| `POST` | `/submit` | Submit a student assignment for grading |
| `GET` | `/status/<job-id>` | Poll for job status and results |
| `GET` | `/logs/<job-id>` | Fetch raw logs for a completed job |
| `GET` | `/` | Health check |

All endpoints require a `Authorization: Bearer <token>` header.

**`POST /submit`** accepts a multipart form with:
- `image` — assignment name (the last character is parsed as the PA number, e.g. `pa2` → runs `/autograder/source/PA2/cluster/run_autograder`)
- `script` — a `.zip` archive of the student's submission

The server responds immediately with `202 Accepted` and a `job_id`. The client should poll `/status/<job-id>` until the status is `succeeded` or `failed`.

**`GET /status/<job-id>`** returns a JSON payload:

```json
{
  "status": "succeeded",
  "results": "{ ...grading JSON... }",
  "score": 0,
  "latency": "1m23s",
  "error": ""
}
```

Grading results are parsed from the job logs between `---JSON_START---` and `---JSON_END---` markers. If those markers are absent (e.g. the student's code crashed), a fallback zero-score result is returned.

### Environment Variables

| Variable | Required | Description |
|---|---|---|
| `RUNNER_IMAGE` | Yes | Docker image used to run student code |
| `REGISTRY_SECRET_NAME` | Yes | Kubernetes image pull secret name |
| `GPU_DEVICE_PATH` | Yes | Host path to the GPU device (e.g. `/dev/mali0`) |
| `AUTOGRADER_TIMEOUT` | No | Job timeout in seconds (default: `120`) |
| `KUBECONFIG` | No | Path to kubeconfig file; omit to use in-cluster config |

### How Jobs Are Run

For each submission the service:

1. Creates a Kubernetes **ConfigMap** containing the student's zip archive
2. Creates a Kubernetes **Job** that mounts the ConfigMap and runs the autograder script
3. Monitors the job in a background goroutine, polling every 2 seconds
4. Fetches pod logs on completion and parses the grading JSON
5. Deletes the Job and ConfigMap after completion

Each job pod:
- Uses **pod anti-affinity** (`work: assignmentExecution` label, `topologyKey: kubernetes.io/hostname`) to enforce one pod per node
- Mounts the GPU device (`GPU_DEVICE_PATH`), `/dev/dri`, and a HuggingFace model cache from the host
- Runs as **privileged** with `hostPID: true` for GPU access
- Never restarts (`RestartPolicy: Never`)
- Is automatically deleted 120 seconds after completion

The in-memory job store is cleaned up after 1 hour. The service does not persist state across restarts.

> **Configuration note:** The specific deployment configuration (Kubernetes manifests, secret values, registry details) is not currently documented here.

---

## GPU Access

The Rubik Pi 3 includes a Qualcomm Adreno GPU. We used it primarily for **OpenCL** access. To access the GPU from within a container, use the pre-built Docker image maintained here:

> https://github.com/KastnerRG/qualcomm-docker-image

Follow the instructions in that repo to build and run the image on a node. The container provides the necessary drivers and runtime environment for OpenCL workloads on the Rubik Pi.

---

## Known Issues and Lessons Learned

### One pod per node

When running student workloads, we found it best to schedule **one pod per node**. This gives each student full access to the board's resources (CPU, GPU, memory) without contention from other pods.

Enforce this using pod anti-affinity in the pod spec:

```yaml
affinity:
  podAntiAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchLabels:
            app: <your-app-label>
        topologyKey: kubernetes.io/hostname
```

This tells the scheduler to place no two pods with the same label on the same node.

---

### Control plane taint

The control plane node has its **taint preserved** — workloads are not scheduled on the control plane by default. This is standard practice, but with only 14 nodes, it reduces scheduling capacity to 13 workers.

> **Reliability issue:** We initially removed the taint so workloads could be scheduled on the control plane node. This caused reliability problems — workload pressure on the control plane interfered with cluster management. Restoring the taint resolved the issue. Do not remove the taint on the control plane node.

---

## Next Steps

See [`../v1/proposal.md`](../v1/proposal.md) for the v1 proposal, which covers scaling this cluster to 100–200 nodes using a tray-based rack design and centralized power distribution.
