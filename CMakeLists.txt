  build-linux:
    name: Build Server (Linux - 32-bit Compatible)
    runs-on: ubuntu-22.04
    env:
      CC: gcc-9
      CXX: g++-9
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        with:
          submodules: recursive

      - name: Install dependencies (with 32-bit support)
        run: |
          sudo apt-get update -y
          sudo apt-get install -y build-essential gcc-9 g++-9 gcc-9-multilib g++-9-multilib libc6-dev-i386

      - name: Install Conan
        run: pip3 install "conan<2.0"

      - name: Configure and Build (32-bit)
        run: |
          cmake -B build \
            -DCMAKE_BUILD_TYPE=Release \
            -DBUILD_CLIENT=OFF \
            -DCMAKE_C_FLAGS="-m32" \
            -DCMAKE_CXX_FLAGS="-m32 -fpermissive -I$PWD/server/third-party/omp-sdk/include" \
            -DCMAKE_SHARED_LINKER_FLAGS="-m32"
          cmake --build build --config Release --parallel

      - name: Verify Architecture
        run: |
          file build/server/*.so
          # Должно быть: ELF 32-bit LSB shared object, Intel 80386

      - name: Check GLIBC compatibility
        run: |
          objdump -T build/server/*.so | grep GLIBC_ | sort -u

      - name: Upload Server Artifact
        uses: actions/upload-artifact@v4
        with:
          name: linux
          path: build/server/*.so
