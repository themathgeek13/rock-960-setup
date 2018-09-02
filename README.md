# rock-960-setup
Setup scripts and instructions for the ROCK960 based on RK3399. I bought this board from Vamrs in Shenzen, via SeeedStudio, for $100.

The board supports full Ubuntu desktop, but the 96boards website will give you Ubuntu Server Edition. Install this first, then install a desktop on top. See different commands [here](https://www.linuxtrainingacademy.com/install-desktop-on-ubuntu-server/).

The board image does not have OpenCL installed out of the box. You can use the [rockchip-linux libmali repository on GitHub](https://www.github.com/rockchip-linux/libmali) and follow these instructions to install. Instructions were borrowed from [here](http://tessy.org/wiki/index.php?Firefly RK3399).

```
$ git clone https://www.github.com/rockchip-linux/libmali
$ cd libmali
$ cmake .
$ make
$ sudo make install
```

Now replace only the libOpenCL.so file as follows. This works for me, YMMV.

```
$ sudo rm /usr/lib/aarch64-linux-gnu/libOpenCL.so
$ sudo cp -p libmali/lib/aarch64-linux-gnu/libmali-midgard-t86x-r13p0-wayland.so /usr/lib/aarch64-linux-gnu/
$ cd /usr/lib/aarch64-linux-gnu/
$ sudo ln -s libmali-midgard-t86x-r13p0.so libOpenCL.so
```

The above commands should actually suffice, but to build it correctly, finish the procedure by running this.

```
sudo apt install opencl-headers ocl-icd-opencl-dev
```

Having completed OpenCL installation, lets check if it works! Following benchmark from [here](https://gist.github.com/mli/585aed2cec0b5178b1a510f9f236afa2).

## Sanity Check with `clpeak`

We can check if the opencl driver works properly by using the benchmark tool `clpeak`

```bash
sudo apt-get update && sudo apt-get install cmake git
git clone https://github.com/krrishnarraj/clpeak && cd clpeak
cmake . -DCMAKE_CXX_COMPILER=g++ && make
./clpeak
```

If everything works well, then you probably will see the following outputs:

```
Platform: ARM Platform
  Device: Mali-T860
    Driver version  : 1.2 (Linux ARM)
    Compute units   : 4
    Clock frequency : 200 MHz

    Global memory bandwidth (GBPS)
      float   : 3.17
      float2  : 6.07
      float4  : 7.88
      float8  : 6.55
      float16 : 6.26

    Single-precision compute (GFLOPS)
      float   : 25.09
      float2  : 45.51
      float4  : 46.22
      float8  : 41.67
      float16 : 46.40

    half-precision compute (GFLOPS)
      half   : 23.11
      half2  : 50.19
      half4  : 98.30
      half8  : 93.48
      half16 : 93.94

    Double-precision compute (GFLOPS)
      double   : 3.59
      double2  : 3.30
      double4  : 20.97
      double8  : 20.65
      double16 : 20.39

    Integer compute (GIOPS)
      int   : 20.15
      int2  : 49.64
      int4  : 47.12
      int8  : 49.17
      int16 : 41.47

    Transfer bandwidth (GBPS)
      enqueueWriteBuffer         : 4.61
      enqueueReadBuffer          : 2.60
      enqueueMapBuffer(for read) : 475.11
        memcpy from mapped ptr   : 2.50
      enqueueUnmap(after write)  : 2790.39
        memcpy to mapped ptr     : 1.92

    Kernel launch latency : 190.64 us
```

Additional RK3399 benchmarks and custom library for deep learning called TVM are available [here](http://lmzheng.net/posts/2018/01/opt-arm-mali).
