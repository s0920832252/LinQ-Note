---
tags: LinQ , C# , Element Operators
---

# First、Single、Last、ElementAt

LINQ 中有設計一組專門用來取出來源序列中**單一**項目值的擴充方法 , 也就是  **First、Single、Last、ElementAt** . 它們也都搭配了一組取不到資料就輸出來源資料型別預設值的方法 : **FirstOrDefault、LastOrDefault、ElementAtOrDefault、SingleOrDefault**. 值得注意的是 , 這八個方法都是立即執行（Immediately execution）查詢. 因此不用擔心延遲執行的問題.



### [First](https://docs.microsoft.com/zh-tw/dotnet/api/system.linq.enumerable.first?view=netframework-4.8)
First 算是 LinQ 取值類擴充方法中很常使用的方法 , 它會回傳來源序列中的第一個元素或符合條件的第一個元素. 若序列中不存在符合條件的元素 , 它會拋出例外. 它有兩個多載方法如下

1. First<TSource>(this IEnumerable<TSource>)	
    - Returns the first element of a sequence.

2. First<TSource>(this IEnumerable<TSource>, **Func<TSource,Boolean**>)	
    - Returns the first element in a sequence that **satisfies a specified condition.**

#### 使用時機
- 當你期望滿足你想要的條件的項目存在**複數(大於等於一)個**在集合中 , 然後你只想要取出第一個達成你條件的項目 ,  這時候可以使用 First.

#### First 的用處
取出序列中第一個項目 、 以及第一個年紀大於七十的項目
```C#
public static IEnumerable<(string name, int age)> GetPeople()
{
     yield return (name: "小王", age: 15);
     yield return (name: "老黃", age: 31);
     yield return (name: "阿高", age: 74);
}
```
```C#
static void Main(string[] args)
{
     var people = GetPeople();
     var queryFirst = people.First();
     Console.WriteLine(queryFirst);
     var queryResult = people.First(person => person.age > 70);
     Console.WriteLine(queryResult);
     Console.ReadKey();
}
```
##### 輸出結果
(小王 , 15)
(阿高 , 74)

#### 簡單實作自己的 First
```C#
public static TSource MyFirst<TSource>(this IEnumerable<TSource> source)
{
     if (source == null)
          throw new Exception("null");

     var e = source.GetEnumerator();
     return e.MoveNext() ? e.Current : throw new Exception("not found");
}

public static TSource MyFirst<TSource>(this IEnumerable<TSource> source, Func<TSource, bool> predicate)
{
     if (source == null || predicate == null)
          throw new Exception("null");

     foreach (var item in source)
     {
          if (predicate(item))
          {
               return item;
          }
     }
     throw new Exception("not found");
}
```

---

### [FirstOrDefault](https://docs.microsoft.com/zh-tw/dotnet/api/system.linq.enumerable.firstordefault?view=netframework-4.8)
FirstOrDefault 也是 LinQ 取值類擴充方法中常使用的方法 , 它會回傳來源序列中的第一個元素或符合條件的第一個元素. 若序列中不存在符合條件的元素 , 它會**回傳該集合元素型別的預設值**. 它有兩個多載方法如下

1. FirstOrDefault<TSource>(this IEnumerable<TSource>)	
    - Returns the first element of a sequence, or a default value if the sequence contains no elements.
2. FirstOrDefault<TSource>(this IEnumerable<TSource>, Func<TSource,Boolean>)	
    - Returns the first element of the sequence that satisfies a condition or a default value if no such element is found.

#### 使用時機
- 當你期望滿足你想要的條件的項目存在**複數(大於等於一)個**在集合中 , 然後你只想要取出第一個達成你條件的項目 , 且**你不想要處理 try catch , 想自己處理若是項目不存在於集合中的狀況的時候** , 這時候可以使用 FirstOrDefault.

