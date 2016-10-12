1. Get NDK https://developer.android.com/ndk/index.html

2. Get and build openssl (folow https://wiki.openssl.org/index.php/Android)


	wget https://www.openssl.org/source/openssl-1.0.1t.tar.gz
	wget https://wiki.openssl.org/images/7/70/Setenv-android.sh
	/bin/bash
	ANDROID_NDK_ROOT=${ANDROID_NDK} source Setenv-android.sh
	./config no-rc2 no-rc4 no-rc5 no-idea no-camellia no-des no-cast no-md2 no-md4 no-ripemd no-mdc2 no-whirlpool no-patents no-dsa no-dh no-asm no-ssl2 no-ssl3 no-srp no-cms no-ec2m no-jpake no-ec_nistp_64_gcc_128 no-weak-ssl-ciphers no-sock no-krb5 no-ec no-ecdsa no-ecdh no-gost no-engine no-hw no-rsax no-sctp no-srtp no-rfc3779 no-montasm no-shared no-store no-unit-test no-zlib no-zlib-dynamic no-comp --openssldir=$(pwd)/openssl
	make depend
	make build_libs
	mkdir lib
	cp libcrypto.a libssl.a lib

3. fuse

Make standalone toolchain

	{ANDROID_NDK}/build/tools/make-standalone-toolchain.sh --arch=arm --ndk-dir=${ANDROID_NDK} --install-dir=${ANDROID_TOOLCHAIN} --platform=android-18 --system=linux-x86_64
	export PATH=${ANDROID_TOOLCHAIN}/bin:$PATH

	Patch fuse for bionic (absent some functionality like pthread_cancel
	and st_mtim in sys/time.h)
	
	Configure and build fuse

	CFLAGS="-D__ANDROID__" ./configure --disable-shared --enable-static --host=arm-linux-androideabi --with-sysroot=/home/hoxnox/devel/android/toolchain/sysroot --disable-shared --enable-static --disable-example --disable-util --prefix=$(pwd)/libfuse make install

4. encfs

	Prepare toolchain file
	Patch easyloging++, FindFUSE.cmake, CMakeLists.txt
	LDFLAGS="-fPIE -pie" CFLAGS="-D__ANDROID__ -D BUILD_NLS=0 -fPIE -fPIC" cmake -DCMAKE_TOOLCHAIN_FILE=../../toolchain.cmake -DFUSE_USE_STATIC_LIBS=True -DCMAKE_INSTALL_PREFIX=$(pwd)/encfs_distr -DCMAKE_BUILD_TYPE=Release ..
	make install

2. Prepare toolchain (see NDK programmers guide, chapter Standalone toolchan) and change compiler's environment

	{ANDROID_NDK}/build/tools/make-standalone-toolchain.sh --arch=arm --ndk-dir=${ANDROID_NDK} --install-dir=${ANDROID_TOOLCHAIN} --platform=android-21 --system=linux-x86_64

	export PATH=${ANDROID_TOOLCHAIN}/bin:$PATH
	export CC=arm-linux-androideabi-gcc
	export CXX=arm-linux-androideabi-g++
	export CFLAGS="-DNDEBUG -fno-rtti -fno-exceptions -g0 -O2"
3. 

	export PATH=${ANDROID_TOOLCHAIN}/bin:$PATH
	export CC=arm-linux-androideabi-gcc
	export CXX=arm-linux-androideabi-g++
	export CFLAGS="-DNDEBUG -fno-rtti -fno-exceptions -g0 -O2"
