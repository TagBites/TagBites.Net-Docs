# Release Notes

## 1.0.2 (2024-07-31)
- fix: controller task method returns null value

## 1.0.1 (2024-07-30)
- feat: `Server.ListenAsync()` — start listening without a background thread indirection, awaitable directly.
- fix: controller task method incorrect values

## 1.0.0 (2023-07-03)
- feat: only error message and full exception (remote RMI exceptions now carry both a short message and the full remote exception text — see `NetworkControllerInvocationException`)
- chore: MIT license, removed license-check code
- chore: updated solution structure, target frameworks, and samples

<!-- TODO: the exact 1.0.0-beta4 tag doesn't exist in the git history, so changes between beta4 and this first stable release couldn't be pinned to a precise commit range — verify with the maintainer if a complete list is needed. -->

## 1.0.0-beta4
- feat: custom network configuration support

## 1.0.0-beta3
- fix: no message result for exception in RMI

## 1.0.0-beta2
- more universal controller resolver

## 1.0.0-beta1
- first version