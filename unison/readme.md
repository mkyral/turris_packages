# Prerequisites

* working gcc ;-)

## Step 0 - Preparations

* Create working folder and copy there patch files.


```shell
mkdir turris
cd turris/
export TURRIS=$PWD
```

## Step 1 - Download openwrt repo, patch and compile

```shell
cd $TURRIS

git clone --depth 1 https://gitlab.labs.nic.cz/turris/openwrt.git
cd openwrt/

# For Turris 1.x
cp -f configs/turris .config

# For Turris Omnia
cp -f configs/omnia .config

# update config
make defconfig
```

* Add ocaml and unison packages

```
git apply --whitespace=nowarn ../0001-toolchain-add-OCaml-compiler.patch
git apply --whitespace=nowarn ../0002-package-Add-Unison-File-Synchronizer.patch
```

# update config file again

```
make defconfig
```

* Enable ocaml compilation

```shell
echo "
CONFIG_DEVEL=y
CONFIG_TOOLCHAINOPTS=y
CONFIG_OCAML=y
" >> .config

make defconfig
```


* Compile tools and libraries ocaml a ncurses

```shell
make toolchain/install
make package/ncurses/install
```

**Note**:
If you have multicore CPU, you can use -j<number> parameter to enable parallel compilation.

E.g.:
```shell
make -j4 toolchain/install
make -j4 package/ncurses/install
```

Commands below will use all cores:
```shell
CPUs=$(cat /proc/cpuinfo |grep processor |tail -n 1 |cut -d ":" -f 2); CPUs=$(( CPUs + 2 ))
make -j${CPUs} toolchain/install
make -j${CPUs} package/ncurses/install
```

## Step 2 - final Unison compilation and packaging

* Enable, compile and prepare Unison package
```shell
echo "CONFIG_PACKAGE_unison=m" >>.config
make package/unison/compile
make package/unison/install
```

If all finished correctly, you can found the final package in: `$TURRIS/openwrt/bin/mpc85xx/packages/base/`

e.g.:
```shell
ls -la $TURRIS/openwrt/bin/mpc85xx/packages/base/unison*.ipk
-rw-r--r-- 1 marian users 2189096 12. may 20.27 /mnt/data/turris/openwrt/bin/mpc85xx/packages/base/unison_2.48.3-1_mpc85xx.ipk
```

## Step 3 - installation
* Copy package to Turris and install it using command:
```shell
opkg install /cesta/k/balíčku/unison_2.48.3-1_mpc85xx.ipk
```

* Test unison:
```shell
unison -version
```

* Output should looks like:
```shell
turris ~ # unison -version
unison version 2.48.3
turris ~ #
```
