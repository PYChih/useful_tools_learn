# 1_compile_101
###### tags: `C-21st`
[TOC]
## 基本的環境
#### Linux
- 啥都不用幹
#### Windows下編譯C語言
- [ ] 將Cygwin的bin路徑加到windows的path環境中
- [ ] mintty
- Cygwin:
    - cygwin編譯的程式碼會使用到cygwin1.dll函式庫提供的POSIX函式
    - 如果想在沒有安裝Cygwin的主機上執行以Cygwin編譯的程式，就必須同時提供執行檔與cygwin1.dll
    - [ ] cygwin1.dll使用的是==類GPL授權==，也就是說，要是你將這個DLL抽離Cygwin單獨散布就必須要發布你的應用程式的原始程式碼(?)
    - 如果這是問題，就需要透過其他方式重新編譯，讓程式不會相依於cygwin1.dll
    - [ ] 檢查程式或動態連結函式庫所使用的其他函式庫:
        - [ ] Cygwin: `cygcheck libxx.dll`
        - [ ] linux: `ldd libxx.so`
- MinGW(Minimalist GNU for windows)
    - 如果程式不需要POSIZ函式，可以使用MinGW
    - 最大問題: 預先編譯的函式庫太少
    - 透過Cygwin的Mingw32編譯建立不使用POSIX的程式碼:
        - `./configure --host=mingw32`
        - `make && make install`
        - 在MinGW下編譯，不論是使用命令列編譯或是autotools，只要是在MinGW下編譯，最終結果都是原生windows執行檔
- code:block
    - 利用MinGW在Windows上編譯的IDE
## 協助工具
- 編譯器(gcc)
- make
- pkg-config(尋找函式庫使用)
- 編輯器
- autotools:
    - 使用autotools建置套件:
    `./configure && make && make install`
## 使用工具編譯程式
### 連結函式庫的方式
- 從命令列直接編譯
    - 1. 設定變數標示使用的編譯器旗標
    - 2. 設定變數表示連結的函式庫
        - 編譯期連結的函式庫
        - 執行期的函式庫
    - 3. 設定系統使用上述變數進行編譯
- note: 要使用函式庫，就必須告知編譯器引入函式庫中的函式兩次，一次在編譯過程，一次在連結過程，對於在標準位置的函式庫而言，這兩個動作透過:
    - include
    - 編譯器命令列的`-l`
