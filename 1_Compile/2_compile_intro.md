# 2_compile_intro
###### tags: `tools`
[TOC]
## 編譯簡介
- 查看`g++`版本:
    - `g++ --version`
        - `g++ (Ubuntu 7.5.0-3ubuntu1~18.04) 7.5.0`
- `g++`在執行編譯工作時候的流程:
    - 1. 預處理，生成`.i`的文件
    - 2. 將預處理後的文件轉換成匯編語言，生成`.s`文件
    - 3. 將彙編變為目標代碼(機器代碼)，生成`.o`的文件
    - 4. 連結目標代碼，生成可執行程序
- 1. 預處理階段
    - `-E`: stop after the preprocessing stage; do not run the compiler proper. The output is in the form of preprocessed source code, which is sent to the standard output.
    - `g++ -E test.cpp > test.i`
        - 查看test.i
    - 這一步主要做了巨集的替換和註釋的消除
- 2. 將預處理後的文件轉換成匯編語言
    - `-S`: Stop after the stage of compilation proper; do not assemble. The output is in the form of an assembler code file for each non-assembler input file specified.
    - `g++ -S test.cpp`
        - 查看`test.s`
    - 這一步主要表示彙編文件，用編譯器打開就是彙編指令
- 3. 將彙編語言變為目標代碼(機器代碼)
    - `-c`: compile or assemble the source files, but do not link.
    - `g++ -c test.cpp`
        - 查看`test.o`
    - 這一步就是生成目標文件，用編譯器打開就是二進制機器碼
- 4. 連接目標代碼，生成可執行程序
    - `-o`: place output in file file. This applies to whatever sort of output is being produced, whether it be an executable file, an object file, an assembler file or preprocessed C code.
    - `g++ test.o -o test`
## makefile基本介紹
- makefile基本格式:
    ```
    target : prerequisites
        command
    ```
    - target: 目標文件，可以是object file也可以是可執行文件
    - prerequisities: 生成target所需要的文件或目標
    - command: make需要執行的命令(任意的shell命令)
- 基本語法
    - 顯示規則
        - 指名target和prerequisit
        - 一條規則可以包含多個target，這意味每個target的prerequisite都相同，當一個target被修改後，整個規則中的其他target文件都會被重新編譯或執行
    - 隱晦規則
        - make的自動推倒功能所執行的規則
    - 變量定義
        - makefile中定義的變量，一般是字串
    - 文件指示
        - makefile中引用其他makefile
    - 註釋
        - 使用`#`
### 範例:
- [x] 使用makefile編譯一個使用opencv顯示圖片的cpp檔
- [參考](https://www.itread01.com/content/1544773878.html)
### makefile編寫
- 1. 編寫clean
    - 作用是刪除所有的`.o`和可執行文件
    ```
    clean:
        rm *o [exefile]
    ```
- 2. 編寫目標文件1:依賴文件1
    ```
    DisplayImage.o:DisplayImage.cpp
        g++ -c DisplayImage.cpp -o DisplayImage.o
    ```
    - recall: `-c`生成`.o`目標文件
- 3. 編寫目標文件2:依賴文件2
    ```
    DisplayImage:DisplayImage.o
        g++ DisplayImage.o -o DisplayImage
    ```
- 4. 應用opencv庫和頭文件
    - 結果這步文章沒講要自己查-.-
    ```
    export PKG_CONFIG_PATH=
    CXXFLAGS:=
    ```
## cmake基本介紹
- 安裝cmake
- 流程:
    - 編寫CMakeLists.txt
    - 執行cmake path生成makefile，其中path是CMakeLists所在目錄
    - 使用make進行編譯
- cmake常用命令
    - ==cmake_minimum_required==
        - 語法:`cmake_minimum_required(VERSION major[.minor[.patch[.tweak`
        - 簡述: 用於指定需要的cmake的最低版本
        - 範例: `cmake_minimum_required(VERSION 2.8)`
    - ==project==
        - 語法: `project(<projectname> [languageName1 languageName2]`
        - 簡述: 用於指定項目的名稱，一般和項目的文件夾名稱對應
        - 範例: `project(DisplayImage)`
    - ==aux_source_directory==
        - 語法: `aux_source_directory(<dir> <variable>)`
        - 簡述: 用於將dir目錄下的所有元文件的名字保存在變量variable中
        - 範例: `aux_source_directory(src DIR_SRCS)`
    - ==add_executable==
        - 語法: `add_executable(<name> source1 source2)`
        - 簡述: 用於指定一組源文件source1 source2... sourceN編譯出一個可執行文件且命名為name
        - 範例: `add_executable(DisplayImage DisplayImage.cpp)`
    - ==target_link_libraries==
        - 語法: `target_link_libraries(<target> [item1 [item2 [...]]][debug|optimized|general])`
        - 簡述: 指定target需要的鏈接item1 item2，這裡target必須已經被創建，鏈接的item可以是已經存在的target
        - 範例: `target_link_libraries(DisplayImage ${OpenCV_LIBS})`
    - ==add_subdirectory==
        - 語法: add_subdirectory(source_dir [binary_dir])
        - 簡述: 用於添加一個需要進行構建的子目錄
        - 範例:`add_subdirectory(Lib)`
    - ==include_directories==
        - 語法: `include_directories(AFTER|BEFORE]`
        - 簡述: 用於設定目錄，這些設定的目錄將被編譯器用來查找include文件
        - 範例: `include_directories($PROJECT_SOURCE_DIR/lib)`
### 範例:
- [x] 使用cmake編譯一個使用opencv顯示圖片的cpp檔
### 編寫CMakeLists
- 文件實際上放哪都可以
    ```
    cmake_minimum_required(VERSION 2.8)
    project(DisplayImage)
    find_package(OpenCV REQUIRED)
    add_executable(DisplayImage DisplayImage.cpp)
    target_link_libraries(DisplayImage ${OpenCV_LIBS})
    ```
- 進入`build/`執行: `cmake ..`
- 執行: `make`
- 執行: `DisplayImage ../dog.jpg`
## 指令整理
- `g++`:
    - `-E`, `-S`, `-c`, `-o`
## reference

- [g++, CMake和Makefile了解一下](https://mp.weixin.qq.com/s?__biz=MzA3NDIyMjM1NA==&mid=2649031006&idx=1&sn=c2bbb57e95ccf651eec22fe378160095&chksm=8712bf23b0653635fb1a932aa33dea5a5f6d75e4767cdbebd4b8809b108c8b2f4339b215f8ea&scene=21#wechat_redirect)
- [CMake Documentation](https://cmake.org/cmake/help/v2.8.8/cmake.html#section_Commands)
- [makefile opencv的案例](https://www.itread01.com/content/1544773878.html)