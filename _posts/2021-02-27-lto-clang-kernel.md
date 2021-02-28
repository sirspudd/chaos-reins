---
title:  "Clang LTO enablement"
date:   2021-02-26 12:10:59 -0700
published: true
tags: [clang, llvm, linux, oss, lto]
---

# TL&DR

make LLVM=1 LLVM_IAS=1

# Enabling the clang LTO functionality in the kernel

My daily driving kernel is the HEAD of mainline built with CC=clang since I became aware of this being possible some point in the middle of 2020. Having seen the clang lto stuff land, I decided to attempt to consume it and squandered time unecessarily cluing myself up from first principles instead of reading the damn documentation.

I had historically been using:

```
make CC=clang HOSTCC=clang
```

needs to become

```
make LLVM=1 LLVM_IAS=1
```

in order for the clang lto options to be populated in menuconfig.

This is well covered in the kernel [llvm](https://www.kernel.org/doc/html/latest/kbuild/llvm.html) when you eventually find them. Google did not actually bring them to bare until I stubbed my toe on the page in the tree.

Having changed my make calls as indicated above, I could enable:

```
+# CONFIG_STACKPROTECTOR is not set
+CONFIG_LTO=y
+CONFIG_LTO_CLANG=y
+CONFIG_ARCH_SUPPORTS_LTO_CLANG=y
+CONFIG_ARCH_SUPPORTS_LTO_CLANG_THIN=y
+CONFIG_HAS_LTO_CLANG=y
+# CONFIG_LTO_NONE is not set
+CONFIG_LTO_CLANG_FULL=y
+# CONFIG_LTO_CLANG_THIN is not set
```

and it compiled out the can and booted to a functional system, generating a kernel package 4megs smaller than the one it replaced.

Because I was enabling this in my pkgbuild personal kernel recipe, and because I overzealously enabled -OZ at the same time, I was missattributing my woes to the clang LTO pathway; the fine peeps on ClangBuiltLinux pointed out that I was almost certainly being boned by upstream build failure, and now I roll with:

```
build() {
  set -o nounset
  set -o pipefail
  set -o errexit
  #set -o xtrace

  cd $_kernel_src_dir && git clean -xdf

  if [[ -n ${_config_preset} ]]; then
    echo ${_make_cmd} ${_config_preset}
    ${_make_cmd} ${_config_preset}
  elif [[ -n ${_config_file} ]]; then
    cp ${_config_file} ${_kernel_src_dir}/.config
  else
    echo "Kernel not configured; bailing"
    exit 1
  fi

  ${_make_cmd} all 2>&1 | tee ${log_path}

  if [ -n "${_pi_ver}" ]; then
    ${_make_cmd} dtbs 2>&1 | tee -a ${log_path}
  fi

  set +o nounset
  set +o pipefail
  set +o errexit
}
```

In order to cease sausage production at the loss of the first employee to the mangler.
