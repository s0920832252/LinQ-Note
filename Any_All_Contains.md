---
tags: LinQ , C# , Quantifiers Operators
---

# Any & All & Contains
Any , All and Contains 均是非常容易會使用到的方法 , 另外需要注意的是這三個方法都不具有延遲執行的特性.

### [Any](https://docs.microsoft.com/zh-tw/dotnet/api/system.linq.enumerable.any?view=netframework-4.8)
判斷序列的任何項目是否符合條件或包含任何項目    
    
Any 有兩個多載方法如下
1. ```C#
    public static bool Any<TSource> (this IEnumerable<TSource> source);
    ```
    - 判斷序列中包含的項目數**是否**不為零

2. ```C#
    public static bool Any<TSource> (this IEnumerable<TSource> source, Func<TSource,bool> predicate);
    ```
    - 判斷序列中的任何項目**是否**有任何一個符合條件 
    - 實作上與 First 類似 , 差別在於回傳值. First 會回傳第一個符合條件的項目 , Any 則是發現第一個符合條件的項目時會回傳 true , 反之若沒有任何項目符合條件會回傳 false
---
#### 使用時機
- 需要判斷集合是否包含任何項目.
- 需要使用 **predicate** 表達指定的項目 , 以判斷集合中是否存在某個指定項目
#### Any 的用法
```C#
// 列出每個人持有的電話 , 若是他沒有電話則不要列出.
static void Main(string[] args)
{
    var people = new[]
    {
        new  { Name="大名" , CellPhone = new string[]{"0989112266","03-29007536"}},
        new  { Name="小黃" , CellPhone = new string[]{"0961334475"}},
        new  { Name="大龍" , CellPhone = new string[]{}},
        new  { Name="包子" , CellPhone = new string[]{"0953793211", "0961751112", "0975362475"}}
    };

    // 使用 Where 篩選出有電話的人 , 透過 Any 判斷這個人是否有電話
    var query = people.Where(person => person.CellPhone.Any())
                      .SelectMany((person) => person.CellPhone, (person, phone) => $"{person.Name} 持有的手機號碼是 {phone}");

    // 阿龍沒有手機 , 所以不會 print 阿龍出來
    foreach (var item in query)
    {
        Console.WriteLine(item);
    }

    Console.ReadKey();
}
```
##### 輸出結果
大名 持有的手機號碼是 0989112266     
大名 持有的手機號碼是 03-29007536     
小黃 持有的手機號碼是 0961334475     
包子 持有的手機號碼是 0953793211     
包子 持有的手機號碼是 0961751112     
包子 持有的手機號碼是 0975362475     

```C#
// 找出持有 03- 的電話的人所持有的手機號碼
static void Main(string[] args)
{
    var people = new[]
    {
        new  { Name="大名" , CellPhone = new string[]{"0989112266","03-29007536"}},
        new  { Name="小黃" , CellPhone = new string[]{"0961334475"}},
        new  { Name="大龍" , CellPhone = new string[]{}},
        new  { Name="包子" , CellPhone = new string[]{"0953793211", "0961751112", "0975362475"}}
    };
    // 透過 Any 判斷這個人是否持有 03- 開頭的手機
    var query = people.Where(person => person.CellPhone.Any(num => num.Contains("03-")))
                      .SelectMany((person) => person.CellPhone, (person, phone) => $"{person.Name} 持有的手機號碼是 {phone}");

    foreach (var item in query)
    {
        Console.WriteLine(item);
    }

    Console.ReadKey();
}
```
##### 輸出結果
大名 持有的手機號碼是 0989112266     
大名 持有的手機號碼是 03-29007536     

---

#### 簡單實作自己的 Any
```C#
public static bool MyAny<TSource>(this IEnumerable<TSource> source)
{
    if (source is null)
    {
        throw new Exception("source is null");
    }

    return source.GetEnumerator().MoveNext();
}
```
```C#
public static bool MyAny<TSource>(this IEnumerable<TSource> source, Func<TSource, bool> predicate)
{
    if (source is null || predicate is null)
    {
        throw new Exception("either source or predicate is null");
    }

    foreach (var item in source)
    {
        if (predicate(item))
        {
            return true;
        }
    }
    return false;
}
```
### All
判斷序列的所有項目**是否全**都符合條件       
    
```C#
public static bool All<TSource> (this IEnumerable<TSource> source, Func<TSource,bool> predicate);
```

