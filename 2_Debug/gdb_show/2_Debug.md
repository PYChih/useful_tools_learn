# 2_Debug
###### tags: `C-21st`
[TOC]
## overview
- 除錯:
    - 除錯器的介紹
    - 使用GDB檢查邏輯錯誤等瑣碎的問題
    - 使用Valgrind檢查記憶體配置不當或洩漏等更加技術性的問題
## 除錯
### 使用除錯器
- 除錯器的介紹
- Frame的堆疊(stack)
### 除錯範例
- 使用: [stddev_bugged.c](https://github.com/b-k/21st-Century-Examples/blob/master/stddev_bugged.c)
- 正確版本: [stddev.c](https://github.com/b-k/21st-Century-Examples/blob/master/stddev.c)
- 編譯後執行gdb: 
- 問1: 這個程式做了些什麼
    - `r`: run指令會執行程式
    - 可以看到程式產生一些mean和variance，程式能夠順利執行完畢，不會產生segfault也沒有其他錯誤
- 問2: 顯示程式碼 
    - `l`: list指令能夠顯示程式碼
        - 不加參數顯示的是接下來的十行程式碼
        - 使用`l main`會顯示以main函數為中心的十行程式碼
    - 這個程式算mean、var有錯
- 問3: 怎樣能夠看到mean_and_var的行為
    - 想讓程式在mean_and_var暫停，得先放個breakpoint:
        - `b mean_and_var`
    - 設定好中斷點後，重新執行程式就會在中斷點的位置停止
- 問4: data內容和預期是否相符?
    - `p`: print指令可以查看這個frame裡的data變數值
    - 如果變數是指標，可以直接解參考
        - `p *data`
    - 有個特別的`@-`語法能夠顯示陣列裡一連串的元素
        - `p *data@10`
- 問5: 數值是否與main傳入的相同
    - 可以使用`bt`: 列出frame stack
    - ![](https://i.imgur.com/PYxH3mU.png)
    - frame堆疊包含目前所在的frame共有兩層，目前frame的呼叫者就是main。接下來看看frame1的資料內容，先切換到frame1: `f 1`
    - ![](https://i.imgur.com/PGz2pHl.png)
        - 這個frame的資料陣列名稱是d: `p *d@7`
        - 看起來和mean_and_var frame接收到的資料相同，所以資料及似乎沒有異常
        - 要繼續執行不一定需要切換到frame0，如果要切換到frame0直接使用`f 0`
        - 也可以使用堆疊相對位置的方式切換frame: `down`
- 問6: 平行執行緒有沒有問題?
    - 使用`info threads`可以顯示執行緒列表
    - ![](https://i.imgur.com/trHHXlX.png)
        - 只有一個活躍的執行緒，不會有多執行緒的問題
        - 如果有多個執行緒可以透過GDB的thread 2指令切換到第二個執行緒
- 問7: mean_and_var做了什麼
    - 我們可以逐行執行程式: `n`
    - 不輸入直接按Enter會重複執行上一個指令
    - 行號呈現出程式跳著執行，這是因為每一個步驟，除錯器實際上是執行機器碼指令
    - 在程式裡有個處理data矩陣的for迴圈，我們可以把中斷點設在迴圈當中: `b 21`
    - 可以使用GDB的`info break`顯示所有的中斷點
    - 現在已經不需要第一個中斷點了，可以關閉第一個中斷點: `dis 1`
        - 執行完後用`info break`查看，Enb顯示: n
        - 可以透過`enable 1`重啟中斷點
        - 若是確定中斷點不需要，可以透過`del 1`刪除中斷點
- 問8: 迴圈當中的變數值看起來如何
    - 可以用r指令重新執行，也可以用c繼續執行
    - `info local`可以看到所有的區域變數
    - `info args`指令可以確認輸入參數
- 問9: 我們知道輸出的平均值有錯，那每個迴圈執行時avg會有怎樣的變化?
    - 可以在每次停在中斷點時輸入`p avg`但display指令能夠自動產生相同效果 `disp avg`
    - 可以使用`undisp 1`關閉自動顯示
- 問10: 先前檢查過data，那個ratio與count呢?
    - `disp ratio`
    - `disp count`
    - 可以看到count依次增加，但ratio沒有變化
- 問11: 什麼地方會設定ratio?
    - 用文字編輯器或l指令查看
### 除錯器常用指令
- `run`: 從頭執行程式
- `run args`: 從指定的命令列參數從頭開始執行程式
- `b get_ress`: 在指定的函式暫停程式
- `b nyt_feeds.c:105`: 在指定的程式行之前暫停程式
- `break 105`: 如果已經停在nyt_feeds.c:105裡，效果就跟b nyt_feeds.c相同
- `info break`: 列出中斷點
- `watch curl`: 如果指定變數的值發生變化就暫停
- `dis/ ena/ del`: 停用/重啟/刪除中斷點
- `p url`: 列出變數url值
- `p *an_array@10`: 列出an_array前十個變數
- `mem read -tdouble -c10 an_array`: 從an_array讀取數量10個的double型別項目
- info args/ info vars: 取得函式的所有區域變數值
- `disp url`: 每次程式暫停就顯示url值
- `undisp n`: 停止顯示第n個顯示項目
- `info thread`: 列出目前所在執行緒
- `thread n`: 切換到執行緒n
- `bt`: 列出frame堆疊
- `f 3`: 檢視frame 3
- `up / down`: 在frame堆疊往上/往下移動
- `s`: 單步執行，會進入函式內
- `n`: 執行下一步，但不會進入函式
- `u`: 執行到目前所在行的下一行
- `c`: 執行到下個中斷點或程式結尾
- `ret`: 立刻結束目前的函式
- `j`: 跳到指定的位置
- `l`: 目前在位置附近的十行程式碼
- `enter`: 重複上一個指令
### GDB變數
- 在GDB建立輔助的變數，減少大量重複的打字輸入
    ```
    // 範例程式 donothing.c
    int main() {
        int x[20] = {};
        x[0] = 3;
    }
    ```
- 第一次定義變數時，GDB要使用set與`$`
    - `set $ptr=&x[3]`
    - `p *$vd@10`
- 這些變數是真正的變數
    - 比如定義一個變數指向陣列的第3個元素
    - `set $ptr=&x[3]`
    - `p *$ptr=8`: 這個會真的去改x
    - `p *($ptr++)`: 指標往後
        - 由於直接按enter會執行前一個命令，所以每次按enter就會讓我們得到下一個的值
- 每個輸出都指定有各自的變數名稱，可以像自訂變數一樣操作(比如解參考它)
- 另一個技巧是: `$`本身代表上一個變數
### 印出結構
- [ ] 定義巨集顯示複雜資料結構
    - [ ] 二維陣列
- 可以藉由hook讓印出的結構更清楚
### reference
- [hook-print](https://sourceware.org/gdb/onlinedocs/gdb/Hooks.html)