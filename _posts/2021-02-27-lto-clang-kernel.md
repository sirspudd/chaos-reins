---
title:  "Clang LTO enablement"
date:   2021-02-26 12:10:59 -0700
published: true
tags: [clang, llvm, linux, oss, lto]
---

My daily driving kernel is the HEAD of mainline built with CC=clang since I became aware of this being possible some point in the middle of 2020. Having seen the clang lto stuff land, I decided to attempt to consume it and squandered time unecessarily cluing myself up from first principles instead of reading the damn documentation.

The TL&DR is:

```
make CLANG=1
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
