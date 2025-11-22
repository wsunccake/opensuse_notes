# LAMMPS 編譯與安裝

在 openSUSE 16 環境下，使用 Intel oneAPI 編譯器來編譯 LAMMPS (Large-scale Atomic/Molecular Massively Parallel Simulator)。LAMMPS 是一款功能強大的分子動力學模擬軟體，廣泛應用於材料科學、化學、物理學等領域。

## 1. 安裝步驟

### 1-1. 環境準備

在編譯之前，須設定 Intel oneAPI 編譯環境。Intel oneAPI 提供了一整套高效能的開發工具，包括 C++ 編譯器、MPI 函式庫等，非常適合用來編譯科學計算軟體。

- **`/opt/intel/oneapi/setvars.sh`**: 此指令用於初始化 Intel oneAPI 的環境變數。執行後，系統才能找到 `mpiicpx` 等編譯指令以及相關的函式庫。

  ```bash
  tux@suse16:~> source /opt/intel/oneapi/setvars.sh
  :: initializing oneAPI environment ...
     -bash: BASH_VERSION = 5.2.26(1)-release
     :: oneAPI environment initialized ::
  ```

### 1-2. 原始碼下載與解壓縮

接著，我們需要取得 LAMMPS 的原始碼並解壓縮。

- **`tar zxf`**: 這指令會將 `lammps-stable.tar.gz` 這個壓縮檔解開。`z` 表示處理 gzip 格式，`x` 表示解壓縮，`f` 表示從檔案操作。

  ```bash
  tux@suse16:~> tar zxf lammps-stable.tar.gz
  tux@suse16:~> cd lammps-22Jul2025/src/
  ```

  解壓縮後，使用 `cd` 指令進入原始碼的核心目錄 `src`，所有的編譯工作都將在此進行。

### 1-3. 編譯設定

LAMMPS 提供了多種預設的 Makefile，以支援不同的作業系統、編譯器和硬體組合。

- **`tree MAKE/`**: 透過此指令，可以查看 `MAKE` 資料夾下的結構，裡面包含了所有可用的 Makefile 範本。

本次安裝，選用 `Makefile.intel_cpu_intelmpi` 這個針對 Intel CPU 和 Intel MPI 的設定檔。

#### 步驟 1: 備份並修改 Makefile

為了安全起見，先複製一份原始的 Makefile 再進行修改。

- **`cp MAKE/OPTIONS/Makefile.intel_cpu_intelmpi MAKE/OPTIONS/Makefile.intel_cpu_intelmpi.org`**: 將原始檔案備份為 `.org` 結尾的檔案。

接著，根據 oneAPI 的新版本編譯器 `icpx` 來修改設定檔。舊版的 `icpc` 已經被新版的 `icpx` (基於 LLVM) 所取代。

- **修改 `MAKE/OPTIONS/Makefile.intel_cpu_intelmpi`**:
  - 將 `CC = mpiicpc` 改為 `CC = mpiicpx`。
  - 將 `LINK = mpiicpc` 改為 `LINK = mpiicpx`。
  - 從 `CCFLAGS` 中移除 `-qno-offload` 和 `-restrict` 這兩個在新版編譯器中可能不再需要或語法變更的參數。

**修改前**:

```makefile
# MAKE/OPTIONS/Makefile.intel_cpu_intelmpi
CC =            mpiicpc -std=c++11 -diag-disable=10441 -diag-disable=2196 -diag-disable=11074 -diag-disable=11076
OPTFLAGS =      -xHost -O2 -fp-model fast=2 -no-prec-div -qoverride-limits \
                -qopt-zmm-usage=high
CCFLAGS =       -qopenmp -qno-offload -ansi-alias -restrict \
                -DLMP_INTEL_USELRT -DLMP_USE_MKL_RNG $(OPTFLAGS) \
                -I$(MKLROOT)/include
...
LINK =          mpiicpc -std=c++11 -diag-disable=10441 -diag-disable=2196
```

**修改後**:

```makefile
# MAKE/OPTIONS/Makefile.intel_cpu_intelmpi
CC =            mpiicpx -std=c++11 -diag-disable=10441 -diag-disable=2196 -diag-disable=11074 -diag-disable=11076
OPTFLAGS =      -xHost -O2 -fp-model fast=2 -qoverride-limits \
                -qopt-zmm-usage=high
CCFLAGS =       -qopenmp -ansi-alias \
                -DLMP_INTEL_USELRT -DLMP_USE_MKL_RNG $(OPTFLAGS) \
                -I$(MKLROOT)/include
...
LINK =          mpiicpx -std=c++11 -diag-disable=10441 -diag-disable=2196
```

### 1-4. 執行編譯

設定完成後，就可開始編譯。

- **`make intel_cpu_intelmpi`**: `make` 會根據剛剛修改的 `Makefile.intel_cpu_intelmpi` 檔案，啟動編譯程序。這個過程會需要一些時間，具體長度取決於機器的效能。

  ```bash
  tux@suse16:~/lammps-22Jul2025/src > make intel_cpu_intelmpi
  ... (編譯過程訊息) ...
  ```

編譯成功後，會在 `src` 目錄下產生一個以 `lmp_` 開頭的執行檔。

- **`ls lmp_*`**: 使用此指令可以找到編譯好的 LAMMPS 執行檔。

  ```bash
  tux@suse16:~/lammps-22Jul2025/src > ls lmp_*
  lmp_intel_cpu_intelmpi
  ```

  這個 `lmp_intel_cpu_intelmpi` 就是我們可以使用的 LAMMPS 主程式。

---

## 練習

1.  **研究編譯參數**：查閱 Intel 編譯器文件，了解 `-xHost` 和 `-O2` 這兩個 `OPTFLAGS` 參數的具體作用是什麼？
2.  **嘗試不同編譯目標**：在 `MAKE/` 目錄下找一個 `Makefile.serial`，嘗試編譯一個非 MPI 版本的 LAMMPS。使用哪個 `make` 指令？
3.  **清理編譯檔案**：在 `make` 的世界裡，通常會有一個 `clean` 或 `distclean` 的目標，用來清除編譯過程中產生的暫存檔。請在 LAMMPS 的 Makefile 中找找看，並嘗試執行它。

---

## 總結

學習如何在 openSUSE 系統上，從原始碼編譯 LAMMPS。重點在於選擇並修改適合的 Makefile，特別是將編譯器從舊的 `mpiicpc` 更新為新的 `mpiicpx`。透過這個過程，不僅安裝了 LAMMPS，也對其編譯系統有了更深入的了解。正確的編譯設定是發揮硬體最大效能的關鍵。