#### 使用時機
- 需要使用 predicate 表達指定的項目 , 以判斷集合中項目是否全都滿足這個指定項目    
#### All 的用法
```C#
class Pet
{
    public string Name { get; set; }
    public int Age { get; set; }
}

static void Main(string[] args)
{
    // Create an array of Pets.
    Pet[] pets = {  new Pet { Name="Barley", Age=10 },
                    new Pet { Name="Boots", Age=4 },
                    new Pet { Name="Whiskers", Age=6 }
                 };

    // Determine whether all pet names 
    // in the array start with 'B'.
    bool allStartWithB = pets.All(pet => pet.Name.StartsWith("B"));

    Console.WriteLine("{0} pet names start with 'B'.", allStartWithB ? "All" : "Not all");

    Console.ReadKey();
}
```
##### 輸出結果
Not all pet names start with 'B'.

#### 簡單實作自己的 All
```C#
public static bool MyAll<TSource>(this IEnumerable<TSource> source, Func<TSource, bool> predicate)
{
    if (source is null || predicate is null)
    {
        throw new Exception("source or predicate is null");
    }

    foreach (var element in source)
    {
        if (!predicate(element))
        {
            return false;
        }
    }
    return true;
}
```

### Contains
判斷序列是否包含指定的項目    
    
Contains 有兩個多載方法
1. public static bool Contains<TSource> (this IEnumerable<TSource> source, TSource value);	
    - 使用預設的相等比較子 (Comparer) 來判斷序列是否包含指定的項目。
2.  public static bool Contains<TSource> (this IEnumerable<TSource> source, TSource value, IEqualityComparer<TSource> comparer);	
    - 使用指定的 IEqualityComparer<T> 來判斷序列是否包含指定的項目。

#### 使用時機
- 想要找到指定的某個項目是否已存在於集合中.

#### Contains 的用法
```C#
// 判斷 fruits 中是否存在 mango
static void Main(string[] args)
{
     string[] fruits = { "apple", "banana", "mango", "orange", "passionfruit", "grape" };

     string fruit = "mango";

     bool hasMango = fruits.Contains(fruit);

     Console.WriteLine("The array {0} contain '{1}'.", hasMango ? "does" : "does not", fruit);

     Console.ReadKey();
}
```

##### 輸出結果
The array does contain 'mango'.    
    
    
```
class EqualityComparer<TSource> : IEqualityComparer<TSource> where TSource : class
{
     private PropertyInfo[] properties = null;
     public bool Equals(TSource x, TSource y)
     {
          if (properties == null)
               properties = x.GetType().GetProperties();
          return properties.Aggregate(true, (result, item) =>
          {
               return result && item.GetValue(x).Equals(item.GetValue(y));
          });
     }

     public int GetHashCode(TSource obj)
     {
          if (properties == null)
               properties = obj.GetType().GetProperties();
          return properties.Aggregate(0, (result, item) =>
          {
               return result ^ item.GetValue(obj).GetHashCode();
          });
     }
}

public class Product
{
     public string Name { get; set; }
     public int Code { get; set; }
}

static void Main(string[] args)
{
     Product[] fruits = { 
         new Product { Name = "apple", Code = 9 },
         new Product { Name = "orange", Code = 4 },
         new Product { Name = "lemon", Code = 12 } 
     };

     Product apple = new Product { Name = "apple", Code = 9 };
     Product kiwi = new Product { Name = "kiwi", Code = 8 };

     EqualityComparer<Product> prodc = new EqualityComparer<Product>();

     bool hasApple = fruits.Contains(apple, prodc);
     bool hasKiwi = fruits.Contains(kiwi, prodc);

     Console.WriteLine("Apple? " + hasApple);
     Console.WriteLine("Kiwi? " + hasKiwi);

     Console.ReadKey();
}
```
##### 輸出結果
Apple? True       
Kiwi? False      

#### 簡單實作自己的 Contains
```C#
public static bool MyContains<TSource>(this IEnumerable<TSource> source, TSource value)
{
     if (source is null)
     {
          throw new Exception("source is null");
     }

     foreach (var element in source)
     {
          if (element.Equals(value))
          {
               return true;
          }
     }
     return false;
}

public static bool MyContains<TSource>(this IEnumerable<TSource> source, TSource value, IEqualityComparer<TSource> comparer)
{
     if (source is null || comparer is null)
     {
          throw new Exception("source or comparer is null");
     }

     foreach (var element in source)
     {
          if (comparer.Equals(element, value))
          {
               return true;
          }
     }
     return false;
}
```

### Summary 
- Any 和 Contains 均可以用來判斷集合中是否包含指定的項目 , 其差別在於 Any 是透過 predicate 表達指定的項目 , 而 Contains 則是直接丟入指定的項目.







---

### Thank you! 

You can find me on

- [GitHub](https://github.com/s0920832252)
- [Facebook](https://www.facebook.com/fourtune.chen)

若有謬誤 , 煩請告知 , 新手發帖請多包涵

# :100: :muscle: :tada: :sheep: 
