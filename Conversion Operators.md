---
tags: LinQ , C# , Conversion Operators
---

# ToArray & ToList & ToLookup  & ToDictionary & ToHashSet & OfType & Cast & AsEnumerable

### 前言
有時候一個資料集合的集合類型並非我們當前需要的 , 此時可使用 LinQ 的轉型的方法. 例如目前資料集合為 List , 但某個方法的輸入參數為陣列 , 此時就需要使用 ToArray.
1. ToArray() - 集合類型轉成陣列
2. ToList() - 集合類型轉成 List
3. ToLookup() - 集合類型轉成 Lookup
4. ToDictionary - 集合類型轉成 Dictionary
5. ToHashSet - 集合類型轉型成 HashSet
6. OfType - 將集合內元素轉型某型別 , **不能轉型的元素會被過濾**
7. Cast - 將集合內元素強制轉型某型別 , **若有不能轉型的元素會跳例外**
8. AsEnumerable - 轉成 Enumerable
    - 用來轉成 Enumerable<T> , 然後就可以使用其他的 LinQ 方法
    - 從 DB 取資料時 , 使用 AsEnumerable 後 , 代表資料會從 DB 讀進 Memory

### [ToArray](https://docs.microsoft.com/zh-tw/dotnet/api/system.linq.enumerable.toarray?view=netframework-4.8)
```C#
public static TSource[] ToArray<TSource> (
                    this IEnumerable<TSource> source
);
```

#### ToArray 的使用
```C#
static void Main(string[] args)
{
    List<(string Company, double Weight)> packages = new List<(string, double)>
    {   ("Coho Vineyard", 25.2 ),
        ("Lucerne Publishing",  18.7 ),
        ("Wingtip Toys",  6.0 ),
        ("Adventure Works",  33.8 ),
    };

    string[] companies = packages.Select(package => package.Company).ToArray();
    foreach (string company in companies)
    {
        Console.WriteLine(company);
    }
    Console.ReadKey();
}
```
##### 輸出結果
Coho Vineyard    
Lucerne Publishing    
Wingtip Toys    
Adventure Works    
#### 簡單實作自己的 ToArray
```C#
public static TSource[] MyToArray<TSource>(this IEnumerable<TSource> source)
{
    if (source is null) throw new Exception("source");

    int size = 0;
    foreach (var i in source) size++;

    if (size == 0) return Array.Empty<TSource>();

    TSource[] items = new TSource[size];
    int index = 0;
    foreach (TSource item in source)
    {
        items[index++] = item;
    }
    return items;
}
```
#### ToArray 的 Source Code
```C#
public static TSource[] ToArray2<TSource>(this IEnumerable<TSource> source)
{
     if (source == null) throw Error.ArgumentNull("source");
     return new Buffer<TSource>(source).ToArray();
}

struct Buffer<TElement>
{
     internal TElement[] items;
     internal int count;

     internal Buffer(IEnumerable<TElement> source)
     {
          TElement[] items = null;
          int count = 0;
          if (source is ICollection<TElement> collection)
          {
               count = collection.Count;
               if (count > 0)
               {
                    items = new TElement[count];
                    collection.CopyTo(items, 0);
               }
          }
          else
          {
               foreach (TElement item in source)
               {
                    if (items == null)
                    {
                         items = new TElement[4];
                    }
                    else if (items.Length == count)
                    {
                         TElement[] newItems = new TElement[checked(count * 2)];
                         Array.Copy(items, 0, newItems, 0, count);
                         items = newItems;
                    }
                    items[count] = item;
                    count++;
                }
          }
          this.items = items;
          this.count = count;
    }

    internal TElement[] ToArray()
    {
        if (count == 0) return new TElement[0];
        if (items.Length == count) return items;
        TElement[] result = new TElement[count];
        Array.Copy(items, 0, result, 0, count);
        return result;
    }
}
```

### [ToList](https://docs.microsoft.com/zh-tw/dotnet/api/system.linq.enumerable.tolist?view=netframework-4.8)

```C#
public static List<TSource> ToList<TSource> (
            this IEnumerable<TSource> source
);
```
#### ToList 的使用

