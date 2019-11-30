---
tags: LinQ , C# , Partitioning Operators
---

# Take & Skip

Take 與 Skip 是非常相似的 , 更精確的說法是他們在使用層面是互補的存在. Take 可以從原始資料集合中取出**滿足使用者設定的條件或是個數**的資料以取出**連續的**目標子集合. Skip 可以**透過忽略使用者設定的條件或是個數的資料**以取出**連續的**目標子集合. 也就是說將 Take 與 Skip 兩者取得的資料合併後 , 剛好為原本的資料集合(無重複). 故 Take 與 Skip 被稱為是 Partitioning Operators. 我自己看待這兩個方法 , 是想成使用某個茅點將資料集合切成兩個部分 , 然後一個部分使用 Take 取得 , 而另一個部分使用 Skip 取得.


Take 與 Skip 都各有三個方法可以使用. 
- Take 有 Take、TakeWhile、TakeLast
- Skip 有 Skip、SkipWhile、SkipLast

### Take
Take 可以取出使用者指定數量的連續項目的子集合. 其數量是從資料集合開頭開始算起. **若是使用者指定的數量超過資料集合數量 , 其會將整個資料集合回傳. 不會有例外發生**
```C# 
IEnumerable<TSource> Take<TSource> (this IEnumerable<TSource> source, int count);
```
- count 是指從資料集合開頭開始算起的個數.

#### 使用時機 
槓 , 我從來沒用過呀. :dizzy_face: 網路上查到都是在做分頁的時候會用到呀.
- 當你希望從資料集合中取出指定數量的資料 , 但你又不確定資料集合是否具備這麼多資料的時候.
    - 舉例 : 你希望得到全校每班前三名的學生資料. 故你可能會對每班的學生資料做成績的排序. 然後再取出每班成績**前三順位的學生資料** . 但你不能保證每一班上是不是真的有超過三個人... :crying_cat_face: . 此時使用 Take 就可以少一個 if 的邏輯判斷XD
- 使用數量為基準來進行一次集合分割動作.
    - 例如 : 使用數量三來做為資料切割的茅點 , 所以資料集合將會被分成資料集合的前三個 , 以及剩下的 , 兩部分. 並希望取出**前三個**這部分的資料成員.

#### Take 的用法
取出前三個英文字母並印出
```C#
static void Main(string[] args)
{
    string[] alphabets = { "A", "B", "C", "D", "E", "F", "G", "偷懶不想寫" };
    var firstThreeAlphabet = alphabets.Take(3); // Type is IEnumerable<string>
    foreach (var alphabet in firstThreeAlphabet)
    {
        Console.WriteLine(alphabet);
    }
    Console.ReadKey();
}
```
##### 輸出結果
A   
B   
C   

#### 簡單實作自己的 Take
```C#
public static IEnumerable<TSource> MyTake<TSource>(this IEnumerable<TSource> sources, int count)
{
    if (sources is null) throw new Exception("source is null");
    return MyTakeIterator(sources, count);
}

private static IEnumerable<TSource> MyTakeIterator<TSource>(IEnumerable<TSource> sources, int count)
{
    foreach (var item in sources)
    {
        if (--count < 0) yield break;
        yield return item;
    }
}
```

---

### TakeWhile
TakeWhile 會從資料集合開頭逐一檢查資料成員是否滿足條件 , 一旦**遇到不滿足條件的資料成員會中止搜尋** , 即使仍有符合條件的資料成員. 其與 Where 的差別在於 Where 會走訪所有的資料集合並將**所有滿足條件**的資料成員回傳.

1. IEnumerable<TSource> TakeWhile<TSource> (this IEnumerable<TSource> source, Func<TSource,bool> predicate);
    - 只要指定的條件為 true，就會傳回序列中的項目。
2. IEnumerable<TSource> TakeWhile<TSource> (this IEnumerable<TSource> source, Func<TSource,**int**,bool> predicate)	
    - 只要指定的條件為 true，就會傳回序列中的項目。 項目的索引是用於述詞功能的邏輯中。
- 兩個方法的差別在於 predicate 有沒有傳入 index 參數    
- predicate : 終止查詢的函數 , 一旦其結果為 false , 則立刻中止查詢.
- TakeWhile : 是由 predicate 來判斷是否終止查詢. 也就是說 , TakeWhile 回傳的資料成員皆是 predicate 判斷為 true , 且位置相鄰連續.

#### 使用時機
- 使用條件來進行一次集合**分割**動作
    - 例如 : 條件為年紀大於等於二十 , 資料集合為學生
        ```mermaid
        gantt
            title 使用年紀大於等於二十這個條件分割資料集合
            section 學生
            在第一個遇到年紀小於20的學生左邊:done,des1, 1,3
            在第一個遇到年紀小於20的學生右邊(含):des2, after des1, 5
        ```
        想要**取出紅色部分** , 此時可使用 TakeWhile(年紀大於等於二十) .
