---
tags: LinQ , C# , Cellaneous Operators
---

#  Concat & EqualAll & SequenceEqual & Zip 
### 閒話543
不知道怎麼分類的運算丟這裡:alien: 

---

### [Concat](https://docs.microsoft.com/zh-tw/dotnet/api/system.linq.enumerable.concat?view=netframework-4.8)
在保持原來自己集合元素的先後顺序的情况下把兩個集合連接起来 , 但不會過濾相同的項目.
因為是回傳值是 IEnumerable 型態 , 所以其具有延後執行的特性
```C#
public static IEnumerable<TSource> Concat<TSource> (this IEnumerable<TSource> first, IEnumerable<TSource> second);
```
#### 使用時機
- 需要把兩個序列串接成一個. 
#### Concat 的用法
列出寵物 狗 & 貓 的資訊
```C#
static void Main(string[] args)
{
     var dogs = new[] { (Name : "小黃", Age : 1 ),
                        ( Name : "小狗", Age : 2 ),
                      };

     var cats = new[] {  (Name: "小喵", Age: 7) ,
                         (Name: "小貓", Age: 4) ,
                      };
                      
     var combin = dogs.Concat(cats);
     cats[0].Age = 15; // 其具有延後執行的特性 , 所以小喵的 Age 是 15 而非 7
                              // 列出寵物 狗 & 貓 的資訊
     foreach (var (Name, Age) in combin)
     {
          Console.WriteLine($"{Name} {Age}");
     }
     Console.WriteLine();
     // 也可以使用 Select & SelectMany 做出一樣的結果
     foreach (var (Name, Age) in new[] { dogs.Select(d => d), cats.Select(c => c) }.SelectMany(m => m))
     {
          Console.WriteLine($"{Name} {Age}");
     }

    Console.ReadKey();
}
```
##### 輸出
小黃 1      
小狗 2      
小喵 15      
小貓 4      
              
小黃 1      
小狗 2      
小喵 15      
小貓 4      
#### 簡單實作自己的 Concat
```C#
public static IEnumerable<TSource> MyConcat<TSource>(this IEnumerable<TSource> first, IEnumerable<TSource> second)
{
    if (first is null || second is null)
    {
        throw new Exception("null");
    }
    return MyConcatIterator(first, second);
}

private static IEnumerable<TSource> MyConcatIterator<TSource>(IEnumerable<TSource> first, IEnumerable<TSource> second)
{
    foreach (var item in first)
    {
        yield return item;
    }
    foreach (var item in second)
    {
        yield return item;
    }
}
```
---
### [SequenceEqual](https://docs.microsoft.com/zh-tw/dotnet/api/system.linq.enumerable.sequenceequal?view=netframework-4.8)
SequenceEqual 可以比對兩個集合的內容是否完全相同 , 
比對順序是循序式的. 如以下 , 先比較第一個是否皆為蘋果 , 在比較第二個是否皆為香蕉 ... 
集合名稱        | 第一個 | 第二個 | 第三個 | ...
--------------|:-----:|-----:| ----:|----------------------
小明的水果    | 蘋果 |  香蕉 |    鳳梨 | ...
小王的水果    | 蘋果 |  香蕉 |    鳳梨 |...


因此
- **兩集合的順序不一致會被視為不同** , 
- **兩集合的數量不同也會被視為不同**
- 內容值為參考型別時 , 若該型別沒有實作 IEquatable , 則預設比較是比較物件的參考位址是否相同
    - 若不希望如此 , 可以使用自訂的 IEqualityComparer 來比較.
- 內容值是簡單型別時 , 則比較實際資料是否相同

以下是它的多載形式
1. public static bool SequenceEqual<TSource> (this IEnumerable<TSource> first, IEnumerable<TSource> second);
    - 使用項目之型別的預設相等比較子來比較項目，以判斷兩個序列是否相等。
2. public static bool SequenceEqual<TSource> (this IEnumerable<TSource> first, IEnumerable<TSource> second, IEqualityComparer<TSource> comparer);    
    - 使用指定的 IEqualityComparer<T> 來比較項目，以判斷兩個序列是否相等。

#### 使用時機
- 需要判斷兩個集合是否完全相同
    - 第一個多載用在簡單型別 , 第二個多載用在參考型別

