---
tags: LinQ, LinQ基礎 , C#
---

# LINQ基礎 - 簡介
### 介紹

#### LinQ 的全名為 Language Integrated Query
* Language 指的是程式語言
* Integrated 指的是整合
* Query 指的是查詢

LinQ 是微軟在Net3.5推出的framework , 距今也N年了 , 但其重要性卻不減反增 . **LinQ 作為一種針對集合元素進行查詢的語言** , 不管你使用何種資料型態 , 只要該資料是實作 IEnumerable 介面的型別 , 都可以使用單一的 LINQ 語法去查詢處理. 也就是說 , 不論是物件、SQL 資料庫、XML 文件、各種 Web 服務等 , 你不需要去個別了解他們的查詢操作方式 , 而可以直接透過LinQ查詢操作即可. 

---

### 設計目標
LinQ希望可以藉由統一的資料存取模型來操作不同類型的資料. 因此使用LinQ
- 可以用**簡短易懂的運算式與呼叫式**達到目的
    - 可以減少糾結在資料操作的心力與時間
- 可以維持**一定效能**
    - LinQ 不會因為操作不同類型資料而使效能變差. (雖然多了跳入跳出某個函數的動作XD)
- 可以增加**延展性**
    - 很常透過委派去決定操作的細節.

---

### LinQ架構.

![](https://i.imgur.com/Bi1R7IC.jpg)


#### 由上圖可以知道
1.  LINQ能支援查詢物件、關聯式資料庫、XML , 共三種資料來源.
1.  針對上述三種資料來源 , LINQ也分別對應為三種類型 : LINQ to Objects、LINQ-enabled ADO.NET 、 LINQ to XML.
1.  LINQ 可以用 VB、C# 或「其他」語言撰寫 e.g. Python or Ruby

---
### 寫法
#### Linq有兩種寫法 Query and Lambda 
*  Using SQL like query expressions 
```C#
int[] intArr = { 5, 4, 8, 7 };
IEnumerable<int> greaterThanFive = from intItem in intArr
                                   where intItem >= 5
                                   select intItem;
```
*  Using Lambda Expressions. 
```C#
int[] intArr = { 5, 4, 8, 7 };
IEnumerable<int> greaterThanFive = intArr.Where(intItem => intItem >= 5);
```

以上兩個例子都是從intArr中取得所有大於等於五的整數.

---

### 使用時機

當這個任務需要讓你執行**走訪**這個動作的時候. 像是

- 你想要在陣列中尋找一個元素?
- 你想要在陣列中尋找多個符合條件的元素?
- 你想對某個陣列做排序?
- 你想要對這個陣列每一個元素做操作?
-  ...

---

### 總結
 
LINQ是一系列的擴充方法 , 讓開發人員**可以藉由統一的資料存取模型 , 以相同的模式存取操作各種不同型態的資料集合.**
舉個輕鬆的例子 , 原本你必須記住你目前使用的資料集合的類別有哪些函數 , 以利你操作它們 , 像是操作 List 有一套函數 , 操作 Array 有另一套函數 , 以此類推... 你可能必須記住 N 套函數 , 但現在你只需要記住 LinQ 的那一套函數就能夠對這些資料集合進行那些常見的操作.

---

### Thank you! 

You can find me on

- [GitHub](https://github.com/s0920832252)
- [Facebook](https://www.facebook.com/fourtune.chen)

若有謬誤 , 煩請告知 , 新手發帖請多包涵

# :100: :muscle: :tada: :sheep: 
