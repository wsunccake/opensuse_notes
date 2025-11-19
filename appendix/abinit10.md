# Abinit 10 編譯與安裝

在 openSUSE 16 環境下，從原始碼編譯與安裝 Abinit 10 的完整過程。Abinit 是一套強大的開源軟體，主要用於材料科學領域，透過第一原理計算來預測固體、分子等材料的物理與化學性質。

## 1. 安裝前置相依套件

在開始編譯 Abinit 之前，我們需要先安裝一系列必要的開發工具與函式庫。這些套件提供了編譯器、函式庫以及 Abinit 執行所需的底層支援。

- **`devel_basis`**: 安裝基礎開發工具集合 (pattern)。這包含了如 `gcc`, `make` 等核心編譯工具，是進行原始碼編譯的基礎。
- **`gfortran`**: 安裝 GNU Fortran 編譯器。Abinit 大量使用 Fortran 語言開發，因此 Fortran 編譯器是必需的。
- **`python313-base python313-devel`**: 安裝 Python 3.13 的執行環境與開發檔案。Abinit 的部分腳本與工具會使用到 Python。
- **`hdf5 hdf5-devel`**: 安裝 HDF5 (Hierarchical Data Format 5) 函式庫及其開發檔案。HDF5 用於高效地儲存與管理大量科學數據，例如波函數、電荷密度等。
- **`libxc12 libxc-devel`**: 安裝 Libxc 函式庫及其開發檔案。Libxc 是一個專門計算交換相關泛函 (exchange-correlation functionals) 的函式庫，這是密度泛函理論 (DFT) 計算的核心部分。
- **`openblas-common-devel`**: 安裝 OpenBLAS (Open Basic Linear Algebra Subprograms) 的開發檔案。OpenBLAS 提供了高效能的線性代數運算支援，對於矩陣運算等科學計算至關重要。

```bash
tux@suse16:~ > sudo zypper in -t pattern devel_basis
tux@suse16:~ > sudo zypper in gfortran
tux@suse16:~ > sudo zypper in python313-base python313-devel
tux@suse16:~ > sudo zypper in hdf5 hdf5-devel
tux@suse16:~ > sudo zypper in libxc12 libxc-devel
tux@suse16:~ > sudo zypper in openblas-common-devel
```

- [netcdf](./netcdf4.md)

## 2. 從原始碼編譯 Abinit

安裝完所有相依套件後，接下來我們將從原始碼開始編譯 Abinit。

### 2.1. 編譯環境說明

本次編譯將基於以下 GNU 工具鏈與函式庫：

- **C Compiler**: `gcc`
- **C++ Compiler**: `g++`
- **Fortran Compiler**: `gfortran`
- **MPI (Message Passing Interface)**: `openmpi 5.x`，用於平行計算。
- **Linear Algebra / 線性代數函式庫**: `openblas`
- **FFT (Fast Fourier Transform) 函式庫**: `goedecker` (Abinit 內建)
- **NetCDF**: `netcdf`, `netcdf-fortran`

### 2.2. 編譯步驟

完成編譯過程。

```bash
tux@suse16:~ > export ABINITDIR=$HOME/abinit

tux@suse16:~ > tar zxf abinit-10.4.7.tar.gz
tux@suse16:~ > cd abinit-10.4.7
tux@suse16:~/abinit-10.4.7 > mkdir build
tux@suse16:~/abinit-10.4.7 > cd build

tux@suse16:~/abinit-10.4.7/build > cp ../doc/build/config-template.ac9 $HOSTNAME.ac9
tux@suse16:~/abinit-10.4.7/build > vi $HOSTNAME.ac9
tux@suse16:~/abinit-10.4.7/build > ./configure --with-config-file=$HOSTNAME.ac9 --prefix=$HOME/abinit
tux@suse16:~/abinit-10.4.7/build > make
```

### 2.3. 編譯設定檔 (`$HOSTNAME.ac9`)

這檔案是 `configure` 腳本的輸入，用來告知編譯系統要使用哪些編譯器、函式庫與功能。

```ini
# $HOSTNAME.ac9

# 設定安裝路徑，對應 configure 的 --prefix 參數
prefix="$ABINITDIR"

# 指定用於平行計算的 MPI 編譯器
CC="mpicc"
CXX="mpic++"
FC="mpif90"
F77="mpif77"

# 自動偵測 MPI 的類型 (flavor)
with_mpi_flavor="auto"

# 指定 NetCDF 函式庫的路徑 (如果安裝在非標準路徑)
# with_netcdf="$NCDIR"
# with_netcdf_fortran="$NFDIR"
```

**注意**: 在這個範例中，`with_netcdf` 和 `with_netcdf_fortran` 被註解掉了。如果 HDF5 或 NetCDF 函式庫是手動安裝在特殊路徑，需要取消註解並將 `$NCDIR` 和 `$NFDIR` 替換為實際的路徑。

## 3. 設定環境變數

編譯並安裝完成後，為了讓系統能夠找到 Abinit 的執行檔與動態函式庫，需要設定以下兩個環境變數。

- **`LD_LIBRARY_PATH`**: 將 Abinit 安裝目錄下的 `lib64` (或 `lib`) 路徑加入到動態函式庫的搜尋路徑中。
- **`PATH`**: 將 Abinit 安裝目錄下的 `bin` 路徑加入到執行檔的搜尋路徑中，就可以在任何地方直接輸入 `abinit` 來執行程式。

```bash
tux@suse16:~ > export LD_LIBRARY_PATH=$ABINITDIR/lib64:$LD_LIBRARY_PATH
tux@suse16:~ > export PATH=$ABINITDIR/bin:$PATH
```

為了讓設定永久生效，可將這兩行指令加入到 shell 設定檔中，例如 `~/.bashrc` 或 `~/.zshrc`，然後重新登入或執行 `source ~/.bashrc`。

---

## 練習

1.  **驗證安裝**：在終端機中執行 `abinit --version`，檢查是否能成功輸出版本資訊。
2.  **尋找函式庫**：使用 `find /usr -name "libxc.so*"` 指令，找出 `libxc` 動態函式庫實際被安裝在哪個路徑。
3.  **永久化環境變數**：將 `PATH` 和 `LD_LIBRARY_PATH` 的 `export` 指令加入到您的 `~/.bashrc` 檔案中，並開啟一個新的終端機視窗，確認 `abinit` 指令是否可以直接執行。
4.  **檢查連結的函式庫**：執行 `ldd $(which abinit)` 指令，觀察 Abinit 執行檔連結了哪些動態函式庫，確認是否包含了 `libmpi.so`, `libxc.so` 等。

---

## 總結

在 openSUSE 系統上從頭開始安裝 Abinit 的完整流程。關鍵步驟包括：

1.  **安裝相依套件**：使用 `zypper` 安 строю 所需的工具與函式庫。
2.  **設定與編譯**：下載原始碼，建立獨立的 `build` 目錄，並透過客製化的 `.ac9` 設定檔來配置編譯選項。
3.  **環境設定**：更新 `PATH` 與 `LD_LIBRARY_PATH` 環境變數，確保系統能正確找到 Abinit 程式。
