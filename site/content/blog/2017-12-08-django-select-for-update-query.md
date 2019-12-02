---
title: Django涉及金錢等敏感數據並發操作
m_slug: 'django select for update query '
author: Alex
categories:
  - Django
type: post
date: 2017-12-07T17:54:55.252Z
description: 在涉及一些對數據安全性敏感的應用中需要考慮安全的多線程操作
featured: /images/uploads/pic03.jpg
featuredpath: date
---
> 原創內容, 轉載請注明出處

Django開發的應用, 如果涉及到一些在多線程或者多進程情況下要保證數據的安全時, 我們需要考慮到數據的安全性. 也就是說在並發的環境下, 要避免敏感數據更新到數據庫時候被多線程覆蓋導致的數據不正確. 所以我們需要使用原子性的操作.

具體來說, 有兩種方式:

1. 使用**select_for_update**來鎖定操作的行, 並且結合**update_fields**與**transaction.atomic()**實現原子操作
2. 使用F()來實現原子操作, 當然也是要結合**update_fields**與**transaction.atomic()**

**使用select_for_update來鎖定行**

```
try:
    with transaction.atomic():
        locked_obj = MyModel.objects.select_for_update().get(pk=obj.id)
        locked_obj.price -= 2
        assert locked_obj.price >= 0
        locked_obj.save(update_fields=['price'])
except:
    print 'save failed'
```

比如上面這個代碼, 實際上輸出的sql類似

```
SELECT `一大堆字段` FROM `MyModel` WHERE `MyModel`.`id` = 5396 FOR UPDATE;
```

這裏我們就鎖定了這一行, 其他的對這行的操作都會阻塞住.但是這裏要注意, query語句中所使用的篩選字段一定要加索引或者是主鍵. 否則會導致整張表被鎖定.**最好還是使用主鍵, 並且是確定的值.**

**使用F()來實現原子操作**

```
obj.price -= 9.99
obj.save()
```

當我們一般使用這樣的語句的時候, 會產生如下的代碼:

```
UPDATE `hello_mymodel` SET `price` = '113.46' WHERE `id` = 1
```

可以發現如果是多線程的時候, 很容易導致值被覆蓋因而最終結果不可確定.所以需要原子性的操作.比如

```
obj.price = F('price') - 9.99
obj.save(update_fields=['price'])
```

這樣執行的代碼實際上爲:

```
UPDATE `hello_mymodel` SET `price` = 'price' - '9.99' WHERE `id` = 1
```

這樣就是原子性的操作了.但是這個有一個問題, 如果同時使用了post_save的信號, 那麼就會報錯. 除非重新從數據庫讀取一次值
