goandroid
=========

Patches to the Go tools and runtime to enable Android apps to interface directly with a shared library written in Go. Goandroid also includes a simple demo written in Go, showing off OpenGL ES 2 and touch input.

Running [Go](http://golang.org) code from Android apps is currently not possible, because the Go tools can only output executables while Android requires any foreign code in shared library (.so) format. This repository contains patches for the Go tools and runtime to enable shared library output, including workarounds to Android specific limitations. It also includes a simple example app written in Go to demonstrate OpenGL ES 2 graphics and touch input.

*Disclaimer*: Please note that the patches are unofficial, and while I hope the Go developers will eventually incorporate at least the patches not specific to Android, there's no guarantee they will.

### Set up ###

This guide is tested on linux/amd64 and assumes you have an android device connected through USB and that you meet the requirements for [building Go from source](http://golang.org/doc/install/source).

1. Download and install the [NDK](http://developer.android.com/tools/sdk/ndk/index.html) at least version r8e. These instructions assumes the NDK is installed in `$NDK`.
2. Create a standalone NDK toolchain (as described in $NDK/docs/STANDALONE-TOOLCHAIN.html):

	`$NDK/build/tools/make-standalone-toolchain.sh --platform=android-9 --install-dir=ndk-toolchain`

	You might need to add `--system=linux-x86_64` or `--system=darwin-x86_64` depending on your system.

	Set `$NDK_TOOLCHAIN` to point at the `ndk-toolchain` directory

3. Clone the golang repository:

	`hg clone https://code.google.com/p/go`

4. Copy the `patches` directory  to the `go/.hg` directory:

	`cp -a patches go/.hg`

5. Enable the `mq` extension by adding the following lines to `go/.hg/hgrc`:

	```
	[extensions]  
	mq =
	codereview = !

	[ui]  
	username = me<me@mail.com>
	```

6. In the `go/src` directory apply the patches and build go:

	```
	cd go/src  
	hg qpush -a  
	CGO_ENABLED=0 GOOS=linux GOARCH=arm ./make.bash  
	CC="$NDK_TOOLCHAIN/bin/arm-linux-androideabi-gcc" GOOS=linux GOARCH=arm GOARM=7 CGO_ENABLED=1 ../bin/go install -tags android -a -v std  
	cd ../..
	```

### Building and installing the example app ###

If everything is set up correctly, you should be able to run `build.sh` in the goandroid root to build and copy `libgoandroid.so` to android/libs. Then, running `ant -f android/build.xml clean debug install` will build and install the final apk to the connected device. Running the app should display a simple color animated triangle that you can move around the screen with your finger.

### Go patches ###

All patches except `android-tls` and `android-build-hacks` correspond to the patches for linux/arm external linking and shared library support discussed on the [golang-nuts mailing list](https://groups.google.com/d/msg/golang-nuts/zmjXkGrEx6Q/L4R8qyw7WW4J).

The `android-tls` patch is a workaround for the missing support for the `R_ARM_TLS_IE32` relocation in the Android linker.

The `android-build-hacks` patch contains various changes to account for the difference between a vanilla linux/arm system and Android.