#### SequenceEqual 的用法
```C#
class Pet
{
     public string Name { get; set; }
     public int Age { get; set; }
}

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

class Product : IEquatable<Product>
{
     public string Name { get; set; }
     public int Code { get; set; }

     public bool Equals(Product other)
     {
          return Name == other.Name && Code == other.Code;
     }
}

static void Main(string[] args)
{
     Pet pet1 = new Pet { Name = "Turbo", Age = 2 };
     Pet pet2 = new Pet { Name = "Peanut", Age = 8 };

     List<Pet> pets1 = new List<Pet> { pet1, pet2 };
     List<Pet> pets2 = new List<Pet> { pet1, pet2 };
     List<Pet> pets3 = new List<Pet> { pet2, pet1 };
     // 比較參考位址
     Console.Write($"The {nameof(pets1)} & {nameof(pets2)} ");
     Console.WriteLine($"{(pets1.SequenceEqual(pets2) ? "are" : "are not")} equal.");
     Console.Write($"The {nameof(pets1)} & {nameof(pets3)} ");
     Console.WriteLine($"{(pets1.SequenceEqual(pets3) ? "are" : "are not")} equal.");

     Console.WriteLine("********");

     Product[] storeA = { new Product { Name = "apple", Code = 9 },
                          new Product { Name = "orange", Code = 4 }
                        };
     Product[] storeB = { new Product { Name = "apple", Code = 9 },
                          new Product { Name = "orange", Code = 4 }
                        };
     Product[] storeC = { new Product { Name = "bannaaaa", Code = 9 },
                          new Product { Name = "orange", Code = 4 }
                        };
     // 比較 Name & Code 是否都相同
     Console.Write($"The {nameof(storeA)} & {nameof(storeB)} ");
     Console.WriteLine($"{(storeA.SequenceEqual(storeB) ? "are" : "are not")} equal.");
     Console.Write($"The {nameof(storeA)} & {nameof(storeC)} ");
     Console.WriteLine($"{(storeA.SequenceEqual(storeC) ? "are" : "are not")} equal.");

     Console.WriteLine("********");
     // 比較 Name & Age 是否都相同
     Console.Write($"The {nameof(pets1)} & {nameof(pets2)} ");
     Console.WriteLine($"{(pets1.SequenceEqual(pets2,new EqualityComparer<Pet>()) ? "are" : "are not")} equal.");
     Console.Write($"The {nameof(pets1)} & {nameof(pets3)} ");
     Console.WriteLine($"{(pets1.SequenceEqual(pets3, new EqualityComparer<Pet>()) ? "are" : "are not")} equal.");

     Console.ReadKey();
}
```
##### 輸出結果
The pets1 & pets2 are equal.          
The pets1 & pets3 are not equal.            
###############       
The storeA & storeB are equal.               
The storeA & storeC are not equal.           
###############           
The pets1 & pets2 are equal.           
The pets1 & pets3 are not equal.           

#### 簡單實作自己的 SequenceEqual
```C#
public static bool MySequenceEqual<TSource>(this IEnumerable<TSource> first, IEnumerable<TSource> second)
{
     return MySequenceEqual(first, second, null);
}

public static bool MySequenceEqual<TSource>(this IEnumerable<TSource> first, IEnumerable<TSource> second, IEqualityComparer<TSource> comparer)
{
     if (comparer == null)
     {
          comparer = EqualityComparer<TSource>.Default;
     }
     if (first == null || second == null)
     {
          throw new Exception(null);
     }
     
     using (IEnumerator<TSource> e1 = first.GetEnumerator())
     using (IEnumerator<TSource> e2 = second.GetEnumerator())
     {
          while (e1.MoveNext())
          {
               if (!(e2.MoveNext() && comparer.Equals(e1.Current, e2.Current)))
               {
                    return false;
               }
          }
          if (e2.MoveNext())
          {
               return false;
          }
     }
     return true;
}
```
---
### [Zip](https://docs.microsoft.com/zh-tw/dotnet/api/system.linq.enumerable.zip?view=netframework-4.8)
Zip 在 C# 4.0 才被加入 . 其會循序的對兩集合的元素做使用者指定的操作 , 例如若是**使用者指定的操作為字串合併** , 則在合併小明以及小王的水果時 , 會得到一個 IEnumerable 型態的序列 , 所以其也具有延遲執行的特性. **另外若是兩集合的長度不同 , Zip 會循序對兩集合操作直到某一個集合已被走訪完畢為止.**
##### 小王和小明的水果一樣多
集合名稱        | 第一個 | 第二個 | 第三個 | ...
--------------|:-----:|:-----:|:----:|----------------------
小明的水果    | 蘋果1 | 香蕉1 | 鳳梨1 | ...
小王的水果    | 蘋果2 | 香蕉2 | 鳳梨2 |...
Zip 的結果    | 蘋果1蘋果2 | 香蕉1香蕉2 | 鳳梨1鳳梨2 |...

