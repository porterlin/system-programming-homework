# 系統程式設計作業

這個 repository 收錄我在系統程式設計課程的 12 份作業，全部以 C 語言撰寫，目標平台為 Linux。多數題目來自 Michael Kerrisk 的《The Linux Programming Interface》，所以像 `4-1`、`44-6` 這樣的資料夾名稱，對應的是書中的習題編號。

## 環境需求

- Linux（程式會用到 `/proc`、`inotify`、System V 與 POSIX IPC）
- `gcc` 與 `make`

每個資料夾都有自己的 `makefile`，編譯方式如下：

```bash
cd hwN        # 或 hwN/<子資料夾>
make
make clean    # 只有部分資料夾提供
```

## 總覽

| 作業 | 主題 | 內容 |
| --- | --- | --- |
| hw1 | 檔案 I/O | `4-1` 仿 `tee`，`4-2` 保留空洞的 `cp` |
| hw2 | 檔案 I/O | 支援 `-n` 的仿 `tail` |
| hw3 | 行程 | 含內建指令的簡易 shell |
| hw4 | `/proc` | `12-1` 依使用者列出行程，`12-2` 行程樹，`12-3` 列出開啟特定檔案的行程 |
| hw5 | 行程、管線 | `part1` 小工具，`part2` 完整 shell，`part3` 字典檔查詢 |
| hw6 | 管線 | `44-1` 雙向管線，`44-2` 自製 `popen`／`pclose` |
| hw7 | 訊號 | `22-3` `sigsuspend` 與 `sigwaitinfo` 比較，`22-4` System V 訊號 API |
| hw8 | IPC | `44-6`、`44-7` FIFO，`46-2` System V 訊息佇列 |
| hw9 | IPC、mmap | `49-1` 以 `mmap` 實作 `cp`，`52-3` 訊息佇列檔案伺服器，`54-1` 共享記憶體與號誌 |
| hw10 | 共享記憶體、訊號 | 使用環形緩衝區的生產者／消費者 |
| hw11 | Socket | 以 Unix socket、TCP、UDP 實作字典伺服器 |
| hw12 | `inotify` | 遞迴監控目錄 |

## 各作業說明

### hw1：檔案 I/O

- **4-1 `tee`**：從標準輸入讀取，同時寫到標準輸出與所有指定的檔案。加上 `-a` 為附加模式，不加則會清空檔案。按下 Ctrl-C 時會正常關閉檔案。
- **4-2 `cpHole`**：複製檔案時，用 `lseek` 把連續的空位元組變成檔案空洞，並保留原檔的權限。

```bash
cd hw1/4-1 && make && echo hello | ./tee -a out1.txt out2.txt
cd hw1/4-2 && make && ./cpHole source dest
```

### hw2：`tail`

從檔案尾端往回搜尋，印出最後幾行。預設 10 行，`-n <行數>` 可指定行數。

```bash
cd hw2 && make
./tail file.txt
./tail -n 5 file.txt
```

### hw3：簡易 shell

拆成多個小模組（`parse.c`、`run_command.c`、`builtin.c`、`is_background.c`）。顯示 `myshell ->` 提示符號後解析輸入，執行內建指令（`echo`，支援 `-n`，以及 `quit`／`exit`／`logout`／`bye`），其他指令則用 fork 與 exec 執行。

```bash
cd hw3 && make && ./shell
```

### hw4：`/proc`

三個程式都是從 `/proc` 讀取資訊。

- **12-1**：列出指定使用者擁有的行程。
- **12-2**：依各行程的父行程編號建立行程樹並印出。
- **12-3**：沿著 `/proc/<pid>/fd` 的連結，印出開啟指定檔案的行程。

```bash
cd hw4/12-1 && make && ./12-1 <使用者名稱>
cd hw4/12-2 && make && ./12-2
cd hw4/12-3 && make && ./12-3 <檔案路徑>
```

### hw5：小工具、shell、字典

- **part1**：`hostinfo`（主機名稱、核心版本、主機 ID）、`mydate`（格式化日期）、`printdir`（用 `getcwd` 印出目前目錄）、`mycat`（印出檔案內容）、`pipe_ls`（透過管線執行 `ls -l`）。
- **part2**：延伸 hw3 的 shell，加入 `cd`、`pwd`、`id`、`hostname`、`builtin`、背景執行（`&`）、輸入輸出重導向（`<`、`>`）與管線（`|`）。
- **part3**：`convert` 把文字字典轉成固定長度紀錄，`file_lookup` 在轉好的檔案中查詢單字。

