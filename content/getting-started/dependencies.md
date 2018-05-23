---
title: "Dependencies"
date: 2018-05-12T21:14:19-04:00
draft: false
weight: 10
---

Building gem5 requires the following components to be installed. Ubuntu 16.04 and newer includes the required versions:

  - git
  - g++ or clang
  - scons
  - python
  - swig
  - protbuf

```bash
sudo apt install git build-essential scons python-dev swig 
sudo apt install libprotobuf-dev python-protobuf protobuf-compiler libgoogle-perftools-dev
```

{{% notice note %}}
While gem5 runs on Linux and Mac OS X, these instructions are written assuming Ubuntu 16.04 or newer. For installing the
required dependencies for other platforms see [Dependencies](/docs/dependencies/).
{{% /notice %}}



