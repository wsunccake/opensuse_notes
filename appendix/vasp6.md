# VASP 6 編譯與安裝

介紹如何在 openSUSE 系統上編譯高效能的科學計算軟體 VASP (Vienna Ab initio Simulation Package) 6.4.2 版本，並整合 VASPsol 和 VTST 等常用模組。

## 1. 前置準備：安裝開發工具

在編譯任何原始碼之前，我們需要確保系統具備必要的開發工具，如 C/C++ 與 Fortran 編譯器、`make` 工具等。

- **`zypper in -t pattern devel_basis`**: 在 openSUSE 中，這條指令會安裝一個名為 `devel_basis` 的軟體模式 (pattern)，其中包含了進行軟體開發所需的核心工具集合。

  ```bash
  # 以 root 權限安裝基礎開發工具
  tux@suse16:~ > sudo zypper in -t pattern devel_basis
  ```

## 2. 編譯 VASP 6.4.2

### 2.1. 設定編譯環境

VASP 的效能高度依賴於數學函式庫 (如 BLAS, LAPACK, ScaLAPACK) 和編譯器。使用 Intel oneAPI 工具組，它提供了高效能的 Fortran/C++ 編譯器 (`ifx`, `icpx`) 和數學核心函式庫 (MKL)。

- **`source /opt/intel/oneapi/setvars.sh`**: 這個指令會執行 Intel oneAPI 的環境設定腳本，它會自動設定 `PATH`, `LD_LIBRARY_PATH` 等環境變數，讓系統能找到 Intel 的編譯器和函式庫。

  ```bash
  # 設定 Intel oneAPI 環境變數
  tux@suse16:~ > source /opt/intel/oneapi/setvars.sh
  ```

### 2.2. 解壓縮與設定 Makefile

首先，解壓縮 VASP 原始碼，並根據編譯環境修改 `makefile`。

```bash
# 解壓縮原始碼
tux@suse16:~ > tar zxf vasp.6.4.2.tgz
tux@suse16:~ > cd vasp.6.4.2

# 複製 Intel 編譯器設定檔範本
tux@suse16:~/vasp.6.4.2 > cp arch/makefile.include.intel makefile.include
```

接下來，修改 `makefile.include` 和 `parse/makefile`，將預設的舊版 Intel 編譯器 (`ifort`, `icc`) 更換為新一代的 LLVM-based 編譯器 (`ifx`, `icx`)，以獲得更好的效能與相容性。

`makefile.include`

這檔案定義了主要的編譯器和旗標。

```diff
# arch/makefile.include
- FC         = mpiifort
- FCL        = mpiifort
...
- CC_LIB     = icc
...
- CXX_PARS   = icpc
# 修改後
+ FC         = mpiifx
+ FCL        = mpiifx
...
+ CC_LIB     = icx
...
+ CXX_PARS   = icpx
```

`parse/makefile`

這檔案是 VASP 內部工具的 makefile，也需要一併修改。

```diff
# parse/makefile
- %.o:    %.F90
-         ifort -c $< -o $@
...
- locproj_test:   call_from_fortran.o $(CPPOBJ_PARS) $(COBJ_PARS) locproj.tab.h
-         ifort call_from_fortran.o $(CPPOBJ_PARS) $(COBJ_PARS)  -lstdc++ -o locproj_test
# 修改後
+ %.o:    %.F90
+         ifx -c $< -o $@
...
+ locproj_test:   call_from_fortran.o $(CPPOBJ_PARS) $(COBJ_PARS) locproj.tab.h
+         ifx call_from_fortran.o $(CPPOBJ_PARS) $(COBJ_PARS)  -lstdc++ -o locproj_test
```

### 2.3. 執行編譯

完成設定後，可開始編譯。VASP 提供多種編譯目標 (target)，以生成不同功能的執行檔。

- **`make <target>`**:
  - `std`: 標準版，包含大部分功能。
  - `gam`: Gamma-point only 版本，針對 Gamma 點計算進行優化，計算速度更快，但僅適用於特定體系。
  - `ncl`: Non-collinear 版本，用於處理非共線磁性的計算。
  - `gpu`: 利用 NVIDIA GPU 進行加速的版本。
  - `all`: 編譯所有版本。

```bash
# 編譯標準版 VASP
tux@suse16:~/vasp.6.4.2 > make std

# 編譯完成後，可以在 bin 目錄下找到執行檔
tux@suse16:~/vasp.6.4.2 > ls bin
vasp_std  vasp_gam  vasp_ncl
```

---

## 3. 整合 VASPsol 模組

VASPsol 是一個隱式溶劑模型模組，用於模擬材料在液體環境中的行為。

### 3.1. 下載與修補

