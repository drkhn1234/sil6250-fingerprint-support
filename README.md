# SIL6250 fingerprint support on Huawei MateBook 14

Findings, methods, and validation results from enabling the **Silead/Petaic
SIL6250** reader on a **Huawei MateBook 14 2024 (FLMH-XX)** running Ubuntu 26.10.

## End result

Fingerprint authentication is working on the tested machine through the Linux
sensor driver and an isolated Wine helper running the official Petaic matching
library. The right index finger is enrolled, the standard PAM fingerprint profile
is enabled, and password fallback is retained.

| Final check | Observed result |
| --- | --- |
| Fresh right-index verification | 6 of 8 presses recognized |
| Fresh unenrolled-thumb verification | All 8 presses rejected |
| Actual `sudo` PAM authentication, password input disabled | Passed on the first attempt |
| Service restart and saved-template reload | Passed |
| Automated tests | 20 library + 8 daemon tests passed; 1 optional test skipped |

Recognition still needs occasional retries or password fallback. These results
come from one participant and a small sample set. They do not establish a
population false-accept rate, spoof resistance, or Windows Hello equivalence.

## Main findings

- The kernel module, TLS provisioning, transport, and image capture worked.
  Enrollment completing did not establish that the native Linux matcher could
  recognize separate presses.
- Comparing the same captured images through the Linux matcher and the official
  Windows library exposed a substantial matching gap.
- Matching more frames alone improved Linux recognition but also accepted a
  wrong-finger control. That variant was excluded from authentication.
- The vendor feature path uses compact binary signatures and an additional
  descriptor that the tested Linux implementation does not fully reproduce.
- The working interim integration runs the pinned vendor library as a separate
  unprivileged account, with a root-only Unix socket and private feature-template
  storage. A reliable native replacement for this unit remains unfinished.

## Report and evidence

- [Full investigation report](REPORT.md): hardware identification, capture
  diagnostics, differential tests, reverse-engineering observations, integration,
  limitations, and remaining native-driver work.
- [Aggregate validation results](results/validation-summary.json): the measured
  counts and exact tested source revisions in machine-readable form.

The published evidence consists of aggregate measurements. Fingerprint images,
feature vectors, enrollment templates, signing material, proprietary binaries,
and decompiled vendor code remain outside this repository.

Investigation date: **2026-10-26**. The device-specific workaround builds on
[AlexDaichendt/sil6250-linux](https://github.com/AlexDaichendt/sil6250-linux) and
[uunicorn/open-fprintd](https://github.com/uunicorn/open-fprintd).
