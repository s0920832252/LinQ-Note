---
tags: LinQ, LinQ基礎 , C#
---

# LinQ基礎 - 延遲執行(Deferred Execution)

### 延遲執行的基礎 － 疊代器 & 走訪
詳細請參考下列文章.
- [LINQ基礎 - Iterator(疊代器模式)](https://hackmd.io/vnZcWNdGRCq1cjMJRMJZZA)
- [LINQ基礎 - IEnumerable & IEnumerator](https://hackmd.io/lSm0-BylRdK-MV0AErhHyw)
- [LINQ基礎 - Yield](https://hackmd.io/P_h9ag3ETIOuZMrCq41hWQ?view)

這邊會用一個例子快速回顧 , 以下是自定義類別
```C#
public class CityCollection : IEnumerable
{
     public IEnumerator GetEnumerator()
     {
          Console.WriteLine("item - 1");
          yield return 1;
          Console.WriteLine("item - 2");
          yield return 2;
          Console.WriteLine("item - 3");
          yield return 3;
          Console.WriteLine("item - finish");
     }
}
```
以下是執行 foreach 的程式
```C#
static void Main(string[] args)
{
     var city = new CityCollection();
     Console.WriteLine("foreach start");
     foreach (var item in city)
     {
          Console.WriteLine("foreach item :" + item);
     }
     Console.WriteLine("foreach end");
     Console.ReadKey();
}
```
以下是運行結果
![](https://i.imgur.com/2oVy9cd.png)

使用上面例子 , 利用 Visual Studio 去逐步偵錯, 可以知道 foreach 的執行順序其實是
1. 進入 foreach
    - ![](https://i.imgur.com/WnBR9uY.png)

2. 執行 GetEnumerator() , 得到一個 IEnumerator
    - ![](https://i.imgur.com/w2JMH6f.png)

3. 執行 MoveNext() , 以判斷走訪是否結束. 若尚未結束則將 Current 屬性移動到下一個元素.
    - ![](https://i.imgur.com/KuoCeNo.png)

4. 回傳 Current 屬性給 item (也就是 yield return value; 這一行.)
    - ![](https://i.imgur.com/1q6yoCx.png)

所以不論是 IEnumerable 或是 IEnumerable<T> 都提供一個 GetEnumerator() 方法. 再透過所得到的 Enumerator 物件去執行走訪這個動作.

### 延遲執行的時機

一般來說程式執行到哪一行 , 該行運算式就應該立即被執行. 但 LINQ 有一個很重要的特性 , 叫做「延遲執行」（deferred execution）, 或稱為惰性求值（lazy evaluation）. 顧名思義 , 就是在**需要取用查詢結果的時候，才去執行查詢表示式**. 請看下面範例

自定義 Where 以及 Select
```C#
// 回傳符合條件的項目
public static IEnumerable<TSource> MyWhere<TSource>(this IEnumerable<TSource> source, Func<TSource, bool> predicate)
{
     var iterator = source.GetEnumerator();
     while (iterator.MoveNext())
     {
          if (predicate(iterator.Current))
          {
               yield return iterator.Current;
          }
     }
}
```
```C#
// 將項目轉換成某個樣式
public static IEnumerable<TResult> MySelect<TSource, TResult>(this IEnumerable<TSource> source, Func<TSource, TResult> selector)
{
     var iterator = source.GetEnumerator();
     while (iterator.MoveNext())
     {
          yield return selector(iterator.Current);
     }
}
```
測試程式 - query 會篩選 list 中大於 3 的數字並將其加一.
```C#
List<int> list = new List<int>() { 5, 9, 8 };
IEnumerable<int> query = list.MyWhere(item => item > 3).MySelect(item => item + 1);
list.Remove(9);
foreach (var item in query)
{
     Console.WriteLine(item);
}
```
結果會印出 **6 , 9**

query 若是立即執行查詢的話 , 則結果應該是 **6 , 10 , 9** 才對.
所以查詢時機應該是在 foreach 那一行.
由此可以推測 LinQ 的延遲執行有兩個特性
- **建立查詢**與**執行查詢**的時機是不同的
- **執行查詢**的時機為存取 IEnumerable 中元素的時候.

---

### 延遲執行的運作過程

#### 測試程式
```C#
public static IEnumerable<TResult> MySelect<TSource, TResult>(this IEnumerable<TSource> source, Func<TSource, TResult> selector)
{
     var iterator = source.GetEnumerator();
     while (iterator.MoveNext())
     {
          yield return selector(iterator.Current);
     }
}

public static IEnumerable<TSource> MyWhere<TSource>(this IEnumerable<TSource> source, Func<TSource, bool> predicate)
{
     var iterator = source.GetEnumerator();
     while (iterator.MoveNext())
     {
          if (predicate(iterator.Current))
          {
               yield return iterator.Current;
          }
     }
}

public static IEnumerable<(string Name, int Age)> GetStudent()
{
     yield return (Name: "小王", Age: 15);
     yield return (Name: "大明", Age: 23);
     yield return (Name: "老黃", Age: 39);
}

static void Main(string[] args)
{
     var students = GetStudent();
     var names = students.MyWhere(student => student.Age > 18).MySelect(student => student.Name);
     foreach (var name in names)
     {
          Console.WriteLine(name);
     }
     Console.ReadKey();
}
```

原本我以為下列這行的執行順序是 Where 計算完結果後 , 在繼續執行 Select 如下圖.
```C#
students.MyWhere(student => student.Age>18).MySelect(student => student.Name);
```

```sequence
Note right of IEnumerable: 給予所有學生物件
IEnumerable->Where(Age大於18): Name = "小王", Age = 15
IEnumerable->Where(Age大於18): Name = "大明", Age = 23
IEnumerable->Where(Age大於18): Name = "老黃", Age = 39
Where(Age大於18)->Where(Age大於18): 過濾 : 大於十八歲
Note right of Where(Age大於18): 給予大於十八歲的學生物件
Where(Age大於18)->Select: Name = "大明", Age = 23
Where(Age大於18)->Select: Name = "老黃", Age = 39
Select->Select : 轉換物件為字串
Note right of Select: 給予大於十八歲的學生姓名
Select->輸出結果: Name = "大明"
Select->輸出結果: Name = "老黃"
```
但實際情況卻並非如此 :warning: 
再次使用 Visual Studio 去逐步偵錯可發現執行結果為
1. 開始
- ![](https://i.imgur.com/X296GHF.png)
2. 不斷按下 F11 , 本以為會進入 GetStudent 內 ,但卻一路執行到 foreach. 原因是 students 以及 names 都是 IEnumerable<T> 型別. 在開始走訪前 , 都不會執行敘述.
- ![](https://i.imgur.com/Y6fbIXl.png)
3. 呼叫 GetEnumerator()
- ![](https://i.imgur.com/oplSnGE.png)
4. 執行 MoveNext() , 這裡指的是 names 的下一個. 但有趣的是 names 的下一個是什麼!? names 其實是從 people.Where().Select() 的結果而來的. 所以要走訪 names 就需要知道 Select() 完的結果是什麼. 因為延遲執行 , 所以 names 的 MoveNext() 會呼叫 Select(). 有點 chain 的感覺.
- ![](https://i.imgur.com/sXr4wdM.png)
5. 進入 Select() , 準備開始走訪.
- ![](https://i.imgur.com/ZoXel4D.png)
6. 當我們在 Select() 方法中 , 呼叫 MoveNext() 時會去執行 Where() 的方法內容 , 因為 source 是 Where()的結果 , 所以想要走訪 source , 就需要取得 Where() 的結果.
- ![](https://i.imgur.com/lNVsYra.png)
7. 進入 Where() , 準備開始走訪. 
- ![](https://i.imgur.com/bDZxC0t.png)
8. 同理 , where() 內的 source 是 students , 而 studnets 是來自於 GetStudent() 的結果. 
- ![](https://i.imgur.com/3OpOWFH.png)
9. 進入 GetStudent() 內 , 並回傳第一個結果 , 小王.
- ![](https://i.imgur.com/fc6euJr.png)
10. 回到 Where() , 因為小王不符合 predicate 的條件 , 因此沒進入 if 敘述內. 直接繼續執行 while(). 也就是繼續呼叫 MoveNext().
- ![](https://i.imgur.com/5DjZzy5.png)
11. 取得第二個結果 , 大明.
- ![](https://i.imgur.com/5GlrB2G.png)
12. 再次回到 Where , 並再次讓 predicate 來判斷. 大明符合條件 , 所以進入 if 區域內執行 yield return , 回傳結果.
- ![](https://i.imgur.com/qyBJiRJ.png)
13. 回到 Select , 執行 yield retrun , 回傳 selector() 的結果. 
- ![](https://i.imgur.com/AD0MsSd.png)
14. 回到 main , name 接收到回傳的結果. 
- ![](https://i.imgur.com/5w5x3UT.png)
15. 印出結果**大明** , 之後繼續執行 foreach , 直到 MoveNext() 回傳 false 為止 
- ![](https://i.imgur.com/J1OF8CG.png)

所以實際的執行順序 , 應該如下圖所示 :
```sequence
foreach(走訪查詢結果)->Select : 取資料
Select->Where : 取資料
Where->getStudent() : 取資料
getStudent()->Where : 回傳資料 ("小王", 15)
Where->Where : Age > 18 ? false
Where->getStudent() : 繼續取資料
getStudent()->Where : 回傳資料 ("大明", 23)
Where->Where : Age > 18 ? true
Where->Select : 回傳資料 ("大明", 23)
Select->Select : 經過selector()轉換
Select->foreach(走訪查詢結果) : 回傳 "大明"
foreach(走訪查詢結果)->foreach(走訪查詢結果) : 印出 "大明"
foreach(走訪查詢結果)->Select : 繼續取資料 , 直到取完.
```
---

### 結論

1. 可以透過 yield 關鍵字輕易地完成延遲執行的效果.
2. 走訪的動作 , 其實是透過 IEnumerator 來達成.
3. IEnumerable 型別可以作為資料集合操作.
4. 大部分地 LINQ to Objects API 幾乎都是針對IEnumerable<TSource> 進行擴充.
5. 回傳 IEnumerable 型別代表回傳的結果可以走訪. 但卻不會立即走訪. 會直到執行 MoveNext() , 才會開始走訪到下一個. 這也解釋了設定查詢以及執行查詢的驅動時間點不同的原因. 也就是具有延遲執行的特性.
6. 因為 LinQ 具有延遲執行的特性這代表
    - 設定查詢式後 , 異動來源資料的內容 , 稍後取得查詢結果時是依據最後的資料集合去做查詢的.
    - 撰寫一個查詢式後 , 不論要查詢結果幾次 , 都不需要重新撰寫查詢式. 
7. 使用 LinQ 時需要注意是否需要立刻取得**即時的**查詢結果. 是否介意查詢結果會隨著查詢源的操作改變而不同.

---

### 補充 : 立即執行
雖然 LinQ 的方法有些具有延遲執行的特性 , 但有些則沒有.
像是

1. 轉型類型的 API
    - ToList()
    - ToArray()
    - ToDictionary() 
    - ToHashSet()
    - ToLookUp()
    - ...?
2. 回傳值為單一值
    - Max()
    - Min()
    - Count()
    - First()
    - Last()
    - Single()
    - Average()
    - Sum()
    - ...?

---
### Thank you! 

You can find me on

- [GitHub](https://github.com/s0920832252)
- [Facebook](https://www.facebook.com/fourtune.chen)

若有謬誤 , 煩請告知 , 新手發帖請多包涵

# :100: :muscle: :tada: :sheep: 
