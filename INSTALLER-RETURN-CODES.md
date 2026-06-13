# RealNews Listener — Installer Return Codes

The RealNews Listener installer (`RealNewsListener-Setup-x.x.x.exe`) is built with
[Inno Setup](https://jrsoftware.org) and returns the standard Inno Setup exit codes.

## Silent installation

```
RealNewsListener-Setup-1.0.0.exe /VERYSILENT /NORESTART /SUPPRESSMSGBOXES
```

A successful silent installation exits with code **0**. No reboot is required.

## Exit codes

| Code | Meaning | Category |
|---|---|---|
| 0 | Installation completed successfully | Success |
| 1 | Setup failed to initialize | Failure |
| 2 | The user cancelled before installation started | Cancelled by user |
| 3 | Fatal error during the preparation phase (e.g. invalid command line) | Failure |
| 4 | Fatal error during the installation phase | Failure |
| 5 | The user cancelled during installation | Cancelled by user |
| 6 | The Setup process was forcefully terminated | Failure |
| 7 | The "Preparing to Install" stage determined installation cannot proceed | Failure |
| 8 | The "Preparing to Install" stage determined a restart is required before installing | Reboot required, then retry |

Any exit code not listed above should be treated as a failure.

## Uninstaller

The uninstaller (`unins000.exe` in the installation directory) supports the same
silent switches (`/VERYSILENT /NORESTART`) and returns **0** on successful removal.

## Reference

Canonical documentation for these codes is maintained by Inno Setup:
<https://jrsoftware.org/ishelp/index.php?topic=setupexitcodes>

---
*RealNews Listener is a product of N2NHU Labs, distributed under the GNU GPL v3.*
