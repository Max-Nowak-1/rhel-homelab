# 05 - SELinux

## Overview

This section focuses on **Security-Enhanced Linux (SELinux)** and how it provides mandatory access control through security contexts, policies, file labeling, port labeling, and booleans.

The goal of these labs was to understand not only the commands used to manage SELinux, but also how SELinux makes security decisions and how to troubleshoot and maintain SELinux configurations.

---

## Topics Covered

- SELinux operating modes
- Enforcing vs. Permissive vs. Disabled
- SELinux configuration
- Security contexts
- SELinux users, roles, and types
- Process domains
- File contexts
- chcon
- semanage fcontext
- restorecon
- SELinux port labeling
- semanage port
- SELinux-aware file copying
- cp --preserve=context
- SELinux booleans
- Runtime vs. persistent SELinux changes
- Verification and troubleshooting

---

## Labs

### Lab 01 - Modify SELinux Operating Mode

**Objective:**  
Modify the system's SELinux operating mode and verify the resulting state after reboot.

**Environment:**

- RHEL 10
- SELinux enabled
- User account with sudo privileges

**Procedure:**

1. Check the current SELinux operating mode:

        getenforce

2. Edit the SELinux configuration file:

        sudo vi /etc/selinux/config

3. Change the configuration to:

        SELINUX=disabled

4. Reboot the system:

        sudo reboot

5. After the system boots, verify the SELinux state:

        sudo getenforce

6. Confirm that SELinux is disabled.

7. Restore the configuration:

        sudo vi /etc/selinux/config

8. Change the setting back to:

        SELINUX=enforcing

9. Reboot the system again:

        sudo reboot

10. Verify the system is enforcing SELinux:

        sudo getenforce

**Key Concepts:**

- /etc/selinux/config controls the SELinux operating mode configured for system startup.
- getenforce reports the current SELinux operating mode.
- Enforcing actively applies SELinux policy.
- Permissive logs policy violations without blocking the operation.
- Disabled turns SELinux off.
- Changing between Enforcing and Disabled through /etc/selinux/config requires a reboot.

**Verification:**

        getenforce

Expected final state:

        Enforcing

**What I Learned:**

This lab reinforced the difference between SELinux's configured boot-time state and its current runtime state. I also learned that disabling SELinux is fundamentally different from putting SELinux into permissive mode.

---

### Lab 02 - Modifying Context on Files

**Objective:**  
Modify the SELinux context of files and create a persistent file-context rule.

**Procedure:**

1. Create the test directory structure:

        sudo mkdir -p /tmp/d1/d2

2. View the existing SELinux context:

        sudo ls -Zd /tmp/d1
        sudo ls -Zd /tmp/d1/d2

3. Temporarily change the SELinux type recursively:

        sudo chcon -R -t etc_t /tmp/d1

4. Create a persistent SELinux file-context rule:

        sudo semanage fcontext -a -t etc_t "/tmp/d1(/.*)?"

5. Apply the persistent file-context definition:

        sudo restorecon -R /tmp/d1

6. Verify the resulting contexts:

        sudo ls -Zd /tmp/d1
        sudo ls -Zd /tmp/d1/d2

**Key Concepts:**

- SELinux contexts are security labels attached to files, directories, processes, and other objects.
- The SELinux type is an important part of the context used by SELinux policy.
- chcon changes the context directly on the filesystem.
- semanage fcontext creates a persistent file-context mapping.
- restorecon applies the expected SELinux context based on the persistent file-context configuration.
- -R performs the operation recursively.

**Important Distinction:**

        chcon
        ↓
        Changes the current filesystem label

        semanage fcontext
        ↓
        Defines the persistent labeling rule

        restorecon
        ↓
        Applies the persistent rule to the filesystem

**What I Learned:**

The important mental model from this lab was:

> Context = label  
> Policy = rules

The context itself does not determine whether an action is allowed. SELinux policy uses the security context information when deciding whether a process is permitted to access an object.

---

### Lab 03 - Add Network Port to Policy Database

**Objective:**  
Add a TCP network port to the SELinux policy database and associate it with an SELinux port type.

**Procedure:**

1. Check whether the port is already defined:

        sudo semanage port -l | grep 9005

2. Add TCP port `9005` to the `http_port_t` SELinux port type:

        sudo semanage port -a -t http_port_t -p tcp 9005

3. Verify the new mapping:

        sudo semanage port -l | grep 9005

**Key Concepts:**

- semanage port manages SELinux port-to-type mappings.
- -a adds a new port definition.
- -t specifies the SELinux port type.
- -p specifies the protocol.
- -d removes a port definition.
- -m modifies an existing port definition.

Example:

        sudo semanage port -a -t http_port_t -p tcp 9005

This associates TCP port 9005 with the SELinux port type http_port_t.

**Important Distinction:**

Assigning a port to http_port_t does **not** open the port in the firewall.

SELinux port labeling and firewall rules are separate security layers.

For example:

        SELinux
        ↓
        Determines whether a process domain is allowed to use a port type

        firewalld
        ↓
        Determines whether network traffic is allowed through the firewall

Both layers may need to be configured correctly for a network service to function.

**Cleanup:**

        sudo semanage port -d -t http_port_t -p tcp 9005

**What I Learned:**

This lab helped clarify that SELinux does not simply label a port as "an HTTP port." Instead, it associates the port with a SELinux type that policy can use when determining which processes are permitted to bind to or use that port.

