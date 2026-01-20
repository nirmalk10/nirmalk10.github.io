# Docker Build Fixes for Sionna Research Kit and OpenAirInterface (OAI)

This page documents the concrete Docker-level changes required to successfully build the Sionna Research Kit with OpenAirInterface (OAI) on a JetPack 6–aligned environment. These changes were necessary due to mismatches between default assumptions in the build system and newer OS, CUDA, compiler, and UHD versions.

All changes below reflect **active modifications only**; commented or experimental lines are intentionally excluded.

---

## Environment Context

- **Target platform:** Jetson AGX Orin  
- **JetPack:** 6 (L4T R36)  
- **OS:** Ubuntu 22.04  
- **CUDA:** 12.x  
- **SDR:** USRP B206-mini  

---

## Base Image Alignment (Ubuntu 22.04)

The default Dockerfiles referenced CUDA images based on Ubuntu 24.04. This caused dependency and ABI incompatibilities with OAI. The base image was updated to Ubuntu 22.04 to align with JetPack 6 and the surrounding toolchain.

### Patch

```diff
--- a/docker/Dockerfile.base.ubuntu.cuda
+++ b/docker/Dockerfile.base.ubuntu.cuda
@@ -25,7 +25,8 @@
-ARG BASE_IMAGE=nvcr.io/nvidia/cuda:13.0.1-devel-ubuntu24.04
+#ARG BASE_IMAGE=nvcr.io/nvidia/cuda:13.0.1-devel-ubuntu24.04
+ARG BASE_IMAGE=nvcr.io/nvidia/cuda:13.0.1-devel-ubuntu22.04  
```

```diff

--- a/docker/Dockerfile.gNB.ubuntu.cuda
+++ b/docker/Dockerfile.gNB.ubuntu.cuda
@@ -25,7 +25,8 @@
-ARG BASE_IMAGE=nvcr.io/nvidia/cuda:13.0.1-devel-ubuntu24.04
+#ARG BASE_IMAGE=nvcr.io/nvidia/cuda:13.0.1-devel-ubuntu24.04
+ARG BASE_IMAGE=nvcr.io/nvidia/cuda:12.6.2-devel-ubuntu22.04
```
Rationale:
Ubuntu 22.04 provides a stable compatibility point for JetPack 6 without introducing the breaking changes present in 24.04.

UHD Version Update (4.8 → 4.9)
The build was originally pinned to UHD 4.8, which resulted in incompatibilities with newer kernels and Ettus devices. The UHD version was updated to 4.9.

Patch
```diff

--- a/docker/Dockerfile.base.ubuntu.cuda
+++ b/docker/Dockerfile.base.ubuntu.cuda
@@ -33,7 +34,7 @@
-ENV UHD_VERSION=4.8.0.0
+ENV UHD_VERSION=4.9.0.0
```

```diff

--- a/docker/Dockerfile.gNB.ubuntu.cuda
+++ b/docker/Dockerfile.gNB.ubuntu.cuda
@@ -119,7 +122,7 @@
-COPY --from=gnb-base /usr/local/lib/libuhd.so.4.8.0 /usr/local/lib
+COPY --from=gnb-base /usr/local/lib/libuhd.so.4.9.0 /usr/local/lib
```

```diff

--- a/docker/Dockerfile.gNB.ubuntu.cuda
+++ b/docker/Dockerfile.gNB.ubuntu.cuda
@@ -143,7 +170,7 @@
-        /usr/local/lib/libuhd.so.4.8.0
+        /usr/local/lib/libuhd.so.4.9.0
```

Rationale:
UHD 4.9 improves compatibility with modern kernels and avoids runtime linker failures observed with UHD 4.8.

Compiler Upgrade for FlexRIC (GCC ≥ 12)
FlexRIC enforces a minimum GCC version of 12. The default toolchain did not meet this requirement, causing build failures. GCC 12 was explicitly installed and set as the default compiler.

Patch
```diff

--- a/docker/Dockerfile.build.ubuntu.cuda
+++ b/docker/Dockerfile.build.ubuntu.cuda
@@ -63,6 +63,13 @@
+RUN apt-get update && apt-get install -y \
+    gcc-12 g++-12 \
+ && update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-12 120 \
+ && update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-12 120
```

Rationale:
Without this change, the FlexRIC build fails early due to strict compiler version checks.

Boost Version Adjustment (1.83 → 1.74)
The Dockerfile originally requested Boost 1.83, which caused ABI and availability issues on Ubuntu 22.04. The Boost version requirement was downgraded to 1.74.

Patch
```diff

--- a/docker/Dockerfile.gNB.ubuntu.cuda
+++ b/docker/Dockerfile.gNB.ubuntu.cuda
@@ -39,7 +40,7 @@
-ARG BOOST_VERSION="1.83.0"
+ARG BOOST_VERSION="1.74.0"
```

Rationale:
Boost 1.74 is readily available on Ubuntu 22.04 and avoids link-time and runtime incompatibilities.


## Docker Compose Changes for `oai-gnb`

In addition to Dockerfile-level fixes, several changes were required in `docker-compose.yaml` for the `oai-gnb` service. These changes addressed runtime permission issues, UHD access problems, and scheduling stability when running the gNB with a USRP B206-mini.

### Privileged Container Execution

The gNB container was configured to run in privileged mode:

```yaml
privileged: true

```
The container was explicitly limited to eight CPU cores:
```
deploy:
  resources:
    limits:
      cpus: "8.0"
```
Capability dropping was initially evaluated but ultimately disabled:
```
#cap_drop:
#  - ALL
```

Dropping all Linux capabilities prevented UHD from accessing the USRP device correctly.

The UHD runtime libraries inside the container were explicitly aligned with the host UHD installation:
```
volumes:
  - /usr/local/lib/libuhd.so.4.9.0:/usr/local/lib/libuhd.so.4.9.0
  - /usr/local/lib/libuhd.so:/usr/local/lib/libuhd.so
```

This ensured that the gNB used UHD 4.9 consistently at runtime.



Summary
These Docker-level changes enabled a stable and reproducible build of the Sionna Research Kit with OAI on a JetPack 6 environment:

Ubuntu 22.04 base images

CUDA 12.x

UHD 4.9

GCC 12

Boost 1.74

Rather than forcing legacy assumptions onto a newer platform, the build configuration was updated to explicitly accept newer toolchains while maintaining runtime stability and hardware compatibility.
