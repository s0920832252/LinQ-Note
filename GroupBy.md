---
tags: LinQ , C# , Grouping Operators
---

# GroupBy
##### 碎碎念
說老實話 , 研讀文件之後 , 我發現 GroupBy 比我本來以為的還要複雜許多... 我以前只會使用最簡單的形式呀 Orz

### 前言
有時候我們會需要將資料依照組別擺放 , 以便日後查詢使用 e.g. 資料內有多個客戶的銷售紀錄 , 將同一個客戶的銷售紀錄存在一個 List<銷售紀錄> 內 , 並使用客戶名作為 Key , 該客戶的 List<銷售紀錄> 作為 value , 存入一個 Dictinary<客戶名,List<銷售紀錄>>.

在不使用 GroupBy 的情況下 , 需要自己走訪每一筆資料 , 並查看該資料使否存在於 Dictinary 內 , 若無 , 則 new 一個 List<T> 並將該資料存入 , 確保該 Key 對應一個 List<T> 在 Dictionary 內. 之後就只要透過 Key 即可查詢對應的 List<T>資料 . 但使用 GroupBy 後 , 則不需要執行上述動作.

一言以蔽之 , GroupBy 可以幫你將資料依照某個條件分群.


##### [GroupBy](https://docs.microsoft.com/zh-tw/dotnet/csharp/programming-guide/concepts/linq/grouping-data)
使用 GroupBy 時需要指定分組的 Key ( 通常是成員的某個屬性 ) , 以此屬性做為分組的依據
![](https://i.imgur.com/4d4pLmU.png)

### [多載](https://docs.microsoft.com/zh-tw/dotnet/api/system.linq.enumerable.groupby?view=netframework-4.8)
依照 elementSelector , resultSelector , comparer 的有無 , 共有八個多載 , 此處使用 comparer 作為劃分 , 分成四組.
```C#
public static IEnumerable<IGrouping<TKey,TSource>> GroupBy<TSource,TKey> (
    this IEnumerable<TSource> source, 
    Func<TSource,TKey> keySelector
)

public static IEnumerable<IGrouping<TKey,TSource>> GroupBy<TSource,TKey> (
    this IEnumerable<TSource> source, 
    Func<TSource,TKey> keySelector, 
    IEqualityComparer<TKey> comparer
)
```
- 傳入參數
    - keySelector : 指定以甚麼屬性作為分組的依據
    - comparer : 自定義比較器 , 用來比較兩個 key 是否相同. 以決定是否分在同一組
- 回傳值
    - 型態 : IEnumerable<IGrouping<TKey, **TSource**>>
    - 由型態可知 , 回傳值是 IGrouping<TKey, TSource> 的集合
    - IGrouping<TKey, TSource> 是分組後的資料 , 每一個 IGrouping 會有一個
    TKey Key 以及與該 Key 相對應的 TSource Value.
- 總結 : 
    1. 使用 keySelector 設定要使用哪一個 Key 作為資料集合分組的依據 , 然後以此依據分組並輸出成分組後資料的資料集合 IEnumerable<IGrouping<TKey, TSource>> 
    2. 此兩個方法的差在只在於是否使用自定義比較器 , 若不使用 , 會使用[預設比較器](https://docs.microsoft.com/en-us/dotnet/api/system.collections.generic.comparer-1.default?view=netframework-4.8)。
```C#
public static IEnumerable<IGrouping<TKey,TElement>> GroupBy<TSource,TKey,TElement> (
    this IEnumerable<TSource> source, 
    Func<TSource,TKey> keySelector, 
    Func<TSource,TElement> elementSelector
)

public static IEnumerable<IGrouping<TKey,TElement>> GroupBy<TSource,TKey,TElement> (
    this IEnumerable<TSource> source,
    Func<TSource,TKey> keySelector,
    Func<TSource,TElement> elementSelector, 
    IEqualityComparer<TKey> comparer
)
```
- 傳入參數
    - keySelector : 指定以甚麼屬性作為分組的依據
    - comparer : 自定義比較器 , 用來比較兩個 key 是否相同. 以決定是否分在同一組
    - elementSelector : 決定資料成員最後回傳的結果 , 可以想成是 Select .         
- 回傳值    
    - 型態 : IEnumerable<IGrouping<TKey,**TElement**>> 
    - 由型態可知 , 回傳值是 IGrouping<TKey, TElement> 的集合
- 總結 : 
    - 上一組沒有 elementSelector  , 所以其預設是回傳資料成員自身 , 並無任何轉換. 而此組因為其回傳值會透過 elementSelector 轉換後才回傳 , 所以回傳值型態為 IEnumerable<IGrouping<TKey,TElement>>

> 有時候 , 若想要直接拿到 Group 的某項數值而非 Group 時 , 但以上四個多載 , 其回傳值型態都是 IGrouping 的集合. 因此還必須再走訪該 Group 成員去計算結果. 此時可考慮使用以下四種多載
```C#
public static IEnumerable<TResult> GroupBy<TSource,TKey,TResult> (
    this IEnumerable<TSource> source, 
    Func<TSource,TKey> keySelector, 
    Func<TKey,IEnumerable<TSource>,TResult> resultSelector
)

public static IEnumerable<TResult> GroupBy<TSource,TKey,TResult> (
    this IEnumerable<TSource> source,
    Func<TSource,TKey> keySelector, 
    Func<TKey,IEnumerable<TSource>,TResult> resultSelector, 
    IEqualityComparer<TKey> comparer
)
```
- 傳入參數
    - keySelector : 指定以甚麼屬性作為分組的依據
    - comparer : 自定義比較器 , 用來比較兩個 key 是否相同. 以決定是否分在同一組
    - resultSelector : 決定分組後的每一組資料集合應該如何轉換成某個結果並回傳.         
- 回傳值    
    - 型態 : IEnumerable<TResult>
- 總結 : 
    - 回傳值不再是 IEnumerable<IGrouping<,>> , 因為 IGrouping 已經被 resultSelector 轉換成 TResult .

```C#
public static IEnumerable<TResult> GroupBy<TSource,TKey,TElement,TResult> (
    this IEnumerable<TSource> source,
    Func<TSource,TKey> keySelector,
    Func<TSource,TElement> elementSelector,
    Func<TKey,IEnumerable<TElement>,TResult> resultSelector
)

public static IEnumerable<TResult> GroupBy<TSource,TKey,TElement,TResult> (
    this IEnumerable<TSource> source,   
    Func<TSource,TKey> keySelector, 
    Func<TSource,TElement> elementSelector,
    Func<TKey,IEnumerable<TElement>,TResult> resultSelector, 
    IEqualityComparer<TKey> comparer
)
```
- 傳入參數
    - keySelector : 指定以甚麼屬性作為分組的依據
    - comparer : 自定義比較器 , 用來比較兩個 key 是否相同. 以決定是否分在同一組
    - elementSelector : 決定資料成員傳給 resultSelector 的結果. 
    - resultSelector : 決定分組後的每一組資料集合應該如何轉換成某個結果並回傳.      
- 回傳值    
    - 型態 : IEnumerable<TResult>

### GroupBy 的使用方式
```C#
static void Main(string[] args)
{
     List<(string petName, double petAge)> petCollection = new List<(string petName, double petAge)>()
     {
          (petName:"小豬", petAge:5.1),
          (petName:"大黃", petAge:5.9),
          (petName:"小龜", petAge:5.3),
          (petName:"小牛", petAge:4.3),
          (petName:"小馬", petAge:4.9),
          (petName:"小龍", petAge:7),
     };

     var queryResult = petCollection.GroupBy(pet => Math.Floor(pet.petAge));
     foreach (var petGroup in queryResult)
     {
          Console.WriteLine($"使用的 key 是 {petGroup.Key}");
          foreach (var (petName, petAge) in petGroup)
          {
                Console.WriteLine($"寵物名 : {petName} 寵物年紀 : {petAge}");
          }
          Console.WriteLine("=====================================");
     }
     Console.ReadKey();
}
```
##### 輸出結果
![](https://i.imgur.com/eg0fgKj.png)

```C#
static void Main(string[] args)
{
     List<(string petName, double petAge)> petCollection = new List<(string petName, double petAge)>()
            {
                (petName:"小豬", petAge:5.1),
                (petName:"大黃", petAge:5.9),
                (petName:"小龜", petAge:5.3),
                (petName:"小牛", petAge:4.3),
                (petName:"小馬", petAge:4.9),
                (petName:"小龍", petAge:7),
            };

            var queryResult = petCollection.GroupBy(pet => Math.Floor(pet.petAge), pet => $"寵物名稱是 {pet.petName} , 寵物年紀是 {pet.petAge}");
            foreach (var petGroup in queryResult)
            {
                Console.WriteLine($"使用的 key 是 {petGroup.Key}");
                foreach (var petStr in petGroup)
                {
                    Console.WriteLine(petStr);
                }
                Console.WriteLine("=====================================");
            }
            Console.ReadKey();
        }
```
##### 輸出結果
![](https://i.imgur.com/2Ewg17z.png)

```C#
static void Main(string[] args)
{
     List<(string petName, double petAge)> petCollection = new List<(string petName, double petAge)>()
     {
          (petName:"小豬", petAge:5.1),
          (petName:"大黃", petAge:5.9),
          (petName:"小龜", petAge:5.3),
          (petName:"小牛", petAge:4.3),
          (petName:"小馬", petAge:4.9),
          (petName:"小龍", petAge:7),
     };

     var queryResult = petCollection.GroupBy(pet => Math.Floor(pet.petAge), 
                                             (key, petGroup) =>
                                                   ( Key: key,
                                                     PetCount: petGroup.Count(),
                                                     PetAverage: petGroup.Average(pet => pet.petAge),
                                                     PetSum: petGroup.Sum(pet => pet.petAge)
                                                   )
                       );
     foreach (var (Key, PetCount, PetAverage, PetSum) in queryResult)
     {
          Console.WriteLine($"key : {Key}");
          Console.WriteLine($"Count : {PetCount}");
          Console.WriteLine($"Average : {PetAverage}");
          Console.WriteLine($"Sum : {PetSum}");
          Console.WriteLine("=====================================");
     }
     Console.ReadKey();
}
```
##### 輸出結果
![](https://i.imgur.com/1m9jONZ.png)

```C#
static void Main(string[] args)
{
     List<(string petName, double petAge)> petCollection = new List<(string petName, double petAge)>()
     {
          (petName:"小豬", petAge:5.1),
          (petName:"大黃", petAge:5.9),
          (petName:"小龜", petAge:5.3),
          (petName:"小牛", petAge:4.3),
          (petName:"小馬", petAge:4.9),
          (petName:"小龍", petAge:7),
     };

     var queryResult = petCollection.GroupBy(pet => Math.Floor(pet.petAge),
                                             pet => pet.petAge, 
                                             (key, ages) =>
                                                 ( Key: key,
                                                   PetCount: ages.Count(),
                                                   PetAverage: ages.Average(),
                                                   PetSum: ages.Sum()
                                                 )
                       );
     foreach (var (Key, PetCount, PetAverage, PetSum) in queryResult)
     {
          Console.WriteLine($"key : {Key}");
          Console.WriteLine($"Count : {PetCount}");
          Console.WriteLine($"Average : {PetAverage}");
          Console.WriteLine($"Sum : {PetSum}");
          Console.WriteLine("=====================================");
     }
     Console.ReadKey();
}
```
##### 輸出結果
![](https://i.imgur.com/JyIQaa5.png)



### 簡單實作自己的 GroupBy
會使用到 Lookup , 請參考這邊文章
[LinQ基礎 - Lookup](https://hackmd.io/yMO0aHHPQKm61t0ceAeDCQ)
```C#
public static IEnumerable<IGrouping<TKey, TSource>> MyGroupBy<TSource, TKey>(this IEnumerable<TSource> source, Func<TSource, TKey> keySelector)
{
     return Lookup<TKey, TSource>.Create<TSource>(source, keySelector, (element) => element, null);
}

public static IEnumerable<IGrouping<TKey, TSource>> MyGroupBy<TSource, TKey>(this IEnumerable<TSource> source, Func<TSource, TKey> keySelector, IEqualityComparer<TKey> comparer)
{
     return Lookup<TKey, TSource>.Create<TSource>(source, keySelector, (element) => element, comparer);
}

public static IEnumerable<IGrouping<TKey, TElement>> MyGroupBy<TSource, TKey, TElement>(this IEnumerable<TSource> source, Func<TSource, TKey> keySelector, Func<TSource, TElement> elementSelector)
{
     return Lookup<TKey, TElement>.Create<TSource>(source, keySelector, elementSelector, null);
}

public static IEnumerable<IGrouping<TKey, TElement>> MyGroupBy<TSource, TKey, TElement>(this IEnumerable<TSource> source, Func<TSource, TKey> keySelector, Func<TSource, TElement> elementSelector, IEqualityComparer<TKey> comparer)
{
     return Lookup<TKey, TElement>.Create<TSource>(source, keySelector, elementSelector, comparer);
}

public static IEnumerable<TResult> MyGroupBy<TSource, TKey, TResult>(this IEnumerable<TSource> source, Func<TSource, TKey> keySelector, Func<TKey, IEnumerable<TSource>, TResult> resultSelector)
{
     Lookup<TKey, TSource> lookup = Lookup<TKey, TSource>.Create<TSource>(source, keySelector, (element) => element, null);
     return lookup.ApplyResultSelector(resultSelector);
}

public static IEnumerable<TResult> MyGroupBy<TSource, TKey, TElement, TResult>(this IEnumerable<TSource> source, Func<TSource, TKey> keySelector, Func<TSource, TElement> elementSelector, Func<TKey, IEnumerable<TElement>, TResult> resultSelector)
{
     Lookup<TKey, TElement> lookup = Lookup<TKey, TElement>.Create<TSource>(source, keySelector, elementSelector, null);
     return lookup.ApplyResultSelector(resultSelector);
}

public static IEnumerable<TResult> MyGroupBy<TSource, TKey, TResult>(this IEnumerable<TSource> source, Func<TSource, TKey> keySelector, Func<TKey, IEnumerable<TSource>, TResult> resultSelector, IEqualityComparer<TKey> comparer)
{
     Lookup<TKey, TSource> lookup = Lookup<TKey, TSource>.Create<TSource>(source, keySelector, (element) => element, comparer);
     return lookup.ApplyResultSelector(resultSelector);
}

public static IEnumerable<TResult> MyGroupBy<TSource, TKey, TElement, TResult>(this IEnumerable<TSource> source, Func<TSource, TKey> keySelector, Func<TSource, TElement> elementSelector, Func<TKey, IEnumerable<TElement>, TResult> resultSelector, IEqualityComparer<TKey> comparer)
{
     Lookup<TKey, TElement> lookup = Lookup<TKey, TElement>.Create<TSource>(source, keySelector, elementSelector, comparer);
     return lookup.ApplyResultSelector(resultSelector);
}
```




### 總結

### 參考
[[C#] ToLookup, GroupBy, ToDictionary簡單介紹](https://dotblogs.com.tw/kirkchen/2011/07/16/toolookup_groupby_todicionary_introduction)
[Grouping.cs](https://github.com/dotnet/runtime/blob/master/src/libraries/System.Linq/src/System/Linq/Grouping.cs)
[Enumerable.cs](https://github.com/microsoft/referencesource/blob/master/System.Core/System/Linq/Enumerable.cs)
[C#的利器LINQ-GroupBy的原碼探索](https://ithelp.ithome.com.tw/articles/10196274)
[C#的利器LINQ-GroupBy的應用](https://ithelp.ithome.com.tw/articles/10196181)
[[C#]LINQ–GroupBy 群組](https://kw0006667.wordpress.com/2013/05/31/clinqgroupby-%E7%BE%A4%E7%B5%84/)
[利用LINQ GroupBy快速分組歸類](https://blog.darkthread.net/blog/linq-groupby-todictionary-grouping/)
[C# LINQ: GroupBy](https://jasper-it.blogspot.com/2015/01/c-linq-groupby.html)



### Thank you! 

You can find me on

- [GitHub](https://github.com/s0920832252)
- [Facebook](https://www.facebook.com/fourtune.chen)

若有謬誤 , 煩請告知 , 新手發帖請多包涵

# :100: :muscle: :tada: :sheep: 
