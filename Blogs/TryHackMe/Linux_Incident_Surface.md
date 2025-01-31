# Introduction and Scenario

- The Linux Incident Surface focuses on all potential points or sources in the Linux system where an incident could occur or traces of incidents could be found. This could lead to a security breach, which could also be part of the Linux Attack Surface.
- Daniel is a Red teamer, and Liam is a Blue teamer at Cybertees Pvt Ltd. Daniel will help us perform post-exploitation activities. Liam will help us examine various incident surfaces to identify the footprints of the attack.

## Overview of Linux Attack and Incident Surface

- **The Linux Attack Surface** refers to all the points of interaction in a Linux system where an adversary might attempt to exploit vulnerabilities to gain unauthorized access or carry out malicious activities. One of the main purposes of identifying the attack surface is to reduce the number of entry points that the attackers could potentially exploit.
- Some of the key entry points that could be identified as part of the Linux Attack Surface are: *Open Ports*, *Running Services*, *Running vulnerable software or applications*, and *Network communication*.
- The primary goal is to minimize the attack surface by reducing potential weaknesses from the areas the attacker could exploit. Some of the steps that are involved in achieving this goal are: *Identifying and patching the vulnerabilities*, *Minimizing the usage of unwanted services*, *Check the interfaces where the user interacts*, and *Minimizing the publically exposed services, applications, ports, etc*.

- **The Linux Incident Surface**, on the other hand, refers to all the system areas involved in the detection, management, and response to an actual security incident (post-compromise). It includes where security breaches may be detected and analyzed and where a response plan must be implemented to mitigate the incident.
- The main purpose of identifying the incident surface is to hunt down, detect, respond to, and recover from the incident if it has occurred. A security analyst would monitor all areas within the operating system where any traces or footprints of the attack could be found. Some of the key points where we could find the incident traces are highlighted here: *System logs*, *auth.log, syslog, krnl.log, etc*, *Network traffic*, *Running processes*, *Running services*, and *File and Process Integrity*.
- Understanding the incident surface is key to efficiently responding to an ongoing attack, mitigating damage, recovering affected systems, and applying lessons learned to prevent future incidents.

### 1. Attacks & Investigation: Processes and Network Communication

- Processes and network communication are crucial in any operating system in incident investigations. Monitoring and analyzing processes, especially those with network communication, can help identify and address security incidents. Running processes are a key part of the Linux Incident Surface, as they could represent a potential source of evidence of an attack.
- **Investigating Process:** *1. Create and Run a simple process*, there is a directory `/home/activities/process`, there is a code called `simple.c`. Compile and run the process.
![Check the image for your reference](/Blogs/Images/image50.png)
- Now that the program has been compiled and executed from the `/tmp/` directory, let's keep the process running and open a new terminal. We will now explore how to find traces of this activity that could be classified as an incident.
- `ps aux | grep simple`, command displays all processes for all users in a detailed format. The flags are explained here: `a` - shows processes for all users. `u` - User-oriented format. `x` -  Includes processes not attached to a terminal (useful for finding background processes).
![Check the image for your reference](/Blogs/Images/image51.png)
- Let's examine the files/resources connected with this process using the lsof tool. This tool requires the PID to be provided as an argument, as shown below. Command: `lsof -p 2260`
![Check the image for your reference](/Blogs/Images/image52.png)
- **Investigating Network Communication:** In many incident scenarios, processes communicating via network to an external IP or the server are worth investigating. Therefore, examining the processes of making network connections and hunting down suspicious ones is very important.
- To demonstrate, we will execute a process called `netcom` placed in the `/home/activities/processes` directory. Execute the process using the command ./netcom. This process will establish a network connection to a remote IP.
- The first step would be to confirm whether the process is running on the system. Run the command `ps aux | grep netcom` to filter the results.
![Check the image for your reference](/Blogs/Images/image53.png)
- Let's use the following command `lsof -i -P -n` in another terminal to see if there is any network connection associated with this PID, as shown below: *lsof - List open files*, *-i, network connections, including sockets and open network files*, *-P, display port numbers*, and *-n, shows IP address instead of resolving them to hostnames*.
![Check the image for your reference](/Blogs/Images/image54.png)
- **Utilizing Osquery:** Osquery to explore processes and its network connection.
- Osquery command: `SELECT pid, fd, socket, local_address, remote_address FROM process_open_sockets WHERE pid = 2322;`
![Check the image for your reference](/Blogs/Images/image55.png)
- **Where Processes Fit in the Linux Incident Surface:** Processes can be exploited, manipulated, or used by attackers to execute malicious activities. It is important to investigate the processes running on the system from various angles: *A process running from a tmp directory (context matters).*, *A suspicious child-parent process.*, *Process with a suspicious network connection.*, *Orphan process. Not all orphan processes are suspicious, but those with no parent process associated after execution are worth investigating.*, *Suspicious processes that are running through a cronjob.*, and *System-related processes or binaries running from the tmp directory or user directories.*.

#### 2. Persistence
