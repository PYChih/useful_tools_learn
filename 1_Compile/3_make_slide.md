# 3_make_slide
###### tags: `tools`
[TOC]
# 目標
- GNU Make簡介
- autotools極度簡介
- cmake極度簡介
# GNU Make簡介
## 用途
- 我能不能打一行指令就幫我自動編譯
- 我能不能只編一更動過的檔案
    - 包含改了`.h`檔案對應`.c`都可以自動重編
- 可不可以有靈活的編譯組態
    - 設定debug mode還是release mode
    - 設定編譯選項
    - compile time指定巨集

## 範例
### hello Makefile
- makefile編譯helloworld
    ```
    hello(target): hello.c(prerequisites)
        gcc -o hello(recipe) hello.c
    ```
    - 新增: target檔案不存在
        - 通常target是一個檔案，但這不是必要條件
    - 更新:
        - prerequisites檔案更動時間比target檔案還新
### 誰說makefile一定要編譯檔案?
- target, prerequisites and command
    ```
    test_file: dep_file
        @echo Test
    ```
    - 一開始只有makefile, 下make出現錯誤，無法產生dep_file
    - 產生dep_file後==會==執行`@echo Test`
    - 產生test_file就==不==執行`@echo Test`
    - 更新dep_file更動時間後==會==執行`@echo Test`
- note: `@`會讓命令不顯示出來
    - 例子:
        - `@echo Test`只會印出Test
        - `echo Test`: 會印出echo Test\n test
### 樹狀target
- 好幾個target的範例
```
root_target: sub_target1 sub_target2
    @echo $@
sub_target1:
    @echo $@
sub_target2:
    @echo $@
```
- 執行`make`
    ```
    sub_target1
    sub_target2
    root_target
    ```
- note: `$@`展開後是target名稱
## 變數
- 設定變數的幾種方式:
    - `VAR = VAL`
    - `VAR :=VAL`
    - `VAL ?=VAL`
    - `VAR +=VAL`
- 設定時機
    - 檔案，通常就是makefile內
    - make命令的參數
    - 環境變數
- 取值
    - `$(VAR)`
- 內建變數
    - `$@`
        - target名稱
    - `$^`
        - 所有的prerequisites名稱
    - `$<`
        - 第一個prerequisite名稱
        - 用途:
            ```
            target: dep1.c inc.h test.h
            <tab> gcc -o $@ $<
            ```
    - `$?`
        - 比target還新的prerequisites名稱
## 範例
### 變數設定:連動型`=`
- 修改、印出makefile變數的例子
    - 設定var1
    - 設定var2=var1
    - 更改var1
    - 印出var2
- note: 結果會是第二次設定的值
### 變數設定:立刻生效型`:=`
- 把本來的`=`改成`:=`
- note: 結果就不會變了
### 變數設定:預設型`?=`
- 預設變數
    ```
    VAR3?=default
    $(warning $(VAR3))
    ```
- 執行`make`
    - 印出default
- 執行`make VAR1=Test`
    - 印出Test
### 變數設定:加碼型`+=`
- 加碼變數
    ```
    VAR1=first
    $(warning $(VAR1))
    VAR1+=second
    $(warning $(VAR1))
    fake:
        @echo "Do something"
    ```
- 執行`make`
    - 印出first second
### 整合
- 整合以上
```
TARGET=hello
SRCS=hello.c
CFLAGS=-g -Wall -Werror

$(TARGET): $(SRCS)
    $(CC) -o $@ $(CFLAGS) $^
clean:
    rm -f $(TARGET)
```
## 其他功能
- 條件
- function
## 範例
### 條件判斷
- makefile的條件判斷
    ```
    test:
    ifeq ($(LOGNAME),test)
	    $(warning This is test)
    else
	    $(warning $(LOGNAME))
    endif

    fake:
	    @echo
    ```
- 關鍵字:
    - `ifeq (a, b)`
    - `else`
    - `endif`
### function
- 語法:
    - `$(函數名稱 參數)`
    - 例子:
        - `$(warning 訊息)`
        - `$(error 訊息`
        - `$(shell 命令)`
## 常見錯誤:
- 每個命令都是一個子程序，所以`cd`的會不如預期
## 補充:
- [ ] make的預設處理規則
- [x] `.PHONY`
    - 和檔案無關的target
    - 用法: `.PHONY: clean`
### note:
- recipe的執行和target以及prerequisites檔案時間資訊有關
- clean的目的不是要產生clean檔案，也就是說target執行recipe和target是否為檔案無關
# command note:
- [ ] `gcc`
    - `-I`:
- [x] `tree`: 可以顯示路徑下的樹狀結構
    - ![](https://i.imgur.com/RoKouZS.png)
- [x] `cloc`: 計算所有代碼的數量並同時排除注釋與空行
    - ![](https://i.imgur.com/NKKPrIV.png)
- [x] `touch`: change file timestamps

# reference
[GNU Make, Autotools, CMake簡介](https://www.slideshare.net/zzz00072/gnu-make-autotools-cmake)