```C#
static void Main(string[] args)
{
    string[] fruits = { "apple", "passionfruit", "banana", "mango",
                        "orange", "blueberry", "grape", "strawberry" };

    List<int> lengths = fruits.Select(fruit => fruit.Length).ToList();
    foreach (int length in lengths)
    {
        Console.WriteLine(length);
    }
    Console.ReadKey();
}
```
##### 輸出結果
5    
12    
6    
5    
6    
9    
5    
10    
#### 簡單實作自己的 ToList
```C#
public static List<TSource> MyToList<TSource>(this IEnumerable<TSource> source)
{
    return source is null ? 
           throw new Exception("source") :
           new List<TSource>(source);
}
```



### [ToLookup](https://docs.microsoft.com/zh-tw/dotnet/api/system.linq.enumerable.tolookup?view=netframework-4.8)

詳細請參考 [LinQ基礎 - Lookup](https://hackmd.io/yMO0aHHPQKm61t0ceAeDCQ)

#### ToLookup 的多載
```C#
public static ILookup<TKey,TSource> ToLookup<TSource,TKey> (
        this IEnumerable<TSource> source, 
        Func<TSource,TKey> keySelector
);
```
```C#
public static ILookup<TKey,TSource> ToLookup<TSource,TKey> (
        this IEnumerable<TSource> source,
        Func<TSource,TKey> keySelector, 
        IEqualityComparer<TKey> comparer
);
```
```C#
public static ILookup<TKey,TElement> ToLookup<TSource,TKey,TElement> (
        this IEnumerable<TSource> source, 
        Func<TSource,TKey> keySelector,
        Func<TSource,TElement> elementSelector
);
```
```C#
public static ILookup<TKey,TElement> ToLookup<TSource,TKey,TElement> (
        this IEnumerable<TSource> source,
        Func<TSource,TKey> keySelector,
        Func<TSource,TElement> elementSelector,   
        IEqualityComparer<TKey> comparer
);
```
- keySelector : 將資料轉為 group 使用的 key
- elementSelector : 對資料執行 Select
#### ToLookup 的使用
```C#
static void Main(string[] args)
{
    List<(string Company, double Weight)> packages = new List<(string, double)>
    {   ("Coho Vineyard", 25.2 ),
        ("Lucerne Publishing",  18.7 ),
        ("Wingtip Toys",  6.0 ),
        ("Adventure Works",  33.8 ),
        ("Adv QQQ",  35.2 ),
    };

    // 創造個 lookup , group key 為每家公司其名字的第一個字 , 
    //                group 內 value 為 $"{p.Company}  {p.Weight}" , 
    //                可能有複數值
    ILookup<string, string> lookup = packages.ToLookup(p => p.Company.Substring(0, 1),
                                                       p => $"{p.Company}  {p.Weight}"
                                                      );

    foreach (IGrouping<string, string> packageGroup in lookup)
    {
        Console.WriteLine(packageGroup.Key);
        foreach (string str in packageGroup)
            Console.WriteLine($"    {str}");
    }
    Console.ReadKey();
}
```
##### 輸出結果
![](https://i.imgur.com/bxC8guE.png)

#### 簡單實作自己的 ToLookup
```C#
public static ILookup<TKey, TSource> ToLookup<TSource, TKey>(this IEnumerable<TSource> source, Func<TSource, TKey> keySelector, IEqualityComparer<TKey> comparer = null)
{
    if (source is null || keySelector is null)
    {
        throw new Exception("null");
    }
    return Lookup<TKey, TSource>.Create(source, keySelector, comparer);
}

public static ILookup<TKey, TElement> ToLookup<TSource, TKey, TElement>(this IEnumerable<TSource> source, Func<TSource, TKey> keySelector, Func<TSource, TElement> elementSelector, IEqualityComparer<TKey> comparer = null)
{
    if (source is null || keySelector is null || elementSelector is null)
    {
        throw new Exception("null");
    }
    return Lookup<TKey, TElement>.Create(source, keySelector, elementSelector, comparer);
}
```

### [ToDictionary](https://docs.microsoft.com/zh-tw/dotnet/api/system.linq.enumerable.todictionary?view=netframework-4.8)
#### ToDictionary 的多載
```C#
1. public static Dictionary<TKey,TSource> ToDictionary<TSource,TKey> (
        this IEnumerable<TSource> source, 
        Func<TSource,TKey> keySelector
    );
2. public static Dictionary<TKey,TSource> ToDictionary<TSource,TKey> (
        this IEnumerable<TSource> source, 
        Func<TSource,TKey> keySelector, 
        IEqualityComparer<TKey> comparer
    );
3. public static Dictionary<TKey,TElement> ToDictionary<TSource,TKey,TElement> (
        this IEnumerable<TSource> source,
        Func<TSource,TKey> keySelector,
        Func<TSource,TElement> elementSelector
    );
4. public static Dictionary<TKey,TElement> ToDictionary<TSource,TKey,TElement> (
        this IEnumerable<TSource> source,
        Func<TSource,TKey> keySelector, 
        Func<TSource,TElement> elementSelector, 
        IEqualityComparer<TKey> comparer
    );
```
- keySelector - 指定每個元素各自的 key.
- elementSelector - 轉換每個元素成另外的形式 (就是 Select)
- comparer - 自定義比較器

#### ToDictionary 的使用
```C#
static void Main(string[] args)
{
    List<(string Company, double Weight)> packages = new List<(string, double)>
    {   ("Coho Vineyard", 25.2 ),
        ("Lucene Publishing",  18.7 ),
        ("Wingtip Toys",  6.0 ),
        ("Adventure Works",  33.8 ),
    };

    // 指定 Company 作為 Key , 元素自身則成為 Value
    // 建造一個新的 Dictionary
    var valueTuples = packages.ToDictionary(tuple => tuple.Company);
    foreach (var keyValuePair in valueTuples)
    {
        Console.WriteLine($"*Group Key is {keyValuePair.Key}");
        var (company, weight) = keyValuePair.Value;
        Console.WriteLine($"     company Name is {company}");
        Console.WriteLine($"     weight is {weight}");
        Console.WriteLine("------------------------------");
    }
    Console.ReadKey();
}  
```
##### 輸出結果
![](https://i.imgur.com/OcOMZXI.png)

```C#
static void Main(string[] args)
{
    List<(string Company, double Weight)> packages = new List<(string, double)>
    {   ("Coho Vineyard", 25.2 ),
        ("Lucene Publishing",  18.7 ),
        ("Wingtip Toys",  6.0 ),
        ("Adventure Works",  33.8 ),
    };

    // 指定 Company 作為 Key , Value 為元素丟入 elementSelector 後的結果
    // 建造一個新的 Dictionary
    var valueTuples = packages.ToDictionary(tuple => tuple.Company, tuple => tuple.Weight);
    foreach (var keyValuePair in valueTuples)
    {
        Console.WriteLine($"Key & Company is {keyValuePair.Key}");
        Console.WriteLine($"Weight is {keyValuePair.Value}");
        Console.WriteLine("------------------------------");
    }
    Console.ReadKey();
}
```
##### 輸出結果
![](https://i.imgur.com/5R4lsBj.png)



#### 簡單實作自己的 ToDictionary
```C#
public static Dictionary<TKey, TSource> MyToDictionary<TSource, TKey>(this IEnumerable<TSource> source, Func<TSource, TKey> keySelector)
{
    return MyToDictionary<TSource, TKey, TSource>(source, keySelector, item => item, null);
}

public static Dictionary<TKey, TSource> MyToDictionary<TSource, TKey>(this IEnumerable<TSource> source, Func<TSource, TKey> keySelector, IEqualityComparer<TKey> comparer)
{
    return MyToDictionary<TSource, TKey, TSource>(source, keySelector, item => item, comparer);
}

public static Dictionary<TKey, TElement> MyToDictionary<TSource, TKey, TElement>(this IEnumerable<TSource> source, Func<TSource, TKey> keySelector, Func<TSource, TElement> elementSelector)
{
    return MyToDictionary<TSource, TKey, TElement>(source, keySelector, elementSelector, null);
}

public static Dictionary<TKey, TElement> MyToDictionary<TSource, TKey, TElement>(this IEnumerable<TSource> source, Func<TSource, TKey> keySelector, Func<TSource, TElement> elementSelector, IEqualityComparer<TKey> comparer)
{
    if (source is null || keySelector is null || elementSelector is null)
    {
        throw new Exception("null exception");
    }

    var d = new Dictionary<TKey, TElement>(comparer ?? EqualityComparer<TKey>.Default);
    foreach (var element in source)
    {
        d.Add(keySelector(element), elementSelector(element));
    }
    return d;
}
```

### ToHashSet
```C#
public static HashSet<TSource> ToHashSet<TSource> (
         this IEnumerable<TSource> source
    );
```

```C#
public static HashSet<TSource> ToHashSet<TSource> (
         this IEnumerable<TSource> source,  
         IEqualityComparer<TSource> comparer
    );
```

#### ToHashSet  的使用
```C#
static void Main(string[] args)
{
    List<(string Company, double Weight)> packages = new List<(string, double)>
    {   ("Coho Vineyard", 25.2 ),
        ("Lucene Publishing",  18.7 ),
        ("Lucene Publishing",  18.7 ),
        ("Lucene Publishing",  18.7 ),
        ("Wingtip Toys",  6.0 ),
        ("Wingtip Toys",  6.0 ),
        ("Adventure Works",  33.8 ),
        ("Adventure Works",  33.8 ),
        ("Wingtip Toys",  6.0 ),
        ("Ed-venture Works",  33.8 ),
        ("Adventure Works",  33.9 ),
    };

    var hashSet = packages.ToHashSet();
    foreach (var (company, weight) in hashSet)
    {
        Console.WriteLine($"{company} -- {weight}");
        Console.WriteLine("------------------------------");
    }
    Console.ReadKey();
}
```
##### 輸出結果
![](https://i.imgur.com/Ns0h7EW.png)

#### 簡單實作自己的 ToHashSet 
```C#
public static HashSet<TSource> MyToHashSet<TSource>(this IEnumerable<TSource> source)
{
    return source.ToHashSet(null);
}

public static HashSet<TSource> MyToHashSet<TSource>(this IEnumerable<TSource> source, IEqualityComparer<TSource> comparer)
{
    if (source == null)
    {
        throw new Exception("source is null");
    }
    return new HashSet<TSource>(source, comparer ?? EqualityComparer<TSource>.Default);
}
```

### OfType
```C#
public static IEnumerable<TResult> OfType<TResult> (
             this IEnumerable source
    );
```

#### OfType 的使用
```C#
abstract class Animal
{
    public string Name { get; set; }
    public int Age { get; set; }
}

class Dog : Animal
{
}

class Cat : Animal
{
}

static void Main(string[] args)
{
    var animals = new List<Animal>
    {
        null,
        new Dog(){ Name = "小汪", Age = 15 },
        new Dog(){ Name = "小黑", Age = 7 },
        new Cat(){ Name = "肥貓", Age = 3 },
        new Cat(){ Name = "喵貓", Age = 8 },
        new Dog(){ Name = "小笨狗", Age = 18 },
    };

    var dogs = animals.OfType<Dog>();
    foreach (var dog in dogs)
    {
        Console.WriteLine($"dog name is {dog.Name} , dog age is {dog.Age}");
    }

    Console.ReadKey();
}
```

#### 簡單實作自己的 OfType
```C#
public static IEnumerable<TResult> MyOfType<TResult>(this IEnumerable source)
{
    if (source is null)
    {
        throw new Exception("source is null");
    }
    return MyOfTypeIterator<TResult>(source);
}

private static IEnumerable<TResult> MyOfTypeIterator<TResult>(IEnumerable source)
{
    foreach (object obj in source)
    {
        // null 不能轉成 TResult , 所以為 false
        if (obj is TResult item)
        {
            yield return item;
        }
    }
}
```

### Cast
```C#
public static IEnumerable<TResult> Cast<TResult> (
             this IEnumerable source
    );
```

#### Cast 的使用
```C#
private abstract class Animal
{
    public string Name { get; set; }
    public int Age { get; set; }
}

private class Dog : Animal
{
}

private class Cat : Animal
{
}

static void Main(string[] args)
{
    var animals = new List<Animal>
    {
        null, // 使用 Cast 要小心成員為 null 的狀態 , 他無法判斷
        new Dog() {Name = "小汪", Age = 15},
        new Dog() {Name = "小黑", Age = 7},
        new Cat() {Name = "肥貓", Age = 3},
        new Cat() {Name = "喵貓", Age = 8},
        new Dog() {Name = "小笨狗", Age = 18},
    };

    try
    {
        var collection = animals.Cast<Dog>();
        foreach (var item in collection)
        {
            Console.WriteLine($"is null {item is null} ,  dog name is {item?.Name} , dog age is {item?.Age}");
        }
    }
    catch (Exception e)
    {
        Console.WriteLine($"Exception {e.Message}");
    }

    Console.ReadKey();
}
```
##### 輸出結果
is null True ,  dog name is  , dog age is     
is null False ,  dog name is 小汪 , dog age is 15     
is null False ,  dog name is 小黑 , dog age is 7     
Exception 無法將類型 'Cat' 的物件轉換為類型 'Dog'。     
#### 簡單實作自己的 Cast
```C#
public static IEnumerable<TResult> MyCast<TResult>(this IEnumerable source)
{
    switch (source)
    {
        case null:
            throw new Exception("source is null");
        case IEnumerable<TResult> typeSource:
            return typeSource;
        default:
            return MyCastIterator<TResult>(source);
    }
}

private static IEnumerable<TResult> MyCastIterator<TResult>(IEnumerable source)
{
    foreach (TResult obj in source)
    {
        yield return obj;
    }
}
```

### [AsEnumerable](https://docs.microsoft.com/zh-tw/dotnet/api/system.linq.enumerable.asenumerable?view=netframework-4.8)

不常用 , 只有物件轉型時才會用 = ="
不太熟 , 留給以後的自己了 Orz.

#### AsEnumerable 的使用
```C#
// Custom class.
class Clump<T> : List<T>
{
    // Custom implementation of Where().
    public IEnumerable<T> Where(Func<T, bool> predicate)
    {
        Console.WriteLine("In Clump's implementation of Where().");
        return Enumerable.Where(this, predicate);
    }
}

static void AsEnumerableEx1()
{
    // Create a new Clump<T> object.
    Clump<string> fruitClump =
        new Clump<string> { "apple", "passionfruit", "banana", 
            "mango", "orange", "blueberry", "grape", "strawberry" };

    // First call to Where():
    // Call Clump's Where() method with a predicate.
    IEnumerable<string> query1 =
        fruitClump.Where(fruit => fruit.Contains("o"));

    Console.WriteLine("query1 has been created.\n");

    // Second call to Where():
    // First call AsEnumerable() to hide Clump's Where() method and thereby
    // force System.Linq.Enumerable's Where() method to be called.
    IEnumerable<string> query2 =
        fruitClump.AsEnumerable().Where(fruit => fruit.Contains("o"));

    // Display the output.
    Console.WriteLine("query2 has been created.");
}

// This code produces the following output:
//
// In Clump's implementation of Where().
// query1 has been created.
//
// query2 has been created.
```
#### 簡單實作自己的 AsEnumerable
```C#
public static IEnumerable<TSource> AsEnumerable<TSource>(
    this IEnumerable<TSource> source) => source;
```

### 參考
[LINQ使用细节之.AsEnumerable()和.ToList()的区别](https://www.cnblogs.com/mainz/archive/2011/04/08/2009485.html)
[What's the difference(s) between .ToList(), .AsEnumerable(), AsQueryable()?](https://stackoverflow.com/questions/17968469/whats-the-differences-between-tolist-asenumerable-asqueryable)     
[REIMPLEMENTING LINQ TO OBJECTS: PART 36 – ASENUMERABLE](https://codeblog.jonskeet.uk/2011/01/14/reimplementing-linq-to-objects-part-36-asenumerable/)     
[Conversion Operators in LINQ](https://www.c-sharpcorner.com/UploadFile/219d4d/conversion-operators-in-linq/)  
[[LINQ] List泛型 轉 ToDictionary泛型](https://dotblogs.com.tw/yc421206/2011/08/14/33106)  
[[译文]c# /.Net 技巧: ToDictionary() and ToList()](https://www.cnblogs.com/haorui/p/4516208.html)  


### Thank you! 

You can find me on

- [GitHub](https://github.com/s0920832252)
- [Facebook](https://www.facebook.com/fourtune.chen)

若有謬誤 , 煩請告知 , 新手發帖請多包涵

# :100: :muscle: :tada: :sheep: 