---

### Lab 04 - Copy Files with and Without Context

**Objective:**  
Compare how SELinux contexts are handled when copying files normally and when explicitly preserving the source context.

#### Part 1 - Normal Copy

1. Create a test file:

        touch /tmp/sef1

2. View its SELinux context:

        ls -Z /tmp/sef1

3. Copy the file to /usr/local/:

        sudo cp /tmp/sef1 /usr/local/

4. Compare the source and destination contexts:

        ls -Z /tmp/sef1 /usr/local/sef1

The destination file may receive a context appropriate for its new location rather than simply retaining the source file's context.

#### Part 2 - Preserve SELinux Context

1. Create another test file:

        touch /tmp/sef2

2. View its SELinux context:

        ls -Z /tmp/sef2

3. Copy the file while preserving its SELinux context:

        sudo cp --preserve=context /tmp/sef2 /var/local/

4. Compare the contexts:

        ls -Z /tmp/sef2 /var/local/sef2

**Key Concepts:**

- A file's SELinux context is independent of the file's contents.
- Normal file copying can result in the destination receiving a context appropriate for its destination location.
- --preserve=context tells cp to preserve the source SELinux security context.
- SELinux labeling is important because policy decisions depend on these labels.

**What I Learned:**

A successful file copy does not necessarily mean the source and destination will have identical SELinux contexts. Context behavior depends on how the file is copied and whether its security context is explicitly preserved.

---

### Lab 05 - Flip SELinux Booleans

**Objective:**  
Modify an SELinux boolean and understand the difference between runtime and persistent boolean changes.

**Procedure:**

1. Check the current state of the boolean:

        getsebool ssh_use_tcpd

2. Change the boolean at runtime:

        sudo setsebool ssh_use_tcpd 0

3. Verify the new state:

        getsebool ssh_use_tcpd

4. A value of 0 represents:

        off

5. A value of 1 represents:

        on

6. SELinux booleans can also be configured persistently with the -P option:

        sudo setsebool -P ssh_use_tcpd 0

7. Verify the persistent configuration:

        getsebool ssh_use_tcpd

**Key Concepts:**

- SELinux booleans provide configurable policy controls.
- getsebool displays the current boolean state.
- setsebool changes a boolean.
- 0 and off represent a disabled boolean.
- 1 and on represent an enabled boolean.
- setsebool without -P changes the runtime state.
- setsebool -P makes the change persistent.

**Important Distinction:**

        setsebool
        ↓
        Runtime change

        setsebool -P
        ↓
        Persistent change

This is similar to the runtime vs. permanent configuration concept encountered with firewalld.

**Cleanup:**

        sudo setsebool -P ssh_use_tcpd off

**What I Learned:**

SELinux booleans provide a way to enable or disable specific policy behaviors without modifying the underlying SELinux policy. I also reinforced the difference between changing a setting temporarily and making that setting persistent.

---

## Skills Demonstrated

- Checking and modifying SELinux operating modes
- Working with SELinux security contexts
- Understanding SELinux types and process domains
- Managing persistent file-context rules
- Using chcon and restorecon
- Managing SELinux port types with semanage
- Understanding SELinux port labeling vs. firewall configuration
- Preserving SELinux contexts during file operations
- Managing SELinux booleans
- Distinguishing runtime and persistent security configuration
- Verifying SELinux configuration from the command line
- Troubleshooting SELinux labeling issues

---

## Key Takeaways

The major SELinux concepts reinforced throughout these labs were:

### 1. Context vs. Policy

        Context = Label
        Policy = Rules

SELinux uses security contexts as attributes when applying its policy decisions.

### 2. Process Domains vs. File Types

A process has a SELinux **domain**, while files and other objects have SELinux **types**.

Example:

        sshd process
        ↓
        sshd_t

        SSH configuration file
        ↓
        etc_t

The domain and object type are used by SELinux policy when evaluating access.

### 3. Persistent File Labeling

        semanage fcontext
        ↓
        Defines what the context should be

        restorecon
        ↓
        Applies the defined context

        chcon
        ↓
        Directly changes the current label

### 4. SELinux Ports vs. Firewalls

        semanage port
        ↓
        SELinux port labeling/policy

        firewall-cmd
        ↓
        Network traffic filtering

These are separate security controls and should not be treated as the same thing.

### 5. Runtime vs. Persistent Configuration

SELinux contains several configuration mechanisms where understanding whether a change is temporary or persistent is important.

Examples:

        setsebool
        ↓
        Runtime

        setsebool -P
        ↓
        Persistent

Understanding this distinction is essential when troubleshooting configuration changes that appear to disappear after a reboot or reload.

---

## Verification Commands

Useful commands from these labs include:

    getenforce
    sestatus
    ls -Z
    ls -Zd
    ps -eZ
    chcon
    semanage fcontext
    restorecon
    semanage port
    getsebool
    setsebool

---

## Lessons Learned

These labs helped move SELinux from being something I was memorizing into something I can reason about.

The most important takeaway was understanding that SELinux works through multiple layers:

        Process / Domain
                ↓
        Security Context
                ↓
        SELinux Policy
                ↓
        Allow / Deny

I also gained practical experience managing file labels, persistent file-context rules, SELinux port types, booleans, and SELinux operating modes.

The troubleshooting exercises reinforced the importance of checking the actual security context and configuration rather than assuming a permission problem is caused by traditional Linux file permissions alone.
