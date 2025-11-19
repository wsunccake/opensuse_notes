# NetCDF4 安裝

在 openSUSE 16 環境下，如何從原始碼編譯與安裝 NetCDF (Network Common Data Form) 函式庫。NetCDF 是一套廣泛應用於科學計算領域的函式庫與資料格式，它允許使用者儲存、讀取、分享陣列導向的科學數據。

NetCDF 的安裝分為兩個主要部分：

1.  **NetCDF-C**: 核心的 C 語言函式庫，提供了最基礎的 API。
2.  **NetCDF-Fortran**: Fortran 語言的介面，它是一個建立在 NetCDF-C 之上的包裝 (wrapper)。

因此，必須先安裝 **NetCDF-C**，然後才能安裝 **NetCDF-Fortran**。將分別介紹如何使用 GNU (開源) 與 Intel (商業) 兩種不同的編譯器工具鏈來完成安裝。

---

## 1. NetCDF-C 安裝

這是安裝的第一步，我們將編譯支援平行 I/O 的 NetCDF-C 函式庫。

### 1.1. GNU (gcc/g++)

- **編譯環境**:

  - C/C++ Compiler: `gcc` / `g++`
  - MPI: OpenMPI

- **編譯步驟**:

```bash
# 設定環境變數
tux@suse16:~ > export NCDIR=$HOME/netcdf-c-gnu

tux@suse16:~ > tar zxf netcdf-c-4.9.3.tar.gz
tux@suse16:~ > cd netcdf-c-4.9.3

# 使用 OpenMPI 編譯器進行設定
tux@suse16:~ > CC=mpicc CXX=mpic++ LIBS=-ldl ./configure \
  --enable-parallel --prefix=$NCDIR
tux@suse16:~ > make
tux@suse16:~ > make install

# 建立 lib 符號連結以增加相容性
tux@suse16:~ > ln -sr $NCDIR/lib64 $NCDIR/lib
```

### 1.2. Intel (icx/icpx)

- **編譯環境**:

  - C/C++ Compiler: `icx` / `icpx`
  - MPI: IntelMPI

- **編譯步驟**:

```bash
# 設定 Intel 環境變數
tux@suse16:~ > export NCDIR=$HOME/netcdf-c-intel
tux@suse16:~ > source /opt/intel/oneapi/setvars.sh

tux@suse16:~ > tar zxf netcdf-c-4.9.3.tar.gz
tux@suse16:~ > cd netcdf-c-4.9.3

# 使用 Intel MPI 編譯器進行設定
tux@suse16:~ > CC=mpiicx CXX=mpiicpx LIBS=-ldl ./configure \
  --enable-parallel --prefix=$NCDIR
tux@suse16:~ > make
tux@suse16:~ > make install

# 建立 lib 符號連結
tux@suse16:~ > ln -sr $NCDIR/lib64 $NCDIR/lib
```

### 1.3. 環境變數設定與驗證 (NetCDF-C)

安裝完成後，設定環境變數讓系統能找到它，並使用 `nc-config` 工具進行驗證。

```bash
tux@suse16:~ > export LD_LIBRARY_PATH=$NCDIR/lib64:$LD_LIBRARY_PATH
tux@suse16:~ > export PATH=$NCDIR/bin:$PATH

# 驗證安裝，此指令會列出所有編譯旗標、函式庫路徑等資訊
tux@suse16:~ > nc-config --all
```

---

## 2. NetCDF-Fortran 安裝

安裝完 NetCDF-C 後，在其基礎上安裝 Fortran 介面。請務必使用與 **NetCDF-C** 一致的編譯器工具鏈。

### 2.1. GNU (gfortran)

- **編譯步驟**:
  - `HDF5_PLUGIN_PATH`: 指定 HDF5 插件路徑，通常可以從 `nc-config --plugindir` 獲得。

