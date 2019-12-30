---
tags: LinQ , C# , Join Operators
---



# Join
在 SQL 中 , 我們可能會有兩張表 , 一張表是個人的資料 , 而另一張表是寵物的資料 , 然後會有一個 ID 來關聯兩張表. 這時如果我們要找某個人有哪些寵物就會使用到 Join 的語法來合併個人以及寵物的資料

Join 有很多種形式 , 如下圖    
![](https://i.imgur.com/susPQqA.png)    
[參考來源](https://dotblogs.com.tw/brooke/2015/03/15/150726)

LinQ 的 Join 是 Inner Join (圖中最中間) ,

### [過載方法](https://docs.microsoft.com/zh-tw/dotnet/api/system.linq.enumerable.join?view=netframework-4.8)
Join 有兩個過載方法 (差別只在於是否傳入自定義 IEqualityComparer)
```C#
public static IEnumerable<TResult> Join<TOuter, TInner, TKey, TResult>(
                this IEnumerable<TOuter> outer,
                IEnumerable<TInner> inner,
                Func<TOuter, TKey> outerKeySelector,
                Func<TInner, TKey> innerKeySelector,
                Func<TOuter, TInner, TResult> resultSelector
);
```
```C#
public static IEnumerable<TResult> Join<TOuter, TInner, TKey, TResult>(
                this IEnumerable<TOuter> outer,
                IEnumerable<TInner> inner,
                Func<TOuter, TKey> outerKeySelector,
                Func<TInner, TKey> innerKeySelector,
                Func<TOuter, TInner, TResult> resultSelector,
                IEqualityComparer<TKey> comparer
);
```
#### 說明
outer : 集合 A
inner : 集合 B
outerKeySelector : 使用委派將 outer 成員的屬性作為 Key1
innerKeySelector : 使用委派將 inner 成員的屬性作為 Key2
resultSelector : 使用委派將 Key1 == Key2 的 outer 成員 & inner 成員 , 轉換成特定結果.
comparer : 比較 Key1 跟 Key2 是否相等的比較器

#### Join 的執行邏輯
1. 依序走訪 inner 所有成員（TInner） , 傳給 innerKeySelecor 產生 TKey , 然後存到一個 Lookup 中.
2. 依序走訪 outer 所有成員（TOuter） , 傳給 outerKeySelector 產生 TKey , 到步驟 1 產生的 Lookup 查詢是否有相同的 TKey. 不符合就重複步驟 2 （繼續走訪 outer 中下一個 TOuter 成員） , 符合就進行步驟 3.
3. 將 TOuter 和 Key 相等的 TInner 作為參數 , 傳給 resultSelector , 轉成 TResult , 回傳給正在走訪的 IEnumerable<TResult> 中.

### Join 的用法
```C#
class Person
{
     public string Name { get; set; }
}

class Pet
{
     public string Name { get; set; }
     public Person Owner { get; set; }
}

static void Main(string[] args)
{
     var people = new List<Person> {
         new Person { Name = "王大明" },
         new Person { Name = "蔡阿高" },
         new Person { Name = "黃飛龍" }
     };

     var pets = new List<Pet> {
         new Pet { Name = "小白", Owner = people[1] },
         new Pet { Name = "小黑", Owner = people[1] },
         new Pet { Name = "小藍", Owner = people[2] },
         new Pet { Name = "小綠", Owner = people[0] }
     };

     var query = people.Join(
                         pets,
                         person => person.Name,
                         pet => pet.Owner.Name,
                         (person, pet) => new { OwnerName = person.Name, Pet = pet.Name }
                 ).ToList();
     query.ForEach(obj => Console.WriteLine($"{obj.OwnerName} {obj.Pet}"));

     Console.WriteLine("@@@@@@@@@@@@@");

     var sqlLikeQuery = (from person in people
                         join pet in pets on person.Name equals pet.Owner.Name
                         select new { OwnerName = person.Name, Pet = pet.Name }
                         ).ToList();
     sqlLikeQuery.ForEach(obj => Console.WriteLine($"{obj.OwnerName} {obj.Pet}"));

     Console.ReadKey();
}
```

##### 輸出結果
王大明 小綠    
蔡阿高 小白    
蔡阿高 小黑    
黃飛龍 小藍    
@@@@@@@@@@@@@    
王大明 小綠    
蔡阿高 小白    
蔡阿高 小黑    
黃飛龍 小藍    

### 簡單實作自己的 Join
```C#
public static IEnumerable<TResult> Join<TOuter, TInner, TKey, TResult>(this IEnumerable<TOuter> outer, IEnumerable<TInner> inner, Func<TOuter, TKey> outerKeySelector, Func<TInner, TKey> innerKeySelector, Func<TOuter, TInner, TResult> resultSelector, IEqualityComparer<TKey> comparer = null)
{
     if (outer is null || inner is null || outerKeySelector is null || innerKeySelector is null || resultSelector is null)
     {
          throw new ArgumentNullException("null");
     }
     return JoinIterator(outer, inner, outerKeySelector, innerKeySelector, resultSelector, comparer);
}

private static IEnumerable<TResult> JoinIterator<TOuter, TInner, TKey, TResult>(IEnumerable<TOuter> outer, IEnumerable<TInner> inner, Func<TOuter, TKey> outerKeySelector, Func<TInner, TKey> innerKeySelector, Func<TOuter, TInner, TResult> resultSelector, IEqualityComparer<TKey> comparer)
{
     Lookup<TKey, TInner> lookup = Lookup<TKey, TInner>.CreateForJoin(inner, innerKeySelector, comparer);
     foreach (TOuter item in outer)
     {
          Lookup<TKey, TInner>.Grouping g = lookup.GetGrouping(outerKeySelector(item), false);
          if (g != null)
          {
               foreach (var groupItem in g)
               {
                    yield return resultSelector(item, groupItem);
               }
          }
     }
}
```


### Summary
- Join 具備延遲執行的特性
- 輸出資料的排序是先看 outer 的順序再看 inner 的順序
- 若沒有傳入客製比較器 , 則使用 Default 的比較器
- Join 是擴充 IEnumerable<TOuter> , 但是回傳值是 IEnumerable<TResult> , 代表輸出序列型別和輸入序列型別可以不相同
- outer、inner、outerKeySelector、innerKeySelector 或 resultSelector 為 null 時 , 執行期間會發生 ArgumentNullException 例外
- 兩個序列 Join 時，是利用 KeySelector 所取出的 TKey 做相等比較，第一個多載方法用 TKey 型別預設比較子，第二個多載方法則使用自訂的相等比較子
- Join 後的輸出序列中的項目 , 其原本 Join 前 , 分別在 outerKeySelector 和 innerKeySelector 取出之 TKey 值必定相等.

### 參考資料
[EqualityComparer<T>.Default 屬性](https://docs.microsoft.com/zh-tw/dotnet/api/system.collections.generic.equalitycomparer-1.default?view=netframework-4.8)     
[Join.cs](https://github.com/dotnet/corefx/blob/master/src/System.Linq/src/System/Linq/Join.cs)

---
### Thank you! 

You can find me on

- [GitHub](https://github.com/s0920832252)
- [Facebook](https://www.facebook.com/fourtune.chen)

若有謬誤 , 煩請告知 , 新手發帖請多包涵

# :100: :muscle: :tada: :sheep: 
