# Enabling and Troubleshooting Flow Offload on Cisco Secure Firewall 3105 (ASA 9.23.1 / FXOS 2.17)

The Cisco Secure Firewall 3105 includes a hardware **Flow Offload** engine (in the FPGA/NIC) that should accelerate traffic flows up to multi-gigabit rates when properly enabled. On ASA 9.23.1 with FXOS 2.17, flow offload is **supposed** to be on by default for the 3100 series.

If you're seeing only 5â€“15 Mbps throughput on a 10 Gbps link, it indicates the firewall is *not* offloading traffic to hardware (all packets are going through the slow software path). Below are clear troubleshooting steps.

---

## 1. Verify Hardware Flow Offload Status and Initialization

**Show Flow Offload Status:**
```
show flow-offload info detail
```
Look specifically for:
- **"Current running state: Enabled"**
- **"Offload App: Running"**

**Check Offload Counters:**
```
show flow-offload statistics
show flow-offload flow
```
Counters increasing indicate offload working.

**ASA Syslog Messages:**
- `%ASA-6-805001`: Successful offload
- `%ASA-6-805003`: Offload failed

**CPU and Throughput Checks:**
```
show cpu usage
show interface
```
High CPU and low throughput indicate offload isn't functioning.

---

## 2. Confirm ASA/FXOS Version Compatibility (3105)

- **ASA 9.23.1 and FXOS 2.17 are officially compatible**.
- Confirm FXOS build:
```
show version
```
- Check Cisco for latest patches.

**Optional Downgrade Test:**  
ASA 9.22(1.1) + FXOS 2.16 is a stable alternative.  
Obtain from **Cisco Firepower 4100/9300** download pages or Cisco TAC.

---

## 3. Check Known Issues/Bugs (Cisco 3100 Flow Offload)

- **Hardware Accelerator Initialization Errors:**  
  Check boot logs/console output.

- **Interface Speed/Duplex Bug (CSCwm35751):**  
  Ensure correct speed/duplex:
```
show interface
```

- **Unsupported Traffic:**  
  Only TCP/UDP traffic at Layer 3/4 offloads.

- **Dead Connection Detection (DCD):**  
  Should be disabled:
```
no service dead-connection-detection
```

- No additional licenses/modules required for offload on 3105.

---

## 4. Step-by-Step Guide to Resolve Flow Offload Issue

### Step 1: Confirm Offload Configuration  
Verify command:
```
flow-offload enable
```
Reboot after enabling.

**Explicit test policy (optional):**
```
access-list OFFLOAD-ALL permit ip any any
class-map OFFLOAD-ALL
  match access-list OFFLOAD-ALL
policy-map global_policy
  class OFFLOAD-ALL
    set connection advanced-options flow-offload
```

### Step 2: Reboot and Monitor Logs  
Watch console carefully during reboot for hardware errors.

### Step 3: Collect Offload Metrics  
Post-reboot:
```
show flow-offload info detail
show flow-offload statistics
```

### Step 4: Verify Interface Speed/Duplex  
Confirm correct speed:
```
show interface
```
Correct if needed.

### Step 5: Temporarily Disable Offload (Testing)  
Disable offload to test software throughput:
```
no flow-offload enable
```
Reboot and retest throughput for comparison.

### Step 6: Upgrade or Change Software Versions
- **Recommended:** Upgrade ASA to latest (9.23.x) and FXOS (2.17.x).
- **Alternative:** Downgrade temporarily to ASA 9.22(1.x) with FXOS 2.16 for stability testing.

### Step 7: Contact Cisco TAC (If Necessary)  
Open case with Cisco TAC, provide:
- ASA/FXOS versions (`show tech`)
- Boot logs and console/syslog errors
- Throughput tests (with and without offload)

### Step 8: Verify Throughput After Resolution  
Use tools like `iperf` to confirm expected multi-gigabit throughput.

---

## References
- Cisco ASA 9.23 Release Notes (Flow Offload default enabled)
- Cisco ASA Configuration Guide (Flow offload configuration)
- Cisco ASA Syslog Guide (Flow offload messages)
- Cisco Bug CSCwm35751 (3100 interface stuck at 100 Mbps)
- Cisco Secure Firewall Compatibility Guides (FXOS/ASA versions)
- Cisco NetSecOPEN Performance Blog (3105 performance benchmarks)
