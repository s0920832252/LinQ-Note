---
tags: LinQ , C# , Aggregate Operators
---

# Aggregate

### [Aggregate](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/linq/aggregation-operations)

> An aggregation operation computes a single value from a collection of values.

![](https://i.imgur.com/WJruW0n.png)

Aggregate 的意思是加總的、聚合的. 也就是說 Aggregate 會將集合元素透過若干處理**合併**為一個結果 , 並回傳.

Aggregate 方法會走訪每一個元素. 在每次元素拜訪結束後 , 會將計算結果暫存起來 , 用作與下一個 Current Item 進行結合、處理或是使用者指定的運算. 這意思是說前一篇介紹的 Sum、Average、Count、Min 以及 Max 其實都只是 Aggregate 的一種特殊情況之一. 

以 Sum 為例 : 使用者透過傳入委派指定加法作為合併手段 , 因此 Aggregate 會走訪每一個元素 , 將當前拜訪的元素與上次的暫存結果加總起來 , 做為下一次的暫存結果.
其他的方法則以此類推.

### [多載型式](https://docs.microsoft.com/zh-tw/dotnet/api/system.linq.enumerable.aggregate?view=netframework-4.8)

1. TResult Aggregate<TSource,TAccumulate,TResult> (this IEnumerable<TSource> source, TAccumulate seed, Func<TAccumulate,TSource,TAccumulate> func, Func<TAccumulate,TResult> resultSelector)	
    - 將累加函式套用到序列上。 使用指定的值做為初始累加值，並使用指定的函式來選取結果值。
2. TAccumulate Aggregate<TSource,TAccumulate> (this IEnumerable<TSource> source, TAccumulate seed, Func<TAccumulate,TSource,TAccumulate> func)
    - 將累加函式套用到序列上。 使用指定的初始值做為初始累加值。
3. TSource Aggregate<TSource> (this IEnumerable<TSource> source, Func<TSource,TSource,TSource> func)	
    - 將累加函式套用到序列
    
#### 名詞解釋
1. source : 欲走訪的資料集合
2. seed : 初始的累加值
3. func : 處理資料的方法
    - 第一個參數 : 上一次資料處理的結果
    - 第二個參數 : 目前走訪的元素
    - 回傳值 : 將上一次資料處理的結果以及目前走訪的元素做某種資料處理後的結果.
4. resultSelector : 最後的結果 , 會經過 resultSelector 轉換後才回傳.

Aggregate 這個方法其實就是把上一筆資料處理的結果跟目前的元素傳入 func 處理後 , 再將結果丟到下一個元素再做處理，最後就會得到一個唯一的結果
    
### 使用時機
- 需要走訪所有的成員 , 以得到一個計算後的結果.
- 如何決定使用哪一個多載 !?
    - 是否需要將 Seed 這個預設累加值傳入 
    - 最後的結果是否需要透過 resultSelector 轉換後再回傳.
### Aggregate 的用途
##### 使用的資料集合
```C#
public static string[] licenseArray = new string[] { "5918", "A3D5", "W555", "1Q5E" };

public static int[] nums = new int[] { 5, 2, 3, 4 };

public static IEnumerable<(string name, int age)> GetPeople()
{
     yield return (name: "小王", age: 15);
     yield return (name: "老黃", age: 31);
     yield return (name: "阿高", age: 74);
}
```
##### 示範程式
```C#
// 使用多載形式 3
// 將字串陣列中的成員串成一個字串
// 此處使用 sting.Join 恰好也有相同的結果.
string license = licenseArray.Aggregate((l, r) => l + r);
Console.WriteLine(license);
string licenseFromStringJoin = string.Join("", licenseArray);
Console.WriteLine(licenseFromStringJoin);

// 計算乘積
int product = nums.Aggregate((l, r) => l * r);
Console.WriteLine(product);


var people = GetPeople();
// 使用多載形式 2
// 同時取出人群中最大的年紀以及最小的年紀.
var (MinAge, MaxAge) = people.Aggregate((MinAge: int.MaxValue, MaxAge: int.MinValue), (result, element) =>
{
     result.MaxAge = result.MaxAge > element.age ? result.MaxAge : element.age;
     result.MinAge = result.MinAge < element.age ? result.MinAge : element.age;
     return result;
});
Console.WriteLine($"最大年紀 : {MaxAge}  ,  最小年紀 : {MinAge}");

// 同時取出最大的年紀以及該人姓名.
var (name, age) = people.Aggregate((name: "", age: int.MinValue), (result, element) =>
{
     return result.age > element.age ? result : element; ;
});
Console.WriteLine($"名字 : {name} , 最大年紀是 {age} 歲");


// 使用多載形式 1
// 取出最大的年紀以及該人姓名後 , 使用這兩樣資訊轉換成一個 comment 字串.
var comment = people.Aggregate((name: "", age: int.MinValue), (result, element) =>
{
     return result.age > element.age ? result : element; ;
}, (finalResult) => $"名字 : {finalResult.name} , 最大年紀是 {finalResult.age} 歲");
Console.WriteLine(comment);
```
##### 輸出結果
![](https://i.imgur.com/fc56v7g.png)


### 實作自己的 Aggregate
```C#
public static TSource MyAggregate<TSource>(this IEnumerable<TSource> source, Func<TSource, TSource, TSource> func)
{
     if (source is null || func is null) { throw new Exception("null Error"); }
     var enumerator = source.GetEnumerator();
     if (!enumerator.MoveNext()) { throw new Exception("not element"); }

     var result = enumerator.Current;
     while (enumerator.MoveNext())
     {
          result = func(result, enumerator.Current);
     }
     return result;
}

public static TAccumulate MyAggregate<TSource, TAccumulate>(this IEnumerable<TSource> source, TAccumulate seed, Func<TAccumulate, TSource, TAccumulate> func)
{
     if (source is null || func is null) { throw new Exception("null Error"); }

     var result = seed;
     foreach (var item in source)
     {
          result = func(result, item);
     }
     return result;
}

public static TResult MyAggregate<TSource, TAccumulate, TResult>(this IEnumerable<TSource> source, TAccumulate seed, Func<TAccumulate, TSource, TAccumulate> func, Func<TAccumulate, TResult> resultSelector)
{
     if (source is null || func is null || resultSelector is null) { throw new Exception("null Error"); }

     var result = seed;
     foreach (var item in source)
     {
          result = func(result, item);
     }
     return resultSelector(result);
}
```

---

### 總結

1. 實務上 , 也許你有可能碰到需要同時取得一集合的平均值 , 總和 .
請記得不要分別各使用 Sum() 以及 Average() 一次 . 而是使用 Aggregate() 代替.
因為兩者的複雜度是 O(2n) 以及 O(n)
2. 也許你會懷疑為何不直接使用 foreach 就好 , 兩者明明在使用上相差無多. 我個人習慣使用的情境如下 
    - 需要採取的動作很簡單 . 例如將集合內字串元素**合併**為一個字串. 此時使用 foreach 會顯得累贅. 但用 Aggregate() 則只需要一行即可得到結果.
    - 與其他 LinQ 方法搭配的時候. 
        - 例如 var result = Collection.Where(篩選條件).Aggregate()
    


---












### Thank you! 

You can find me on

- [GitHub](https://github.com/s0920832252)
- [Facebook](https://www.facebook.com/fourtune.chen)

若有謬誤 , 煩請告知 , 新手發帖請多包涵

# :100: :muscle: :tada: :sheep: 