#### TakeWhile 的用法
```C#
// 使用字串的字數大於二為條件來進行集合的分割 , 並取出前部分.
static void Main(string[] args)
{
     string[] strArr = { "ABCDE", "FIJ", "KLMN", "OP", "QRST", "UV", "WXYZ" };
     IEnumerable<string> strArrTakeWhileArr = strArr.TakeWhile(s => s.Length > 2);
     foreach (string strArrTakeWhileArrItem in strArrTakeWhileArr)
     {
          Console.WriteLine(strArrTakeWhileArrItem);
     }
     Console.ReadKey();
}
```
##### 輸出結果
ABCDE    
FIJ    
KLMN

```
// 使用字串的字數大於二以及索引小於二為條件來進行集合的分割 , 並取出前部分.
static void Main(string[] args)
{
     string[] strArr = { "ABCDE", "FIJ", "KLMN", "OP", "QRST", "UV", "WXYZ" };
     IEnumerable<string> strArrTakeWhileArr = strArr.TakeWhile((s, index) => 
                                                     s.Length > 2 && index < 2);
     foreach (string strArrTakeWhileArrItem in strArrTakeWhileArr)
     {
          Console.WriteLine(strArrTakeWhileArrItem);
     }
     Console.ReadKey();
}
```
##### 輸出結果
ABCDE    
FIJ


#### 簡單實作自己的 TakeWhile 
```C#
public static IEnumerable<TSource> MyTakeWhile<TSource>(this IEnumerable<TSource> sources, Func<TSource, bool> predicate)
{
     if (sources is null || predicate is null) throw new Exception("source or predicate is null");
     return MyTakeWhileIterator(sources, predicate);
}

private static IEnumerable<TSource> MyTakeWhileIterator<TSource>(IEnumerable<TSource> sources, Func<TSource, bool> predicate)
{
     foreach (var item in sources)
     {
          if (!predicate(item)) yield break;
          yield return item;
     }
}

public static IEnumerable<TSource> MyTakeWhile<TSource>(this IEnumerable<TSource> sources, Func<TSource, int, bool> predicate)
{
     if (sources is null || predicate is null) throw new Exception("source or predicate is null");
     return MyTakeWhileIterator(sources, predicate);
}

private static IEnumerable<TSource> MyTakeWhileIterator<TSource>(IEnumerable<TSource> sources, Func<TSource, int, bool> predicate)
{
     int index = 0;
     foreach (var item in sources)
     {
          if (!predicate(item,index)) yield break;
          yield return item;
          checked { index++; }
     }
}
```

---
### TakeLast
**.NET Core 2.0 後** , 才提供的 LINQ 方法 TakeLast(). TakeLast 在順序上與 Take 相反. TakeLast 可以取出使用者指定數量的連續項目的子集合. **其數量是從資料集合尾部開始算起**. 若是使用者指定的數量超過資料集合數量 , 其會將整個資料集合回傳. 不會有例外發生

```C#
IEnumerable<TSource> TakeLast<TSource> (this IEnumerable<TSource> source, int count);
```
- count: 從最後一個開始倒數的元素數量

#### 使用時機
- 使用 count 去取出倒數的資料成員.
    - 雖然我覺得用 where((item,index)=>index>=collection.size-count) 應該也有一樣的效果
#### TakeLast 的用法
```C#
// 取出倒數的三個字母
static void Main(string[] args)
{
     string[] alphabets = { "A", "B", "C", "D", "E", "F", "G", "偷懶不想寫" };
     var firstThreeAlphabet = alphabets.TakeLast(3); // Type is IEnumerable<string>
     foreach (var alphabet in firstThreeAlphabet)
     {
          Console.WriteLine(alphabet);
     }
     Console.ReadKey();
}
```
##### 輸出結果
F
G
偷懶不想寫

#### 簡單實作自己的 TakeLast
```C#
public static IEnumerable<TSource> MyTakeLast<TSource>(this IEnumerable<TSource> source, int count)
{
     if (source is null)
     {
          throw new Exception("source is null");
     }
     return count <= 0 ? new TSource[0] : MyTakeLastIterator(source, count);
}

private static IEnumerable<TSource> MyTakeLastIterator<TSource>(IEnumerable<TSource> source, int count)
{
     Queue<TSource> queue = new Queue<TSource>();
     foreach (var item in source)
     {
          if (queue.Count >= count)
          {
               queue.Dequeue();
          }
          queue.Enqueue(item);
     }

     foreach (var item in queue)
     {
          yield return item;
     }
}
```
---
### Skip
略過指定數量的集合元素 , 取其剩下的集合元素作為回傳結果. 指定數量的計算是從集合的第一個元素開始算起. **若指定數量大於集合元素則不會回傳任何元素 , 且不會跳出例外.**

 ```C#
 static IEnumerable<TSource> Skip(this IEnumerable<TSource> source, int count);
 ```
 - count: 要略過的元素數量