##### 小王的水果比較多
集合名稱        | 第一個 | 第二個 | 第三個 | ...
--------------|:-----:|:-----:|:----:|----------------------
小明的水果    | 蘋果1 | 無 | 無 | 無
小王的水果    | 蘋果2 | 香蕉2 | 鳳梨2 |...
Zip 的結果    | 蘋果1蘋果2 | 無 | 無 | 無

#### 多載
1. public static IEnumerable<TResult> Zip<TFirst,TSecond,TResult> (this IEnumerable<TFirst> first, **IEnumerable<TSecond> second**, **Func<TFirst,TSecond,TResult> resultSelector**);
    - 將指定的函式套用至兩個序列的對應項目，產生結果的序列。

#### 使用時機
- 想要對某兩個集合循序操作 , 但卻無法知道兩個集合數量的時候.

#### Zip 的用法
```C#
static void Main(string[] args)
{
     List<(string name, int age)> People = new List<(string name, int cost)>
     {
          ("小王" , 99),
          ("小黃" , 50),
     };
     List<(string id, int cost)> Invoices = new List<(string name, int cost)>
     {
          ("鳳梨" , 100),
          ("電動" , 500),
          ("冰箱" , 95),
     };

     // 冰箱會被忽略. 以較短的那個集合為主.
     var zip = People.Zip(Invoices, (person, invoice) 
         => $"{person.age}歲的{person.name} 買了{invoice.id} , 花了 {invoice.cost}");

     foreach (var item in zip)
     {
          Console.WriteLine(item);
     }

     Console.ReadKey();
}
```
##### 輸出結果
99歲的小王 買了鳳梨 , 花了 100
50歲的小黃 買了電動 , 花了 500
#### 簡單實作自己的 Zip
```C#
public static IEnumerable<TResult> MyZip<TFirst, TSecond, TResult>(this IEnumerable<TFirst> first, IEnumerable<TSecond> second, Func<TFirst, TSecond, TResult> resultSelector)
{
     if (first is null || second is null || resultSelector is null)
     {
          throw new Exception("source is null");
     }
     return MyZipIterator(first, second, resultSelector);
}

private static IEnumerable<TResult> MyZipIterator<TFirst, TSecond, TResult>(IEnumerable<TFirst> first, IEnumerable<TSecond> second, Func<TFirst, TSecond, TResult> resultSelector)
{
     var firstEnumerator = first.GetEnumerator();
     var secondEnumerator = second.GetEnumerator();
     while (firstEnumerator.MoveNext() && secondEnumerator.MoveNext())
     {
          yield return resultSelector(firstEnumerator.Current, secondEnumerator.Current);
     }
}
```
---
### 參考資料

[EqualityComparer<T> 類別](https://docs.microsoft.com/zh-tw/dotnet/api/system.collections.generic.equalitycomparer-1?view=netframework-4.8)
SequenceEqual source code - [donetCorefx](https://github.com/dotnet/corefx/blob/master/src/System.Linq/src/System/Linq/SequenceEqual.cs) , [microsoft](https://github.com/microsoft/referencesource/blob/master/System.Core/System/Linq/Enumerable.cs)
    
    
---








### Thank you! 

You can find me on

- [GitHub](https://github.com/s0920832252)
- [Facebook](https://www.facebook.com/fourtune.chen)

若有謬誤 , 煩請告知 , 新手發帖請多包涵

# :100: :muscle: :tada: :sheep: 
