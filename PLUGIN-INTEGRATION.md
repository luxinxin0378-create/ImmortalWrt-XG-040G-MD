# Nokia XG-040G-MD plugin integration and compatibility notes

## Scope and safety

This is a staged integration plan for the custom Airoha AN7581 target. It does not certify that a firmware image boots on every NAND variant. No firmware should be flashed until the image, partition layout, NAND variant, recovery method, and checksums have been verified on the actual device.

## First-pass package set

The Actions workflow currently targets `openwrt-25.12-XG-040G-MD` only, because the selected Argon theme source is explicitly on its `openwrt-25.12` branch and the proxy package dependencies are checked against the 25.12 feeds.

The first-pass set includes:
- Existing QModem and the existing USB serial, QMI, MBIM, NCM, ECM and RNDIS driver selections.
- AdGuard Home plus its LuCI app.
- Argon theme.
- LuCI bandwidth accounting (`luci-app-nlbwmon`).
- LuCI Statistics and selected collectd modules.
- Browser terminal (`luci-app-ttyd`).
- SQM (`luci-app-sqm` and `sqm-scripts`).
- HomeProxy, PassWall and OpenClash interfaces.
- OpenClash's `dnsmasq-full` requirement; ordinary `dnsmasq` is intentionally deselected because both variants must not be selected together.

Installing multiple proxy interfaces does not mean all proxies should run at once. Configure only one transparent-proxy/DNS owner at a time to avoid port, routing and firewall conflicts.

## Deferred from the first image

### SSR Plus
Not enabled in the first-pass config. The package name and its full dependency set must be validated against the selected 25.12 source. Its legacy iptables-oriented components can conflict with a firewall4/nftables build. Do not force-install it just to make the menu entry appear; test it in a separate profile after the first build passes.

### Turbo ACC
Not enabled. The LuCI page alone does not provide kernel acceleration. SFE/shortcut-fe and similar components can require target-specific kernel patches and may not work with this Airoha kernel or firewall4. First validate the exact upstream patch set against the selected kernel and the target's networking stack. Do not enable undocumented acceleration flags.

### Docker / containers
Not enabled. Before adding Docker, verify kernel support for namespaces, cgroups, overlayfs, veth, seccomp and required netfilter features; then verify writable storage/extroot space. The device's 256 MB NAND is not all available for packages, and container images should not be stored in the firmware root filesystem. If the checks pass, test Docker as a separate optional build profile with external USB storage/extroot.

## Build validation gates

1. Update and install all feeds before loading the final `.config`.
2. Run `make defconfig` and confirm all requested first-pass packages remain selected. Missing packages should fail the build rather than silently produce a reduced image.
3. Download package sources and compile; do not treat a successful configuration as a successful compile.
4. Inspect the image manifest, device profile, image sizes, NAND/UBI layout and SHA-256 values.
5. Keep the first successful build as a test artifact; do not automatically publish it as a release.
6. Do not flash until recovery is available and the actual device's NAND variant and partition layout have been verified.

## Current known risks

- The custom source's device image size is different from the official upstream profile; do not change image-size or partition values without validating the custom patches and layout.
- The source has device-specific patches for two NAND variants, but a single image working on both variants is not proven by source inspection alone.
- PassWall/OpenClash/HomeProxy can coexist as packages but can conflict at runtime if multiple services claim DNS interception, transparent proxying or firewall hooks.
- The workflow intentionally uploads a test artifact rather than creating a GitHub Release.