```bash
cd hw5/part1 && make
cd hw5/part2 && make && ./myshell
cd hw5/part3 && make && ./convert <輸入檔> <輸出檔> && ./file_lookup <資料檔>
```

### hw6：管線

- **44-1**：父行程透過一條管線把文字送給子行程，子行程轉成大寫後，再透過第二條管線送回來。
- **44-2**：用 `pipe`、`fork` 與 `dup2` 實作 `my_popen` 和 `my_pclose`。

```bash
cd hw6 && make && ./44-1
```

### hw7：訊號

- **22-3**：測量父子行程交換 N 個訊號所花的時間，比較 `sigsuspend` 與 `sigwaitinfo`。
- **22-4**：以 `sigaction` 和 `sigprocmask` 實作 `sigset`、`sighold`、`sigrelse`、`sigignore`、`sigpause`，並與系統版本對照測試。

```bash
cd hw7 && make
./22-3 <訊號數量>
./22-4
```

### hw8：FIFO 與訊息佇列

- **44-6**：FIFO 流水號伺服器與客戶端，包含一個不開啟回應 FIFO 的客戶端。詳見 `hw8/44-6/README.txt`。
- **44-7**：伺服器以 `O_NONBLOCK` 寫入 FIFO，客戶端負責讀取。若一開始沒有讀取端，伺服器會等 10 秒後重試。
- **46-2**：把流水號伺服器與客戶端改用 System V 訊息佇列重寫。

請先執行伺服器，再執行客戶端。

```bash
cd hw8/44-6 && make && ./server      # 再執行 ./client 或 ./client_without_open_fifo
cd hw8/44-7 && make && ./server      # 再執行 ./client
cd hw8/46-2 && make && ./server      # 再執行 ./client
```

### hw9：`mmap`、訊息佇列、共享記憶體

- **49-1**：用 `mmap` 實作 `cp`。
- **52-3**：以訊息佇列實作檔案伺服器，客戶端送出路徑，伺服器回傳檔案內容。詳見 `hw9/52-3/README.txt`，並附有範例檔 `empty.txt` 與 `test.txt`。
- **54-1**：`writer` 把文字寫進 POSIX 共享記憶體並對具名號誌執行 post，`reader` 等待號誌後印出內容。

```bash
cd hw9/49-1 && make && ./49-1 test.txt copy.txt
cd hw9/52-3 && make && ./server      # 再執行 ./client test.txt
cd hw9/54-1 && make && ./writer      # 在另一個終端機執行 ./reader
```

### hw10：生產者／消費者

生產者與多個消費者透過共享記憶體中的有限緩衝區交換資料。生產者以固定速率送出資料，並用帶有序號的即時訊號通知消費者。

```bash
cd hw10 && make
./hw10 <資料數量 M> <傳送速率 R> <consumer 數量 N> <buffer size B>
```

### hw11：字典伺服器

同一個字典查詢功能，分別用三種傳輸方式實作。資料檔是 `fixrec`，由固定長度的紀錄組成。

| 部分 | 傳輸方式 | 伺服器 | 客戶端 |
| --- | --- | --- | --- |
| part1 | Unix domain socket | `usock_server` | `usock_client` |
| part2 | TCP | `isock_server` | `isock_client` |
| part3 | UDP | `udp_server` | `udp_client` |

```bash
cd hw11/part1 && make
./usock_server fixrec <socket 路徑>
./usock_client <socket 路徑>

cd hw11/part2 && make
./isock_server fixrec
./isock_client <伺服器主機名稱>

cd hw11/part3 && make
./udp_server fixrec
./udp_client <伺服器主機名稱>
```

### hw12：目錄監控

用 `inotify` 監控指定目錄與其所有子目錄，當檔案或目錄被建立、刪除、移出或移入時印出訊息。新建立的子目錄會自動加入監控。

```bash
cd hw12 && make && ./hw12 <目錄>
```

## 備註

- `hw8` 與 `hw9` 資料夾裡的 `tlpi_hdr.h`、`error_functions.c`、`get_num.c` 等檔案，是書中附的輔助程式碼。
- 請在各程式自己的資料夾內執行，因為有些程式使用相對路徑讀取資料檔。
