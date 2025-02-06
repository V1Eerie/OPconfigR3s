# OPconfigR3s

## Self Compilation Guide

### 1. Clone Repositories
```bash
git clone https://github.com/QiuSimons/YAOF.git
cd YAOF
git clone https://github.com/V1Eerie/OPconfigR3s.git
cp ./OPconfigR3s/S* ./
```

### 2. Initialize Build Dependencies
```bash
sudo -E apt-get -qq update
sudo /bin/bash -c "$(curl -sL https://git.io/vokNn)"
sudo -E apt-fast -y -qq install asciidoc bash bcc bin86 binutils bison bzip2 clang-15 llvm-15 clang llvm file flex g++ g++-multilib gawk gcc gcc-multilib gettext git gzip help2man intltool libboost-dev libelf-dev libncurses-dev libncurses5-dev libssl-dev libthread-queue-any-perl libusb-dev libxml-parser-perl make patch perl-modules python3-dev python3-pip python3-pyelftools python3-setuptools rsync sharutils swig time unzip util-linux wget xsltproc zlib1g-dev zip zstd
sudo -E apt-fast -y -qq install dos2unix dwarves quilt npm jq
sudo -E npm install -g pnpm
pip3 install --user -U pylibfdt --break-system-packages
sudo -E apt-get -qq autoremove --purge
sudo -E apt-get -qq clean
sudo -E git config --global user.name 'GitHub Actions' && git config --global user.email 'noreply@github.com'
sudo -E git config --global core.abbrev auto
```

### 3. Prepare Mixedwrt
```bash
cp -r ./SCRIPTS/R3S/. ./SCRIPTS/
cp -r ./SCRIPTS/. ./
/bin/bash 01_get_ready.sh
```

### 4. Prepare Package
```bash
cd openwrt
cp -r ../SCRIPTS/. ./
/bin/bash 02_prepare_package.sh
/bin/bash 02_target_only.sh
/bin/bash 04_remove_upx.sh
cp -rf ../SEED/R3S/config.seed .config
/bin/bash 03_convert_translation.sh
/bin/bash 05_create_acl_for_luci.sh -a
sudo -E chmod -R 755 ./08_fix_permissions.sh
/bin/bash 08_fix_permissions.sh
```

### 5. Prepare for Compilation
```bash
make defconfig
TARGET_DEVICE_ARCH="$(grep "^CONFIG_TARGET_.*_.*=y$" ".config" | head -n 1 | sed 's/^CONFIG_TARGET_//g' | awk -F '_' '{print $1}')"
echo "TARGET_DEVICE_ARCH=${TARGET_DEVICE_ARCH}-3566" >>$GITHUB_ENV
latest_release="$(curl -s https://github.com/openwrt/openwrt/tags | grep -Eo "v[0-9\.]+\-*r*c*[0-9]*.tar.gz" | sed -n '/[2-9][4-9]/p' | sed -n 1p | sed 's/.tar.gz//g' | sed 's/v//g')"
echo "latest_release=${latest_release}" >>$GITHUB_ENV
make download -j50
```

### 6. Compilation
```bash
make -j$(($(nproc) + 1)) package/network/utils/nftables/refresh V=s
IGNORE_ERRORS=1 make -j$(($(nproc) + 1))
```

### Completion
Output file will be located at:
```bash
  YAOF/openwrt/bin/targets/rockchip/armv8/
    openwrt-rockchip-armv8-friendlyarm_nanopi-r3s-ext4-sysupgrade.img.gz
```

### ps1: Additional fix for WSL-Ubuntu24
```bash
ln -s /usr/bin/python3 /usr/bin/python3.8
sudo apt install libpcap-dev dwarves cmake
sudo apt install clang-15 llvm-15 gcc make git npm
```
In **EVERY** new terminal session, run it to fix PATH.
```bash
export PATH=$(echo "$PATH" | tr ':' '\n' | grep -v '^/mnt/c/' | tr '\n' ':' | sed 's/:$//')
```
