# OpenMPI 5 安裝與設定

在 openSUSE 16 系統上安裝與設定 OpenMPI 5 的過程。OpenMPI 是一個高品質、開源的訊息傳遞介面 (Message Passing Interface, MPI) 實作，廣泛應用於高效能與平行運算領域。

## 1. 安裝 OpenMPI 5

使用 openSUSE 的套件管理器 `zypper` 來安裝 OpenMPI 5 相關套件。

- **`openmpi5`**: 包含了執行 MPI 程式所需的核心執行環境 (runtime) 與函式庫。
- **`openmpi5-devel`**: 包含了編譯 MPI 程式所需的開發檔案，例如標頭檔 (`mpi.h`) 與編譯器包裝 (wrapper) 工具。

```bash
tux@suse16:~ > sudo zypper in openmpi5 openmpi5-devel
```

## 2. 設定預設 MPI 環境

在一個系統上可能同時安裝了多個 MPI 的實作版本 (例如 OpenMPI, MPICH 等)。`mpi-selector` 工具可以幫助我們輕鬆地在這些版本之間切換，設定使用者或系統層級的預設 MPI 環境。

- **`--list`**: 列出目前系統上所有可用的 MPI 實作版本。
- **`--user --set openmpi5`**: 將 `openmpi5` 設定為當前使用者的預設 MPI 環境。這會建立相關的符號連結 (symbolic links)，讓 `mpicc` 等指令指向 OpenMPI 5 的版本。若要進行全系統設定，可將 `--user` 替換為 `--system` (需要 `root` 權限)。
- **`--query`**: 查詢目前生效的預設 MPI 版本，用以確認設定是否成功。

```bash
tux@suse16:~ > mpi-selector --list
tux@suse16:~ > mpi-selector --user --set openmpi5
tux@suse16:~ > mpi-selector --query
```

#### 互動式選單

也可透過互動式選單來進行設定。

- **`mpi-selector-menu`**: 啟動一個文字使用者介面 (TUI)，您可以在其中透過方向鍵與 Enter 鍵來選擇並設定預設的 MPI 版本。這對於初學者來說更加直觀。

```bash
# 執行後會進入一個互動式設定畫面
tux@suse16:~ > mpi-selector-menu
```

## 3. 驗證與使用

設定完成後，我們可以透過 `which` 指令來驗證系統是否能正確找到 OpenMPI 5 的相關執行檔。這些指令應該要能回傳位於 OpenMPI 5 安裝路徑下的執行檔路徑。

- **`which mpirun`**: 檢查 MPI 程式執行指令的路徑。
- **`which mpicc`**: 檢查 C 語言的 MPI 編譯器路徑。
- **`which mpic++`**: 檢查 C++ 語言的 MPI 編譯器路徑。
- **`which mpif77` / `which mpif90`**: 檢查 Fortran 77 / 90 語言的 MPI 編譯器路徑。

```bash
tux@suse16:~ > which mpirun
tux@suse16:~ > which mpicc
tux@suse16:~ > which mpic++
tux@suse16:~ > which mpif77
tux@suse16:~ > which mpif90
```

---

## 練習

1.  **編譯並執行 MPI 程式**：

    - 建立一個名為 `hello_mpi.c` 的檔案，內容如下：

      ```c
      #include <mpi.h>
      #include <stdio.h>

      int main(int argc, char** argv) {
          MPI_Init(NULL, NULL);
          int world_size;
          MPI_Comm_size(MPI_COMM_WORLD, &world_size);
          int world_rank;
          MPI_Comm_rank(MPI_COMM_WORLD, &world_rank);
          printf("Hello from processor %d out of %d\n", world_rank, world_size);
          MPI_Finalize();
          return 0;
      }
      ```

    - 使用 `mpicc -o hello_mpi hello_mpi.c` 編譯程式。
    - 使用 `mpirun -np 4 ./hello_mpi` 執行程式，觀察 4 個處理程序輸出的訊息。

2.  **探索互動介面**：執行 `mpi-selector-menu`，熟悉如何在不同 MPI 版本之間切換。

3.  **查看編譯器版本**：執行 `mpicc --version`，確認其顯示的資訊是否與 `gcc` 及 OpenMPI 相關。

---

## 總結

在 openSUSE 上設定 OpenMPI 環境的流程相當簡潔：

1.  **安裝**：使用 `zypper` 安裝 `openmpi5` 與 `openmpi5-devel` 套件。
2.  **設定**：使用 `mpi-selector` 工具指定 `openmpi5` 為預設的 MPI 環境。
3.  **驗證**：透過 `which` 指令確認 `mpirun`、`mpicc` 等指令已正確連結。

完成以上步驟後，就可開始編譯與執行您的 MPI 平行程式了。