#### FirstOrDefault 的用處
取出序列中第一個項目 、 第一個年紀大於七十的項目以及第一個年紀大於八十的項目( 若不存在 , 不要拋出例外 , 而是回傳預設值 )
```C#
public static IEnumerable<(string name, int age)> GetPeople()
{
     yield return (name: "小王", age: 15);
     yield return (name: "老黃", age: 31);
     yield return (name: "阿高", age: 74);
}
```
```C#
static void Main(string[] args)
{
     var people = GetPeople();
     var queryFirst = people.FirstOrDefault();
     Console.WriteLine(queryFirst);
     var queryResult = people.FirstOrDefault(person => person.age > 70);
     Console.WriteLine(queryResult);
     var queryNotFound = people.FirstOrDefault(person => person.age > 80);
     Console.WriteLine(queryNotFound);
     Console.ReadKey();
}
```

##### 輸出結果
(小王 , 15)
(阿高 , 74)
( null , 0 )

#### 簡單實作自己的 FirstOrDefault
```C#
public static TSource MyFirstOrDefault<TSource>(this IEnumerable<TSource> source)
{
     if (source == null)
          throw new Exception("null");

     var e = source.GetEnumerator();
     return e.MoveNext() ? e.Current : default;
}

public static TSource MyFirstOrDefault<TSource>(this IEnumerable<TSource> source, Func<TSource, bool> predicate)
{
     if (source == null || predicate == null)
          throw new Exception("null");

     foreach (var item in source)
     {
          if (predicate(item))
          {
               return item;
          }
     }
     return default; // default 是 C# 的關鍵字 , 可以取出型態的預設值
}
```

---

### [Last](https://docs.microsoft.com/zh-tw/dotnet/api/system.linq.enumerable.last?view=netframework-4.8)

與 First 的用法完全相同 , 差別只在於 Last 會回傳來源序列中的最後一個元素或符合條件的最後一個元素.

#### 使用時機
- 當你期望滿足你想要的條件的項目存在複數(大於等於一)個在集合中 , 然後你只想要取出最後一個達成你條件的項目 , 這時候可以使用 Last.

#### Last 的用處
取出序列中最後一個項目 、 以及最後一個年紀大於七十的項目
```C#
public static IEnumerable<(string name, int age)> GetPeople()
{
     yield return (name: "小王", age: 15);
     yield return (name: "老黃", age: 31);
     yield return (name: "阿高", age: 74);
}
```
```C#
static void Main(string[] args)
{
     var people = GetPeople();
     var queryLast = people.Last();
     Console.WriteLine(queryLast);
     var queryResult = people.Last(person => person.age > 70);
     Console.WriteLine(queryResult);
     Console.ReadKey();
}
```

##### 輸出結果
(阿高 , 74)
(阿高 , 74)

#### 簡單實作自己的 Last
```C#
public static TSource MyLast<TSource>(this IEnumerable<TSource> source)
{
     if (source == null)
          throw new Exception("null");

     var e = source.GetEnumerator();
     TSource result = default;
     bool isFind = false;
     while (e.MoveNext())
     {
          result = e.Current;
          isFind = true;
     }
     return isFind ? result : throw new Exception("not found");
}

public static TSource MyLast<TSource>(this IEnumerable<TSource> source, Func<TSource, bool> predicate)
{
     if (source == null || predicate == null)
          throw new Exception("null");

     TSource result = default;
     bool isFind = false;
     foreach (var item in source)
     {
          if (predicate(item))
          {
               result = item;
               isFind = true;
          }
     }
     return isFind ? result : throw new Exception("not found");
}
```

---

