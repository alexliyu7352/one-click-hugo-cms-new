---
title: Django快速制作繁體或簡體語言包
m_slug: django po setup for Simplified chinese
author: Alex
categories:
  - Django
type: post
date: 2017-12-05T10:45:08.959Z
description: 如何快速的根據簡體或者繁體語言制作對應的語言包
featured: /images/uploads/pic03.jpg
featuredpath: date
linktitle: django繁簡語言包
---
> 原創內容如需轉載請注明出處

- - -

Django是一個非常不錯的框架, 我們使用這個框架可以快速的開發一些web應用.而這其中最方便的一個模組就是多國語言模組.

往往我們做的一個WEB應用會有多個語言的版本, 如果是中文頁面做英文版本, 那麼當然需要一個一個的翻譯相應的字符串到英文. 但是如果是做繁體或者簡體的語言包, 那麼是否有比較快捷的方式呢?並不需要我們一個一個的字符串翻譯呢?

方法是有的, 如果默認語言並不是簡體或者繁體等中文, 那麼就比較簡單了, 直接使用cconv把PO文件從簡體轉換到繁體或者繁體轉換成簡體就好.

但是隨着大中華WEB應用的增多, 越來越多的WEB應用可能並不是考慮英文或者日文優先.而是直接默認就是繁體但是需要簡體語言包, 或者反之. 這個當然也可以實現自動轉換的, 但是操作起來稍微有一點的復雜.需要按照以下的步驟進行

1. 生成你需要翻譯的簡體或者繁體的A.PO文件. 
2. 使用cconv把這個PO文件轉換成對應的繁體或者簡體的B.PO文件.
3. 這時候並不可以直接使用, 因爲cconv會也把msgid翻譯成對應的語言, 這樣等於就識別不出來了.比如您的WEB默認是採用繁體書寫的, 你想要簡體的語言包. 這時候運行了步驟2, 把生成了一個簡體的PO文件. 這時你發現這個PO文件中, msgid也變成簡體了, 這顯然並不符合我們的需求.所以需要接下來做一個處理
4. 通過一段python小腳本, 遍歷A.PO文件, 把每個沒有翻譯的msgstr替換成B.PO文件的對應的msgid.因爲B.PO文件全部都翻譯成簡體了, 因此我們直接把B.PO的msgid復制給對應的A.PO文件的msgstr就實現了簡體語言包的制作.
5. 到上面的步驟4其實已經完成了簡體語言包的制作了, 但是如果默認語言以後改成了比如en-US, 但是頁面本身的書寫語言卻是繁體中文, 這時候就會出現不能順利切換語言的問題.所以需要把頁面書寫語言所對應的語言包也生成一個MO文件.

具體代碼如下:

1. 對應步驟2, 把默認語言繁體的PO文件, 先轉換成簡體的PO文件, 這時候會有兩個文件一個是最終簡體包一個是用來提取字符串的臨時文件.

```
cp locale/zh_Hant/LC_MESSAGES/django.po locale/zh_Hans/LC_MESSAGES/django.po
```

```
cconv -t UTF8-CN -f UTF8-TW locale/zh_Hans/LC_MESSAGES/django.po -o locale/zh_Hans/LC_MESSAGES/django.po.bak
```

2. 對應步驟4, 提取臨時簡體PO文件中的已經翻譯成簡體的msgid替換到最終簡體包PO文件對應的msgstr.

```
# -*- coding:utf-8 -*-
from __future__ import unicode_literals
import polib

__author__ = 'alex'

current_po_path = "./locale/zh_hans/LC_MESSAGES/django.po"
hant_po_path = "./locale/zh_hans/LC_MESSAGES/django.po.bak"

po = polib.pofile(current_po_path)
po_hant = polib.pofile(hant_po_path)


def judge_pure_english(keyword):
    return all(ord(c) < 128 for c in keyword)


for i in range(0, len(po)):
    # 遍歷所有的需要翻譯的字符串
    # 先判斷remote_file是否有對應的值, 如果有則直接復制, 如果沒有則使用msgid作爲值
    if po[i].msgstr:
        po[i].msgstr = po_hant[i].msgstr
    else:
        # 如果不是英文才需要處理, 英文應該預先翻譯好
        if not judge_pure_english(po[i].msgid):
            po[i].msgstr = po_hant[i].msgid

po.save()
```

3. 如果有需要那麼對應上面的步驟5, 對默認繁體語言的PO文件也生成完整的MO文件, 避免修改Django默認語言後不正常.

```
cconv -f UTF8-CN -t UTF8-TW ./locale/zh_Hans/LC_MESSAGES/django.po -o ./locale/zh_Hant/LC_MESSAGES/django.po
```

---

題外話, 如果有這種需求, 就是要把整個項目中的簡體轉換成繁體或者繁體轉換成簡體, 那麼也很簡單, 一樣使用cconv來轉換, 只需要一行代碼即可.

```
find . -type f -name '*.py' -exec cconv -t UTF8-CN -f UTF8-TW {} -o {}.result \; -exec mv {}.result {} \;

```