- [x] ==範例1-1==: 命令列編譯[erf.c](https://github.com/b-k/21st-Century-Examples/blob/master/erf.c)
- 說明:
    - 編譯:`#include`: 將`.h`的內容複製到原始檔的相對位置，`.h`並沒有說明函數的行為，只表示這個函式需要的參數以及回傳值型別，這就足以讓編譯器檢查函數使用方式的正確性，並產生帶有註記的目的檔
    - 連結:
        - `math.h`檔案中的數學函式被分離放在個別函式庫，必須利用`-lm`旗標告知連結器，其中`-l`是指定連結函式庫的旗標
        - 連結器預設會在最後附加`-lc`旗標，連結到標準`libc`函式庫
        - 其他範例: 
            - `-lglib-2.0`: 連結GLib2.0
            - `-lgsl`: 連結GNU Scientific Library
    - 完整編譯命令: 
        ```
        gcc erf.c -o erf -lm -g -Wall -O3 -std=gnu11
        ```
        - `-o`: 輸出檔名(沒指定預設是`a.out`)
        - `-g`: 產生除錯用的符號表
        - `-std=gnu11`: gcc特有的旗標，指示編譯器接受符合C11與POSIX標準的程式碼
            - 如果使用的gcc版本早於C11可以使用`-std=gnu99`
        - `-O3`: 最佳化等級3
            - 如果除錯時發現最佳化改變了太多變數，影響除錯進行，可以切換回`-O0`
        - `-Wall`: 增加編譯器警告
            - `-W1`: 顯示編譯器警告而非注意
            - `-Werror`: 讓編譯器將警告視為錯誤
### 路徑
- 一般的設定下，函式庫至少有三個可能的安裝位置
    - 作業系統商定義目錄，安裝官方提供的函式庫
    - 供系統管理員安裝套件的目錄，避免檔案內容受到作業系統升級的影響
        - 一般在`/usr/local`
    - 一般使用者沒有在上述兩個位置安裝檔案的權限，而是安裝在家目錄
- 範例:
    - `gcc -I/usr/local/include use_useful.c -o use_useful -L/usr/local/lib -luseful`
        - `-I`: 將指定的目錄加到引入檔搜尋路徑
        - `-L`: 將路徑加入函式庫搜尋路徑
        - `-luseful`: 函式庫的名子(ex: 像本來的`math.h`就是`-lm`)
    - note: 順序很重要
        - 如果specific.o相依於Libbroad函式庫，Libbroad相依於Libgeneral，那就必須使用:
        - `gcc specific.o -lbroad -lgeneral`
            - 連結器先找第一個項目`specific.o`列出一系列還沒有找到對應位置的函數、結構與變數名稱，接著找下一個項目，同時可能增加新的為對應項目，包含最後隱藏的預設`-lc`連結器就會停止繼續處理。
- 搜尋函式庫的位置:
    - `find /usr -name 'libuseful*'`
    - 搜尋`/usr`目錄下檔名以`libuseful`開頭的檔案
#### pkg-config
- 透過維護旗標與位置資料庫解決搜尋檔案問題
- 例子:
    - `pkg-config --libs gsl libxml-2.0`
        - 輸出: `-lgsl -lgslcblas -lm -lxml2`
    - `pkg-config --cflags gsl libxml-2.0`
        - 輸出: `-I/usr/include/libxml2`
- shell技巧:
    - 用反引號(backtick)把命令括起來時，會用命令的執行結果取代命令，也就是: 
    ```
    gcc `pkg-config --cflags --libs gsl libxml-2.0` -o specific specific.c
    ```
    編譯器看到的命令會就會是pkg-config執行的結果
### 執行期連結
- 靜態函式庫由編譯器連結，將函式庫相關部分複製到最終執行檔
- 共享函式庫在執行期與程式連結，這表示在執行期要再次面對編譯時間期相同的找尋函式庫問題
    - 如果函式庫在標準位置，就沒差，如果並非安裝在標準位置，就要透過其他方式修改執行期的函式庫搜尋路徑:
        - 使用autotools打包程式時，Libtool能夠加入正確的旗標
        - [ ] 使用gcc搭配位於libpath路徑的函式庫編譯時，可以在makefile的最後加上:
        - `LDADD=-L[libpath] -Wl, -R[libpath]`
                    - `-L`提供編譯器找尋函式庫解析符號的位置
                    - `-Wl`旗標會將`-L`旗標從gcc傳遞到連結器
                    - 連結器再將`-R`旗標的資訊嵌入程式
        - `export LD_LIBRARY_PATH=libpath:$LD_LIBRARY_PATH`
## 使用Makefile
- 最簡單的makefile
```
P=program_name
OBJECTS=
CFLAGS = -g -Wall -O3
LDLIBS=
CC=c99

$(p): $(OBJECTS)
```
### 設定變數
- shell與make都使用`$`表示變數的值，shell使用`$var`，make則要求超過一個字元的變數名稱必須以小括號刮起來
    - 因此: `$(p):$(OBJECTS)`等同於`program_name:`
- 以下幾種方式能讓make辨識出變數:
    - 執行make前先從shell中設定變數(export變數)，表示當shell產生子程序時，子程序環境變數清單中也會有這些變數，例子:
        - `export CFLAGS='-g -Wall -O3'`
        - 可以省略make file的第一行，改成在每個作業階段(session)使用`export P=program_name`設定變數，就不用一再重複編輯makefile
    - 將export命令放在shell的啟動命令稿(如`.bashrc`或`.zshrc`以確保每次登入或開啟新shell時，都能正確設定變數，如果確定CFLAGS的值每次都會相同，就可以設定在啟動檔
    - 將變數指派命令放置在執行命令之前，就能夠針對個別命令設定環境變數。`env`命令會列出所有的環境變數，`PANTS=kakhi env | grep PANTS`
        - 這也是shell不允許指派命令等號的兩側出現空格的原因，空格是用來判別指派命令與後續指令之用
        - 使用這種形式的設定與export的差別是只會對該行命令有效，執行完上面的命令後再執行`env | grep PANTS`可以確認PANTS已經不在環境變數中
        - 可以指定任意數量的變數:
            - `PANTS=kakhi PLANTS="ficus fern" env | grep 'P.*NTS'`
    - 在makefile的開頭設定變數
        - 在makefile中等號的兩邊可以有空白，不會造成錯誤
    - make也允許從命令列設定變數，不會與shell產生交互影響，因此，以下兩行命令有相同的效果:
        - `make CFLAGS="-g -Wall"`: 設定makefile變數
        - `CFLAGS="-g -Wall" make`: 針對make以及子程序設定環境變數
- 奇怪的知識:
    - 在c語言中可以使用getenv取得環境變數(在stdlib.h)
    - [x] ==範例1-2==: 環境變數提供調整程式細節的快速機制[getenv.c](https://github.com/b-k/21st-Century-Examples/blob/master/getenv.c)
- make也提供幾個內建變數
    - `$@`: 目標的完整檔案名稱，目標是指需要建置的檔案，例如`.o`檔案需要從`.c`檔案編譯而來，或是程式透過連結`.o`檔案而來
    - `$*`: 不含延伸檔名的目標黨，如果目標是`prog.o`那麼`$*`會是`prog`，`$*C`就會是`prog.c`
    - `$<`: 觸發建置目標的檔案名稱，例如建置`prog.o`的原因是因為`prog.c`有所異動，那麼`$<`就會是`prog.c`
### 規則
- makefile執行的流程以及變數對流程的影響
- 除了設定變數以外，makefile的其他部份都呈現出以下形式:
    ```
    目標: 相依
         命令搞
    ```
- 如果透過`make 目標`的方式呼叫特定目標，會檢查對應的相依性。
- 如果目標與相依性都是檔案，而且目標檔案的建立時間遠於相依的檔案，那目標檔案就處於最新的狀態，也就不需要執行任何動作
- 否則，會先暫停對目標的處理，先執行或產生相依所需的檔案
- 當所有相依的命令搞都執行完畢處於最新狀態時，才會執行目標的命令搞
- [x] 修改makefile編譯erf.c
### 使用函式庫原始碼
- 目前為止介紹了使用make編譯自己寫的程式碼，編譯其他人提供的程式碼完全是另一回事
- 以GNU Scientific Library做為示範套件，這個套件也包含了許多數值計算的函數
- GSL是用autotools打包，autotools是一組能建立可以在任何主機上使用的函式庫的工具
- 指令:
    - `wget ftp://ftp.gnu.org/gnu/gsl/gsl-1.16.tar.gz`
    - `tar xvzf gsl-*gz`
    - `cd gsl-1.16`
    - `./configure`
    - `make`
    - `sudo make install`
- [x] ==範例1-3==用GSL重新執行範例1-1: [gsl_erf.c](https://github.com/b-k/21st-Century-Examples/blob/master/gsl_erf.c)
# Linux cammand
- [x] find
    - `find /usr -name 'libuseful*'`
    - 搜尋`/usr`目錄下檔名以`libuseful`開頭的檔案
- [x] wc
    - print newline, word, and byte counts for each file
    - `-l` lines
    - `-w` words
    - `-c` bytes
- [x] export
    - 在shell中執行程序時，shell會提供一组環境變量。export可新增，修改或删除，供後續執行的程序使用。export的效力僅於該次登陸
- [x] env
    - 列出所有環境變數，可以搭配grep使用