### LastOrDefault
與 FirstOrDefault 的用法完全相同 , 差別只在於 LastOrDefault 會回傳來源序列中的最後一個元素或符合條件的最後一個元素.
#### 使用時機
- 當你期望滿足你想要的條件的項目存在複數(大於等於一)個在集合中 , 然後你只想要取出最後一個達成你條件的項目 , 且你不想要處理 try catch , 想自己處理若是項目不存在於集合中的狀況的時候 , 這時候可以使用 LastOrDefault.
#### LastOrDefault 的用處
取出序列中最後一個項目 、 最後一個年紀大於七十的項目以及最後一個年紀大於八十的項目( 若不存在 , 不要拋出例外 , 而是回傳預設值 )
```C#
public static IEnumerable<(string name, int age)> GetPeople()
{
     yield return (name: "小王", age: 15);
     yield return (name: "老黃", age: 31);
     yield return (name: "阿高", age: 74);
}
```
```C#
static void Main(string[] args)
{
     var people = GetPeople();
     var queryLast = people.LastOrDefault();
     Console.WriteLine(queryLast);
     var queryResult = people.LastOrDefault(person => person.age > 70);
     Console.WriteLine(queryResult);
     var queryNotFound = people.LastOrDefault(person => person.age > 80);
     Console.WriteLine(queryNotFound);

     Console.ReadKey();
}
```
##### 輸出結果
(阿高 , 74)
(阿高 , 74)
( null , 0 )

#### 簡單實作自己的 LastOrDefault
```C#
public static TSource MyLastOrDefault<TSource>(this IEnumerable<TSource> source)
{
     if (source == null)
         throw new Exception("null");

     var e = source.GetEnumerator();
     TSource result = default;
     bool isFind = false;
     while (e.MoveNext())
     {
          result = e.Current;
          isFind = true;
     }
     return isFind ? result : default;
}

public static TSource MyLastOrDefault<TSource>(this IEnumerable<TSource> source, Func<TSource, bool> predicate)
{
     if (source == null || predicate == null)
         throw new Exception("null");

     TSource result = default;
     bool isFind = false;
     foreach (var item in source)
     {
          if (predicate(item))
          {
               result = item;
               isFind = true;
          }
     }
     return isFind ? result : default;
}
```
---

### [Single](https://docs.microsoft.com/en-us/dotnet/api/system.linq.enumerable.single?view=netframework-4.8)

Single 會回傳來源序列中唯一的元素或唯一一個符合條件的元素. 若序列中**不存在符合條件的元素**或是**超過一個符合條件的元素** , 它會拋出例外. 它有兩個多載方法如下

1. Single<TSource>(this IEnumerable<TSource>)	
    - Returns the only element of a sequence, and throws an exception if there is not exactly one element in the sequence.
    
2. Single<TSource>(this IEnumerable<TSource>, Func<TSource,Boolean>)	
    - Returns the only element of a sequence that satisfies a specified condition, and throws an exception if more than one such element exists.

#### 使用時機

- 用來取出集合中唯一的或是唯一符合條件的元素. 會使用的時機通常取決於是否在乎**是不是集合中唯一**的這個限制.

#### Single 的用處
分別使用兩個多載形式去取出 唯一一個年紀大於七十的項目.

```C#
public static IEnumerable<(string name, int age)> GetPeople()
{
     yield return (name: "小王", age: 15);
     yield return (name: "老黃", age: 31);
     yield return (name: "阿高", age: 74);
}
```
```C#
static void Main(string[] args)
{
     var people = GetPeople();
     // logically , querySingle and queryResult both filter the values to be returned based on the predicate.
     // but queryResult have better performance than querySingle.
     var querySingle = people.Where(person => person.age > 70).Single();
     var queryResult = people.Single(person => person.age > 70);
     Console.WriteLine(querySingle);
     Console.WriteLine(queryResult);
     Console.ReadKey();
}
```
##### 輸出結果
(阿高 , 74)
(阿高 , 74)

#### 簡單實作自己的 Single
```C#
public static TSource MySingle<TSource>(this IEnumerable<TSource> source)
{
     if (source == null)
          throw new Exception("null");

     var e = source.GetEnumerator();
     if (!e.MoveNext()) throw new Exception("Not Element");
     var result = e.Current;
     return e.MoveNext() ? throw new Exception("More than one") : result;
}

public static TSource MySingle<TSource>(this IEnumerable<TSource> source, Func<TSource, bool> predicate)
{
     if (source == null || predicate == null)
          throw new Exception("null");
          
     int count = 0;
     TSource result = default;
     foreach (var item in source)
     {
          if (predicate(item))
          {
               result = item;
               count++;
          }
     }
     switch (count)
     {
          case 0: throw new Exception("Not Element");
          case 1: return result;
          default: throw new Exception("More than one");
     }
}
```