#### 使用時機

- 對集合的運算操作 , 希望略過某幾個元素.
    - 例如 : 計算集合中的最大值. 預先取出集合的第一個元素 , 然後依序比對剩下的元素的大小. 因為第一個元素已被取出 , 因此走訪比對的時候 , 需要略過第一個元素.
Set.Skip(1).Aggregate(Set.First(), (result, item) => result > item ? result : item);
    
- 使用數量作為基準來進行集合分割動作
    - 例如 : 使用數量三來做為資料切割的茅點 , 所以資料集合將會被分成資料集合的前三個 , 以及剩下的 , 兩部分. 並希望取出**剩下的**這部分的資料成員.

#### Skip 的用法
```C#
// 略過六個字母後 , 輸出剩下的字
static void Main(string[] args)
{
     string[] alphabets = { "A", "B", "C", "D", "E", "F", "G", "偷懶不想寫" };
     var firstThreeAlphabet = alphabets.Skip(6); // Type is IEnumerable<string>
     foreach (var alphabet in firstThreeAlphabet)
     {
          Console.WriteLine(alphabet);
     }
}
```
##### 輸出結果
G
偷懶不想寫
#### 簡單實作自己的 Skip
```C#
public static IEnumerable<TSource> MySkip<TSource>(this IEnumerable<TSource> source, int count)
{
     if (source is null) throw new Exception("source is null");
     count = count < 0 ? 0 : count;
     return MySkipIterator(source, count);
}

private static IEnumerable<TSource> MySkipIterator<TSource>(IEnumerable<TSource> source, int count)
{
     foreach (var item in source)
     {
          if (--count < 0)
              yield return item;
     }
}
```
### SkipWhile
SkipWhile 會從資料集合開頭逐一檢查資料成員是否滿足條件 , 滿足條件的資料成員會略過繼續走訪 , 直到遇到遇到第一個條件不滿足條件的項目 , 然後將剩餘的項目回傳(包含第一個條件不滿足的項目) .
1. IEnumerable<TSource> SkipWhile<TSource>(this IEnumerable<TSource> source, Func<TSource,Boolean> predicate)	
    - 只要指定的條件為 true，便略過序列中的項目，然後傳回其餘項目。
2. IEnumerable<TSource> SkipWhile<TSource>(this IEnumerable<TSource> source, Func<TSource,Int32,Boolean predicate>)	
    - 只要指定的條件為 true，便略過序列中的項目，然後傳回其餘項目。 項目的索引是用於述詞功能的邏輯中。
- 兩個方法的差別在於 predicate 有沒有傳入 index 參數
- predicate : 判斷忽略的函數 , 若結果為 true 則忽略繼續走訪 ,一旦其結果為 false , 則回傳剩餘的項目.
- SkipWhile : 回傳第一個不符合條件的元素以及在此元素之後的所有集合元素.

#### 使用時機
- 使用條件來進行一次集合**分割**動作
    - 例如 : 條件為年紀大於等於二十 , 資料集合為學生
        ```mermaid
        gantt
            title 使用年紀大於等於二十這個條件分割資料集合
            section 學生
            在第一個遇到年紀小於20的學生左邊:done,des1, 1,3
            在第一個遇到年紀小於20的學生右邊(含):des2, after des1, 5
        ```
        想要**取出藍色部分** , 此時可使用 SkipWhile(年紀大於等於二十) .
#### SkipWhile 的用法
```C#
// 使用字串是否等於 "F" 作為條件
static void Main(string[] args)
{
     string[] alphabets = { "A", "B", "C", "D", "E", "F", "G", "偷懶不想寫" };
     var firstThreeAlphabet = alphabets.SkipWhile(element => element != "F"); // Type is IEnumerable<string>
     foreach (var alphabet in firstThreeAlphabet)
     {
          Console.WriteLine(alphabet);
     }
     Console.ReadKey();
}
```
##### 輸出結果
F
G
偷懶不想寫

```C#
// 使用 index 是否小於 6 作為條件
static void Main(string[] args)
{
     string[] alphabets = { "A", "B", "C", "D", "E", "F", "G", "偷懶不想寫" };
     var firstThreeAlphabet = alphabets.SkipWhile((element, index) => index < 6); // Type is IEnumerable<string>
     foreach (var alphabet in firstThreeAlphabet)
     {
          Console.WriteLine(alphabet);
     }
     Console.ReadKey();
}
```
##### 輸出結果
G
偷懶不想寫

