---
title: Pycharm&&IDEA快速編譯typescript
m_slug: pycharm and idea compile typescript
author: Alex
categories:
  - Typescript
type: post
date: 2017-12-06T17:09:06.296Z
description: 如何在不配置config的情況下快速編譯typescript
featured: /images/uploads/pic02.jpg
featuredpath: date
---
> 原創內容如需轉載請注明出處

- - -

我們使用Pycharm或者IDEA時候的很多項目前端模組我們都是會使用typescript來寫, 畢竟typescript可以使得開發javascript像寫java或者其他語言一樣.

但是我們知道, typescript雖然有些瀏覽器支持,但是最保險的還是編譯成javascript.而編譯成javascript一般來說我們需要先配置config文件然後用tsc編譯.

但是其實在上面我說的這兩個IDE中不用那麼麻煩, 只需要在設置中設置一下即可完成編譯.設置時需要注意幾點(設置是在**File---settings---language & frameworks---typescript**裏面設置):

1. 配置好nodejs以及安裝typescript, 這個應該大家都會
2. 在源碼下創建一個用來引用所有模組的app.ts文件, 裏面使用**reference **引用所有需要編譯的模組文件,如:
   ```
   /// <reference path="./views/devices.ts" />
   /// <reference path="./views/helpdesk.ts" />
   /// <reference path="./views/users.ts" />
   ```
3. 在**Compile scope**中新建一個scope, 並且包含您要編譯的源碼目錄.(這裏注意, 如果你有引用d.ts的定義文件請排除在編譯目錄以外)
4. 選擇上**Compile main file only**選項, 并且設置路徑爲步驟2所創建的這個引用文件.
5. **Options**中設置參數**–outFile /path/xxx.js **這個參數可以讓typescript輸出一個單一的文件, 便於管理.
6. 編譯的時候只需要在typescript的窗口左側的按鈕上選擇**Compile all**即可編譯成功, 並且生成一個單一的文件.或者選擇上**reCompile on changes**選項, 這樣一旦某個文件修改了就會自動重新編譯

生成了單一文件後即可使用webpack或者其他的打包工具進行打包.但是如果項目比較簡單那麼可以直接對上面操作編譯生成的單一的javascript文件進行壓縮加密. 這裏就需要用到另一個工具了:**uglifyjs2**

我們可以使用如下的命令進行再次的壓縮混淆js文件:

```
uglifyjs ./your path/xxx.js -o ./your path/xxx.min.js -c -m --keep-fnames
```