首先，從 GitHub 下載 VASPsol 的原始碼，並將其整合進 VASP 的原始碼中。

```bash
# 下載 VASPsol
tux@suse16:~ $ git clone https://github.com/henniggroup/VASPsol.git

# 進入 VASP 原始碼目錄
tux@suse16:~/vasp.6.4.2 >
# 複製 VASPsol 的主要原始碼檔案
tux@suse16:~/vasp.6.4.2 > cp ~/VASPsol/src/solvation.F .
# 應用 patch，使 VASPsol 與 VASP 6.1.0+ 版本相容
tux@suse16:~/vasp.6.4.2 > patch -Np0 < ~/VASPsol/patches/pbz_patch_610
```

### 3.2. 重新編譯

加入新檔案後，需要重新編譯 VASP，讓 VASPsol 的功能被包含進來。

```bash
# 重新編譯標準版
tux@suse16:~/vasp.6.4.2 > make std
```

---

## 4. 整合 VTST 模組

VTST (Vibrational Transition State Theory) Tools 是一套用於尋找過渡態、計算反應能壘的工具，其中最著名的是 NEB (Nudged Elastic Band) 方法。

### 4.1. 下載與整合

首先，下載 VTST 原始碼並將其複製到 VASP 的 `src` 目錄下。

```bash
# 下載並解壓縮 VTST
tux@suse16:~ > tar zxf vtstcode-213.tgz
# 將 VTST 原始碼複製到 VASP 的 src 目錄
tux@suse16:~ > cp -r ~/vtstcode-213/vtstcode6.4.0/* ~/vasp.6.4.2/src/.
```

### 4.2. 修改原始碼與 Makefile

為了讓 VASP 能夠識別並編譯 VTST 的程式碼，需要手動修改幾個檔案。

`src/main.F`

在 `main.F` 中，需要修改 `CHAIN_FORCE` 函式的呼叫參數，加入應力張量 `TSIF`。

```diff
! src/main.F
-      CALL CHAIN_FORCE(T_INFO%NIONS,DYN%POSION,TOTEN,TIFOR, &
-           LATT_CUR%A,LATT_CUR%B,IO%IU6)
...
-      IF (LCHAIN) CALL chain_init( T_INFO, IO)
# 修改後
+      CALL CHAIN_FORCE(T_INFO%NIONS,DYN%POSION,TOTEN,TIFOR, &
+           TSIF,LATT_CUR%A,LATT_CUR%B,IO%IU6)
...
+      CALL chain_init( T_INFO, IO)
```

`src/.objects`

這個檔案列出了所有需要被編譯的物件檔 (`.o`)。我們需要將 VTST 的所有物件檔加入其中。

```diff
# src/.objects
        elf.o \
        hamil_rot.o \
        chain.o \
        dyna.o \
...
# 修改後
        elf.o \
        hamil_rot.o \
+       bfgs.o dynmat.o instanton.o lbfgs.o sd.o cg.o dimer.o bbm.o \
+       fire.o lanczos.o neb.o qm.o \
+       pyamff_fortran/*.o ml_pyamff.o \
+       opt.o \
        chain.o \
        dyna.o \
```

`src/makefile`

最後，告訴 `make` 程式需要額外處理 `pyamff_fortran` 目錄，並將 `libs` 加入依賴項目。

```diff
# src/makefile
- LIB=lib parser
...
- dependencies: sources
# 修改後
+ LIB=lib parser pyamff_fortran
...
+ dependencies: sources libs
```

### 4.3. 重新編譯

完成所有修改後，再次重新編譯 VASP。

```bash
# 重新編譯以包含 VTST
tux@suse16:~/vasp.6.4.2 > make std
```

### 練習

1.  **編譯 Gamma-point 版本**：執行 `make gam`，並比較 `vasp_std` 和 `vasp_gam` 的檔案大小。
2.  **清理編譯檔案**：執行 `make veryclean` 會清除所有編譯生成的 `.o` 和 `mod` 檔案，若要重新編譯，這是一個好習慣。
3.  **檢查連結函式庫**：使用 `ldd bin/vasp_std` 指令，查看 VASP 執行檔連結了哪些動態函式庫，確認是否包含 Intel MKL (`libmkl_*.so`)。

---

## 總結

學習編譯 VASP 6 流程：

1.  **環境準備**：安裝了基礎開發工具並設定了 Intel oneAPI 編譯環境。
2.  **基礎編譯**：修改 `makefile` 以使用新一代的 `ifx` 編譯器，並成功編譯出標準版的 VASP。
3.  **模組整合**：學習了如何將 VASPsol 和 VTST 這兩個重要的功能模組，透過複製原始碼、打補丁 (patch) 和修改 `makefile` 的方式，成功整合進 VASP 中。