#### 簡單實作自己的 SkipWhile
```C#
public static IEnumerable<TSource> MySkipWhile<TSource>(this IEnumerable<TSource> source, Func<TSource, bool> predicate)
{
     if (source is null || predicate is null) throw new Exception("either soure or predicate is null");
         return MySkipWhileIterator(source, predicate);
}

private static IEnumerable<TSource> MySkipWhileIterator<TSource>(IEnumerable<TSource> source, Func<TSource, bool> predicate)
{
     bool findSpiltPonit = false;
     foreach (var item in source)
     {
          if (!findSpiltPonit && !predicate(item))
          {
               findSpiltPonit = true;
          }

          if (findSpiltPonit)
          {
               yield return item;
          }
     }
}

public static IEnumerable<TSource> MySkipWhile<TSource>(this IEnumerable<TSource> source, Func<TSource, int, bool> predicate)
{
     if (source is null || predicate is null) throw new Exception("either soure or predicate is null");
         return MySkipWhileIterator(source, predicate);
}

private static IEnumerable<TSource> MySkipWhileIterator<TSource>(IEnumerable<TSource> source, Func<TSource, int, bool> predicate)
{
     bool findSpiltPonit = false;
     int index = 0;
     foreach (var item in source)
     {
          if (!findSpiltPonit && !predicate(item, checked(index++)))
          {
               findSpiltPonit = true;
          }

          if (findSpiltPonit)
          {
               yield return item;
          }
     }
}
```

---

### SkipLast
.NET Core 2.0 後 , 才提供的 LINQ 方法 SkipLast(). SkipLast 在順序上與 SkipLast 相反. 由集合的最後一個元素往前記數，再到達指定的數量之前的元素都忽略不算在結果集合中. 若是使用者指定的數量超過資料集合數量則不會回傳任何元素 , 且不會跳出例外.

```C# 
static IEnumerable<TSource> SkipLast(this IEnumerable<TSource> source, int count);
```
- count: 要忽略的元素數量
- SkipLast 回傳忽略指定數量的元素的集合 , 從最後一個元素開始計算.

#### 使用時機
- 使用 count 去取出倒數忽略後的剩餘的資料成員.
    - 用 Where((item,index) => index < collection.size - count) 應該也有一樣的效果

#### SkipLast 的用法
```C#
// 忽略倒數5個元素 , 其餘的印出.
static void Main()
{
     string[] alphabets = { "A", "B", "C", "D", "E", "F", "G", "偷懶不想寫" };
     var firstThreeAlphabet = alphabets.SkipLast(5); // Type is IEnumerable<string>
     foreach (var alphabet in firstThreeAlphabet)
     {
          Console.WriteLine(alphabet);
     }
     Console.ReadKey();
}
```
##### 輸出結果
A
B
C

#### 簡單實作自己的 SkipLast
```C#
public static IEnumerable<TSource> MySkipLast<TSource>(this IEnumerable<TSource> source, int count)
{
     if (source is null)
     {
          throw new Exception("source is null");
     }
     return count <= 0 ? MySkipLastIterator(source, 0) : MySkipLastIterator(source, count);
}

private static IEnumerable<TSource> MySkipLastIterator<TSource>(IEnumerable<TSource> source, int count)
{
     Queue<TSource> queue = new Queue<TSource>();
     foreach (var item in source)
     {
          if (queue.Count == count)
          {
               yield return queue.Dequeue();
          }
          queue.Enqueue(item);
     }
}
```

---

### Summary
1. Take 可以用這張示意圖來做為總結
    - [圖片來源](https://ithelp.ithome.com.tw/articles/10197118)
    - ![](https://i.imgur.com/bYTzrhJ.png)
1. Skip 可以用這張示意圖來做為總結
    - [圖片來源](https://ithelp.ithome.com.tw/articles/10196894)
    - ![](https://i.imgur.com/4ZTVz2Y.png)

1. 有延遲執行的特性 - 使用 foreach 或是使用 GetEnumerator() 時才會執行函式.
1. 沒有查詢運算式 ( SQL like Query syntax )
1. Take 或 TakeLast 指定的 count 數量大於集合數量 , 則傳回完整的集合
1. Take 或 TakeLast 指定的 count 數量小於等於零 , 則傳回空的集合
1. Skip 或 SkipLast 指定的 count 數量大於集合數量，則傳回空集合
1. Skip 或 SkipLast 指定的 count 數量小於等於零，則傳回完整的集合
1. Take 跟 Skip 是互補的 , 以相同的條件叫用 Skip - Take 、 TakeLast - SkipLast 、 TakeWhile - SkipWhile , 兩個結果合在一起的元素集合會是原本的集合

---

### Thank you! 

You can find me on

- [GitHub](https://github.com/s0920832252)
- [Facebook](https://www.facebook.com/fourtune.chen)

若有謬誤 , 煩請告知 , 新手發帖請多包涵

# :100: :muscle: :tada: :sheep: 
