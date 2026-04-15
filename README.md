# ARIA-MATLAB (Modernized Fork)
**ARIA Interfaces for Matlab and Simulink**

[![C/C++](https://img.shields.io/badge/C%2F++-Modernized-00599C.svg)]()
[![MATLAB](https://img.shields.io/badge/MATLAB-R2025b%20Ready-e16f3d.svg)]()

> **⚠️ FORK NOTICE: MODERNIZED FOR MODERN LINUX & MATLAB**
> This repository is a modernized fork of the original `aria-matlab` interface (v2013) by Adept MobileRobots. The original codebase contained legacy C patterns, linker path errors, and thread-safety violations (`mexPrintf`) that caused hard crashes on modern MATLAB versions.
> 
> This fork specifically targets the integration of the **Pioneer-3AT** robot with modern Linux distributions (Ubuntu/Zorin) and recent MATLAB releases, ensuring thread-safe logging and correct compilation.

---

## 🛠️ Compatibility & Prerequisites
* **ARIA Library:** You must have the modernized ARIA library installed in `/usr/local/Aria`.
* **OS:** Linux (Ubuntu 22.04 / Zorin OS 17 or newer).
* **Compiler:** GCC/G++ with GNU Make.
* **MATLAB:** R2012a through R2025b (Tested on 64-bit).

---

## 🚀 Installation & Build Guide

The installation directory is strict. This repository **must** be cloned as a subdirectory inside your ARIA installation path.

### Step 1: Clone and Configure Permissions
```bash
cd /usr/local/Aria

# Clone the repository
sudo git clone https://github.com/filpsl/aria-matlab.git

# Rename the folder exactly to 'matlab'
sudo mv aria-matlab matlab

# CRITICAL: Take ownership of the folder to prevent Sudo/Path loops during build
sudo chown -R $USER:$USER /usr/local/Aria/matlab
sudo chown -R $USER:$USER /usr/local/Aria/lib
```

### Step 2: Build the C Library (`libariac`) and .MEX files.
Navigate into the folder and build the library. 
> **⚠️ WARNING:** **Do NOT use `sudo make`**. Using `sudo` resets your system `PATH` and will cause the build to fail because it won't find the `matlab` executable.

```bash
cd /usr/local/Aria/matlab
make clean
make
```

Update the Linux linker cache so the system recognizes the new `libariac.so`:
```bash
sudo ldconfig
```

---

## ⚙️ Quickstart & Usage

Initialize the library and connect to the Pioneer-3AT robot. 

**Standard USB-Serial Connection:**
```matlab
% Default baud rate is usually 9600 or 115200
aria_init('-rp', '/dev/ttyUSB0'); 
arrobot_connect;
arrobot_setvels(200, 0, 0); % Move forward
```

**Remote Connection (TCP to Serial Bridge):**
If the robot is connected to an onboard computer and you are running MATLAB on a remote machine, you can use `socat` on the onboard computer to create a raw TCP bridge.
*On the Robot's Onboard Computer:*
```bash
sudo socat TCP-LISTEN:8101,fork,reuseaddr /dev/ttyS0,raw,echo=0
```
*In MATLAB (Remote PC):*
```matlab
aria_init('-rh', '[robot_ip]');
arrobot_connect;
```

---

## 🩺 Troubleshooting

### 1. MATLAB Crashes / "Assertion detected"
* **Cause:** Older versions of this wrapper used `mexPrintf` for asynchronous ARIA background threads, which violates MATLAB's strict single-threaded API, causing memory assertion failures.
* **Fix:** This fork overrides the `mex.h` macro (`#undef printf`) and uses standard `fprintf(stdout)` in `aria_init.c` and `ariac.cpp`. If you edit the C source code, **always recompile from inside MATLAB** (`makemex`) to ensure the changes are applied.

### 2. Error: `ArRobot::myPacketReader: Timed out`
This means the serial port was opened, but the robot is not responding to the SIP (Server Information Packet) requests.
* **Permission Issue:** Ensure your Linux user is in the `dialout` group to read serial ports without sudo: `sudo usermod -aG dialout $USER` (Reboot required).
* **Wrong Port:** ARIA defaults to `/dev/ttyS0`. If using USB, explicitly pass `aria_init('-rp', '/dev/ttyUSB0')`.
* **Baud Rate Mismatch:** Ensure your `socat` bridge specifies the exact baud rate of the robot's logic board (e.g., `b115200`).

---

# 📖 Original Documentation (Legacy Reference - v2013)

*Note: The following text is preserved from the original 2013 repository for copyright and architectural reference.*

% ARIA Interfaces for Matlab and Simulink
% September 19, 2013

These interfaces provide a handful of essential features of ARIA in Matlab
or Simulink.

## Available Functions
So far, the following functions are available:

* `aria_init` [args] - Initialize ARIA and store any ARIA arguments given. Must be called first.
* `arrobot_connect` - Connect to the robot and begin background communications/processing cycle thread.
* `arrobot_disconnect` - Disconnect from the robot and free used memory.
* `[x, y, th] = arrobot_getpose` - Get current position estimate.
* `arrobot_setvel(v)` - Set translational velocity.
* `arrobot_setrotvel(v)` - Set rotational velocity.
* `arrobot_setvels(x, y, th)` - Set all velocity components.
* `arrobot_stop` - Stop the robot.
* `v = arrobot_getbatteryvoltage` - Get battery voltage.
* `s = arrobot_isstalled` - Return 1 if either wheel motor is in stalled state.
* `e = arrobot_motorsenabled` - Return 1 if the motors are enabled.
* `arrobot_enable_motors` / `arrobot_disable_motors` - Enable or disable the motors.
* `[s1, s2, s3... sN] = arrobot_getsonar` - Get ranges (mm) of all sonar sensors.

## About ariac
ariac is a simple functional C wrapper to ARIA that provides the minimum API for connecting to a robot, receiving data from it, and controlling it by requesting velocities. All necessary ARIA objects and other state are stored internally by ariac, so the caller (e.g. Matlab) does not need to store and pass in any object references, handles, identifiers or any other state -- only call functions.

## Acknowledgements
Special thanks to Mathworks for their support in developing these interfaces, especially Fred Banser for his work developing the Simulink interface and ability to deploy and run via external mode.
