# LD_PRELOAD Userland Rootkit Analysis

## Context

The investigation started from an alert on an SSH server reporting unusual library linking errors and inconsistencies in filesystem visibility (missing directories despite being expected).

This suggested a potential manipulation at runtime rather than actual file deletion.



## Analysis

A userland rootkit was suspected.

On Linux systems, dynamic linking can be abused using mechanisms such as `LD_PRELOAD` or `/etc/ld.so.preload`, allowing a malicious shared library to be injected into all dynamically linked processes.

This can alter the behavior of standard system utilities without modifying the underlying filesystem.

The dynamic linker configuration was therefore reviewed.

The system was found to use a non-standard shared library configured through the preload mechanism:

```bash
cat /etc/ld.so.preload
```
A custom library was referenced, indicating forced injection at runtime.

To confirm the impact on system binaries, dynamic dependencies of standard utilities were inspected:
```bash
ldd /bin/ls
```
The output confirmed that the suspicious library was loaded alongside standard system libraries, indicating that core system tools were being affected at runtime.

## Findings & Impact

Analysis of the injected library showed that it hooks multiple libc functions, including directory listing, file access, and string filtering functions.

By intercepting these calls, the rootkit is able to:

- hide files and directories
- filter specific names from output
- restrict visibility of certain resources

This explains the inconsistencies observed in filesystem enumeration.

Once the malicious component was no longer active, system behavior returned to normal. Previously hidden files became visible again through standard enumeration tools, confirming that the filesystem itself was intact and only its representation had been altered at runtime.

This validated the presence of a userland rootkit manipulating system output through dynamic linking interception.

## Conclusion

The issue was caused by a userland rootkit leveraging the dynamic linker preload mechanism.

By injecting a malicious shared library into processes, the attacker was able to modify the behavior of system utilities and hide files without altering the filesystem.

This highlights the importance of verifying dynamic linker configurations during incident response investigations.