```bash
# 設定環境變數
tux@suse16:~ > export NFDIR=$HOME/netcdf-fortran-gnu

# 以下指令提示如何取得 configure 所需的路徑資訊
tux@suse16:~ > nc-config --plugindir
tux@suse16:~ > nc-config --libs

tux@suse16:~ > tar zxf netcdf-fortran-4.6.2.tar.gz
tux@suse16:~ > cd netcdf-fortran-4.6.2

# 執行 configure，連結到先前安裝的 NetCDF-C
tux@suse16:~ > CC=mpicc FC=mpif90 F77=mpif77 \
  LDFLAGS=-L$NCDIR/lib CPPFLAGS="-I$NCDIR/include" \
  LIBS="-L$NCDIR/lib64 -lnetcdf" \
  HDF5_PLUGIN_PATH=$NCDIR/hdf5/lib/plugin \
  ./configure --prefix=$NFDIR
tux@suse16:~ > make
tux@suse16:~ > make install

# 建立 lib 符號連結
tux@suse16:~ > ln -sr $NFDIR/lib64 $NFDIR/lib
```

### 2.2. Intel (ifx)

編譯流程與 GNU 類似，但需確保所有環境變數 (`NCDIR`)、編譯器 (`mpiicx`, `mpiifx`) 與函式庫都指向先前用 Intel 工具鏈編譯的版本。

```bash
# 設定 Intel 環境
tux@suse16:~ > export NFDIR=$HOME/netcdf-fortran-intel
tux@suse16:~ > source /opt/intel/oneapi/setvars.sh

# 提示: nc-config 會從 Intel 編譯的 NetCDF-C 安裝中取得路徑
tux@suse16:~ > nc-config --plugindir
tux@suse16:~ > nc-config --libs

tux@suse16:~ > tar zxf netcdf-fortran-4.6.2.tar.gz
tux@suse16:~ > cd netcdf-fortran-4.6.2

# 使用 Intel MPI 編譯器進行設定
tux@suse16:~ > CC=mpiicx FC=mpiifx F77=mpiifx \
  LDFLAGS=-L$NCDIR/lib CPPFLAGS="-I$NCDIR/include" \
  LIBS="-L$NCDIR/lib64 -lnetcdf" \
  HDF5_PLUGIN_PATH=$NCDIR/hdf5/lib/plugin \
  ./configure --prefix=$NFDIR
tux@suse16:~ > make
tux@suse16:~ > make install

# 建立 lib 符號連結
tux@suse16:~ > ln -sr $NFDIR/lib64 $NFDIR/lib
```

### 2.3. 環境變數設定與驗證 (NetCDF-Fortran)

與 NetCDF-C 類似，設定環境變數並使用 `nf-config` 工具進行驗證。

```bash
tux@suse16:~ > export LD_LIBRARY_PATH=$NFDIR/lib64:$LD_LIBRARY_PATH
tux@suse16:~ > export PATH=$NFDIR/bin:$PATH

# 驗證 Fortran 函式庫安裝
tux@suse16:~ > nf-config --all
```

---

## 練習

1.  **比較設定工具**：分別執行 `nc-config --all` 與 `nf-config --all`，比較兩者的輸出有何異同。
2.  **解釋旗標意義**：說明為何在 `configure` `netcdf-fortran` 時，`LDFLAGS` 和 `CPPFLAGS` 是不可或缺的？
3.  **尋找範例程式**：在 `netcdf-fortran` 的原始碼目錄中，尋找 `examples` 資料夾，並嘗試使用 `nf-config` 提供的旗標來手動編譯其中一個範例程式。例如：`mpif90 simple_xy_wr.f90 $(nf-config --fflags) $(nf-config --flibs)`。
4.  **自動化腳本**：撰寫一個 Shell 腳本，自動完成下載、解壓縮、編譯及安裝 NetCDF-C 和 NetCDF-Fortran (GNU 版本) 的所有步驟。

---

## 總結

成功安裝 NetCDF 的關鍵在於理解其 C 與 Fortran 函式庫的依賴關係，並在編譯時保持工具鏈的一致性。

1.  **先裝 C，再裝 Fortran**：永遠先編譯與安裝 `netcdf-c`，再編譯 `netcdf-fortran`。
2.  **保持工具鏈一致**：若 `netcdf-c` 使用 GNU 編譯器，`netcdf-fortran` 也必須使用 GNU 編譯器。
3.  **善用 `nc-config`**：`nc-config` 是 `netcdf-c` 安裝後的重要工具，它提供了編譯 `netcdf-fortran` 或其他相依軟體時所需的各種路徑與旗標。
4.  **明確指定路徑**：在 `configure` `netcdf-fortran` 時，務必使用 `LDFLAGS` 和 `CPPFLAGS` 明確指向 `netcdf-c` 的安裝位置。