---

### [SingleOrDefault](https://docs.microsoft.com/zh-tw/dotnet/api/system.linq.enumerable.singleordefault?view=netframework-4.8)

SingleOrDefault 會回傳來源序列中唯一的元素或唯一一個符合條件的元素.
1. 序列中不存在符合條件的元素 , 它會**回傳預設值**. 
2. 序列中超過一個符合條件的元素 , 它會超出例外. 

它有兩個多載方法如下
1. SingleOrDefault<TSource>(thisIEnumerable<TSource>)	
    - Returns the only element of a sequence, or a default value if the sequence is empty; this method throws an exception if there is more than one element in the sequence.

2. SingleOrDefault<TSource>(this IEnumerable<TSource>, Func<TSource,Boolean>)	
    - Returns the only element of a sequence that satisfies a specified condition or a default value if no such element exists; this method throws an exception if more than one element satisfies the condition.
    
#### 使用時機
- 用來取出集合中唯一的或是唯一符合條件的元素. 會使用的時機通常取決於是否在乎是不是集合中唯一的這個限制. 
#### SingleOrDefault 的用處
分別使用兩個多載形式去取出 唯一一個年紀大於七十的項目.
```C#
static void Main(string[] args)
{
     var people = GetPeople();
     // logically , querySingle and queryResult both filter the values to be returned based on the predicate.
     // but queryResult have better performance than querySingle.
     var querySingle = people.Where(person => person.age > 70).SingleOrDefault();
     var queryResult = people.SingleOrDefault(person => person.age > 70);
     Console.WriteLine(querySingle);
     Console.WriteLine(queryResult);
     Console.ReadKey();
}
```

#### 簡單實作自己的 SingleOrDefault
```C#
public static TSource MySingleOrDefault<TSource>(this IEnumerable<TSource> source)
{
     if (source == null)
          throw new Exception("null");

     var e = source.GetEnumerator();
     if (!e.MoveNext()) return default;
     var result = e.Current;
     return e.MoveNext() ? throw new Exception("More than one") : result;
}

public static TSource MySingleOrDefault<TSource>(this IEnumerable<TSource> source, Func<TSource, bool> predicate)
{
     if (source == null || predicate == null)
          throw new Exception("null");
          
     int count = 0;
     TSource result = default;
     foreach (var item in source)
     {
          if (predicate(item))
          {
               result = item;
               count++;
          }
     }
     
     switch (count)
     {
          case 0: return default;
          case 1: return result;
          default: throw new Exception("More than one");
     }
}
```

