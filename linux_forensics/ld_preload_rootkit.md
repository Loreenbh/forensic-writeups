# LD_PRELOAD Userland Rootkit Analysis

## Context
The investigation started from an alert on an SSH server reporting unusual library linking errors and missing directories that were expected to exist.
This suggested that the issue was not caused by file deletion, but by something modifying how the system displays information.


## Analysis
A userland rootkit was suspected.
On Linux systems, a mechanism called `LD_PRELOAD` or `/etc/ld.so.preload` can be used to force the system to load a shared library before other libraries. This can change how system commands behave.
The dynamic linker configuration was checked:

```bash
cat /etc/ld.so.preload
```
A custom library was found:
```bash
/lib/x86_64-linux-gnu/libc.hook.so.6
```
This indicates that a non-standard library is being loaded automatically by the system.

To confirm its effect, the dependencies of a system command were checked:
```bash
ldd /bin/ls
```
The output showed that this library was loaded together with normal system libraries. This means that basic commands like ls are affected.

## Findings

The injected library was analyzing system calls and modifying their behavior.

It hooks important functions such as:
- file listing functions
- file opening functions
- string search functions

Because of this, it can:
- hide files or folders
- filter certain names
- change what the user sees

This explains why some files and directories appeared to be missing.

## Conclusion

The issue was caused by a userland rootkit using the Linux preload mechanism.
By loading a malicious shared library into processes, the attacker was able to modify system behavior and hide files without deleting them.
This shows that system commands are not always reliable when a system is compromised.
