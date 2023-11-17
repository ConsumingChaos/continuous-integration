FROM nixos/nix

ENV DEBIAN_FRONTEND="noninteractive"
ENV LANG "C.UTF-8"
ENV LANGUAGE "C.UTF-8"
ENV LC_ALL "C.UTF-8"

# Enable Nix Flakes and new commands
RUN echo "experimental-features = nix-command flakes" >> /etc/nix/nix.conf

# Setup nix-ld-rs for prebuilt non-Nix binaries (https://github.com/nix-community/nix-ld-rs)
RUN nix profile install \
    github:nix-community/nix-ld-rs \
    nixpkgs#glibc.out \
    nixpkgs#stdenv.cc.cc.lib && \
    mkdir /lib64 && \
    ln -s /nix/var/nix/profiles/default/libexec/nix-ld /lib64/ld-linux-x86-64.so.2 && \
    echo "common --action_env=NIX_LD" >> /etc/bazel.bazelrc && \
    echo "common --action_env=NIX_LD_LIBRARY_PATH" >> /etc/bazel.bazelrc
ENV NIX_LD="/nix/var/nix/profiles/default/lib64/ld-linux-x86-64.so.2"
ENV NIX_LD_LIBRARY_PATH="/nix/var/nix/profiles/default/lib"

### Install packages required by Bazel and its tests
### All Python dependencies of `bazelci.py`.
### Everything from `defaultShellUtils` (https://github.com/NixOS/nixpkgs/blob/master/pkgs/development/tools/build-managers/bazel/bazel_6/default.nix)
### NOTE 1: Some packages get implicitly pulled in, so conflicts are resolved with `--priority 1` annotations.
### NOTE 2: `bash` is specifically replaced by `bashInteractive` as the former is rarely what's actually wanted.
### NOTE 3: `nix` is specifically installed here to ensure the version we try to use has the same `glibc` that `nix-ld` is using.
###
### TROUBLESHOOTING: If any utilities error with `<binary>: error while loading shared libraries: __vdso_gettimeofday: invalid mode for dlopen(): Invalid argument`
### or something roughly like this, this indicates a `glibc` mismatch.  The fix is to make sure that utility is listed in the list below. This issue occurs
### where the utility is already present in the base Docker image but isn't built against the same `glibc` that `nix-ld` is using, leading to the issue when
### the utility is run from `bazel` (via `nix-ld`) which has its loader and `LD_LIBRARY_PATH` overwritten.
RUN nix profile install \
    nixpkgs#bashInteractive \
    nixpkgs#bazel-buildtools \
    nixpkgs#bazelisk \
    nixpkgs#coreutils \
    nixpkgs#diffutils \
    nixpkgs#dnsutils \
    nixpkgs#ed \
    nixpkgs#expect \
    nixpkgs#file \
    nixpkgs#findutils \
    nixpkgs#gawk \
    nixpkgs#git --priority 1 \
    nixpkgs#glibc \
    nixpkgs#gnugrep \
    nixpkgs#gnupatch \
    nixpkgs#gnupg \
    nixpkgs#gnused \
    nixpkgs#gnutar \
    nixpkgs#gzip --priority 1 \
    nixpkgs#iproute2 \
    nixpkgs#iputils \
    nixpkgs#lcov \
    nixpkgs#netcat \
    nixpkgs#nix --priority 1 \
    nixpkgs#openssl \
    nixpkgs#python310 \
    nixpkgs#python310Packages.certifi \
    nixpkgs#python310Packages.charset-normalizer \
    nixpkgs#python310Packages.idna \
    nixpkgs#python310Packages.pip \
    nixpkgs#python310Packages.pyyaml \
    nixpkgs#python310Packages.requests \
    nixpkgs#python310Packages.setuptools \
    nixpkgs#python310Packages.six \
    nixpkgs#python310Packages.urllib3 \
    nixpkgs#python310Packages.wheel \
    nixpkgs#sudo \
    nixpkgs#unzip \
    nixpkgs#unzip \
    nixpkgs#which --priority 1 \
    nixpkgs#zip \
    nixpkgs#zlib && \
    ln -s /root/.nix-profile/bin/bazelisk /bin/bazel
ENV PATH="$PATH:/bin"
ENV PYTHONPATH="/root/.nix-profile/lib/python3.10/site-packages"