---
### 總結 - First、Single、Last、FirstOrDefault、LastOrDefault、SingleOrDefault
[圖片來源](http://www.technicaloverload.com/linq-single-vs-singleordefault-vs-first-vs-firstordefault/)
![](https://i.imgur.com/iVfrxk6.png)

---

### [ElementAt](https://docs.microsoft.com/zh-tw/dotnet/api/system.linq.enumerable.elementat?view=netframework-4.8)

取出序列中指定索引位置的項目. 若索引值不存在 , 則拋出例外. 
```C#
public static TSource ElementAt<TSource> (this IEnumerable<TSource> source, int index);
```

#### 使用時機
- 使用的集合**不是 List 以及 Array 型別** ( 例如 queue ). 但又需要使用索引值取出集合中的項目.
    - ElementAt 會在動作前先判斷是否為 IList 型別 , 再使用 collection [index] 的方式取出值 , 以優化效能.
    - 對於 List 以及 Array 型別的操作 , 建議直接使用 collection [index] 的方式取出值 , 可以減少進入 ElementAt 函數的時間 , 另外可讀性上也較佳(個人偏好 :heartbeat: ) 
#### ElementAt 的用處
取出索引位置為 1 的人
```C#
public static IEnumerable<(string name, int age)> GetPeople()
{
     yield return (name: "小王", age: 15);
     yield return (name: "老黃", age: 31);
     yield return (name: "阿高", age: 74);
}
```
```C#
static void Main(string[] args)
{
     var people = GetPeople();
     var query = people.ElementAt(1);
     Console.WriteLine(query);
     Console.ReadKey();
}
```
##### 輸出結果
(老黃 , 31)

#### 簡單實作自己的 ElementAt
```C#
public static TSource MyElementAt<TSource>(this IEnumerable<TSource> source, int index)
{
     if (source == null)
          throw new Exception("null");
     if (index < 0)
          throw new Exception("index is out of Range");

     if (source is IList<TSource> list) return list[index];
     var enumerator = source.GetEnumerator();
     int determine;
     for (determine = -1; determine < index && enumerator.MoveNext(); determine++) ;
     return determine == index ? enumerator.Current : throw new Exception("index is out of Range");
}
```
---

### [ElementAtOrDefault](https://docs.microsoft.com/zh-tw/dotnet/api/system.linq.enumerable.elementatordefault?view=netframework-4.8)
ElementAtOrDefault 使用上與 ElementAt 無異 , 差別只在於 index 不存在時 , 不會拋出例外 , 而是**回傳型別的預設值.**
```C#
public static TSource ElementAtOrDefault<TSource> (this IEnumerable<TSource> source, int index);
```

#### 使用時機

- 使用的集合不是 List 以及 Array 型別 ( 例如 queue ). 但又需要使用索引值取出集合中的項目. 另外不想使用 try catch 處理 index 不存在的狀況.
    
#### ElementAtOrDefault 的用處
取出索引位置為 1 的人
```C#
public static IEnumerable<(string name, int age)> GetPeople()
{
     yield return (name: "小王", age: 15);
     yield return (name: "老黃", age: 31);
     yield return (name: "阿高", age: 74);
}
```
```C#
static void Main(string[] args)
{
     var people = GetPeople();
     var query = people.ElementAtOrDefault(1);
     Console.WriteLine(query);
     Console.ReadKey();
}
```
##### 輸出結果
(老黃 , 31)
#### 簡單實作自己的 ElementAtOrDefault
```C#
public static TSource MyElementAtOrDefault<TSource>(this IEnumerable<TSource> source, int index)
{
     if (source == null)
          throw new Exception("null");
     if (index >= 0)
     {
          if (source is IList<TSource> list && index < list.Count) return list[index];
          var enumerator = source.GetEnumerator();
          int determine;
          for (determine = -1; determine < index && enumerator.MoveNext(); determine++) ;
          return determine == index ? enumerator.Current : default;
     }
     return default;
}
```

---

### 總結
1. 我們會依照目前的需求選擇使用 First、Single、Last 中的一個 , 這可以使我們的 Code 更具有可讀性.
    - 若是需要符合條件的第一個項目 , 使用 First
    - 若是需要符合條件的最後一個項目 , 使用 Last
    - 若是需要符合條件的僅有一個項目 , 使用 Single
    
2. 若是僅是需要找出集合中符合條件的項目 , 建議使用 First , 效能較佳. 原因是它找到符合條件的項目後 , 會立刻回傳. 而不會將集合中的元素都走訪完畢. 

3. 建議只有非 Array 以及 List 資料型別才使用 ElementAt .

4. 有實作 indexer 的集合類型 , 像是 Array 以及 List 資料型態. 想取出第一個、最後一個或是唯一的元素 , 不建議使用 First、Single、Last. 因為其可以直接透過索引值取值.

---

### Thank you! 

You can find me on

- [GitHub](https://github.com/s0920832252)
- [Facebook](https://www.facebook.com/fourtune.chen)

若有謬誤 , 煩請告知 , 新手發帖請多包涵

# :100: :muscle: :tada: :sheep: 
