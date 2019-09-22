---
tags: LinQ , C# , Aggregate Operators
---

# Count、Sum、Average、Min、Max
Count、Sum、Average、Min、Max 是 LinQ 內用來進行統計運算(?)的函數. 其與 First 相同 , 都是立即執行（Immediately execution）查詢. 因此不用擔心延遲執行的問題. 另外需要特別注意的是上述函數的回傳值只可能是 **Value Type 以及 Nullable Type .** 也就是說 , 像是回傳學生集合中成績最小的學生物件 , 這個動作是無法達成的.

### [Min](https://docs.microsoft.com/zh-tw/dotnet/api/system.linq.enumerable.min?view=netframework-4.8)
回傳序列中指定項目的最小值. 

常用的多載形式如下(不只)
1. TSource Min<TSource>(this IEnumerable<TSource>)
    - Returns the minimum value in a generic sequence.
2. TResult Min<TSource,TResult>(this IEnumerable<TSource>, Func<TSource,TResult>)	
    - Invokes a transform function on each element of a generic sequence and returns the minimum resulting value.    
#### 使用時機
1. 當集合元素類型為 Value Type , 使用建構式 1 取出集合中的最小值
2. 當集合元素類型為 Reference Type , 使用建構式 2 取出集合中**指定屬性的最小值**.

#### Min 的用處
找出 list 中最小的數字 , 以及 people 中最小的年紀. 
```C#
public static List<int> GetList() => new List<int> { 5, 6, 9, 1, 3 };

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
     var list = GetList();
     var minNum = list.Min();
     Console.WriteLine(minNum);
     
     var people = GetPeople();
     var minAge = people.Min(person => person.age);
     Console.WriteLine(minAge);
     
     Console.ReadKey();
}
```

##### 輸出結果
1. 1
2. 15

#### 簡單實作自己的 Min
不考慮集合中存在 null 的情況.
```C#
public static TSource MyMin<TSource>(this IEnumerable<TSource> source)
{
     if (source == null) throw new Exception("null source");

     var enumerator = source.GetEnumerator();
     TSource value = default;
     if (enumerator.MoveNext())
     {
          Comparer<TSource> comparer = Comparer<TSource>.Default;
          bool hasValue = false;
          do
          {
               if (hasValue == false || comparer.Compare(value, enumerator.Current) > 0)
               {
                    value = enumerator.Current;
                    hasValue = true;
               }
          } while (enumerator.MoveNext());
     }
     return value;
}

public static TResult MyMin<TSource, TResult>(this IEnumerable<TSource> source, Func<TSource, TResult> func)
{
     if (source == null) throw new Exception(" null source");

     var enumerator = source.GetEnumerator();
     var value = default(TResult);
     if (enumerator.MoveNext())
     {
          var comparer = Comparer<TResult>.Default;
          bool hasValue = false;
          do
          {
               var temp = func(enumerator.Current);
               if (hasValue == false || comparer.Compare(value, temp) > 0)
               {
                    value = temp;
                    hasValue = true;
               }
          } while (enumerator.MoveNext());
     }
     return value;
}
```
##### 假設你希望取得集合中某個屬性值最小的物件 , 或許可以自己寫一個擴充方法. 如下.
```C#
public static TSource MyMin<TSource, TKey>(this IEnumerable<TSource> source, Func<TSource, TKey> selector)
{
     if (source == null) throw new Exception("null source");

     var enumerator = source.GetEnumerator();
     if (enumerator.MoveNext())
     {
          var comparer = Comparer<TKey>.Default;
          var minKey = selector(enumerator.Current);
          var value = enumerator.Current;
          do
          {
               var key = selector(enumerator.Current);
               if (comparer.Compare(minKey, key) > 0)
               {
                    minKey = key;
                    value = enumerator.Current;
               }
          } while (enumerator.MoveNext());
          return value;
     }
     throw new Exception("no item");
}
```
---

### [Max](https://docs.microsoft.com/zh-tw/dotnet/api/system.linq.enumerable.max?view=netframework-4.8)
回傳序列中指定項目的最大值.

常用的多載形式如下(不只)
1. TSource Max<TSource>(IEnumerable<TSource>)	
    - Returns the maximum value in a generic sequence.
2. TResult Max<TSource,TResult>(IEnumerable<TSource>, Func<TSource,TResult>)	
    - invokes a transform function on each element of a generic sequence and returns the maximum resulting value.
#### 使用時機
1. 當集合元素類型為 Value Type , 使用建構式 1 取出集合中的最大值
2. 當集合元素類型為 Reference Type , 使用建構式 2 取出集合中指定屬性的最大值.
#### Max 的用處
找出 list 中最大的數字 , 以及 people 中最大的年紀. 
```C#
public static List<int> GetList() => new List<int> { 5, 6, 9, 1, 3 };

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
     var list = GetList();
     var maxNum = list.Max();
     Console.WriteLine(minNum);
     
     var people = GetPeople();
     var maxAge = people.Max(person => person.age);
     Console.WriteLine(minAge);
     
     Console.ReadKey();
}
```
##### 輸出結果
1. 9
2. 74
#### 簡單實作自己的 Max
不考慮集合中存在 null 的情況.
```C#
public static TSource MyMax<TSource>(this IEnumerable<TSource> source)
{
     if (source == null) throw new Exception("null source");

     var enumerator = source.GetEnumerator();
     TSource value = default;
     if (enumerator.MoveNext())
     {
          Comparer<TSource> comparer = Comparer<TSource>.Default;
          bool hasValue = false;
          do
          {
               if (hasValue == false || comparer.Compare(value, enumerator.Current) < 0)
               {
                    value = enumerator.Current;
                    hasValue = true;
               }
          } while (enumerator.MoveNext());
     }
     return value;
}

public static TResult MyMax<TSource, TResult>(this IEnumerable<TSource> source, Func<TSource, TResult> func)
{
     if (source == null) throw new Exception(" null source");

     var enumerator = source.GetEnumerator();
     var value = default(TResult);
     if (enumerator.MoveNext())
     {
          var comparer = Comparer<TResult>.Default;
          bool hasValue = false;
          do
          {
               var temp = func(enumerator.Current);
               if (hasValue == false || comparer.Compare(value, temp) < 0)
               {
                    value = temp;
                    hasValue = true;
               }
          } while (enumerator.MoveNext());
     }
     return value;
}
```

---

### [Sum](https://docs.microsoft.com/zh-tw/dotnet/api/system.linq.enumerable.sum?view=netframework-4.8)
回傳序列中指定項目的總和

我在MSDN文件找不到泛型 TResult 的 API 格式 , 不管 , 反正就記成只有下列兩個形式吧 XD
1. TSource Sum<TSource>(this IEnumerable<TSource>)
2. TResult Sum<TSource,TResult>(this IEnumerable<TSource>, Func<TSource,TResult>)
    - 雖然是這麼寫 , 但實際上此實作是將 TResult 可能的類型都實作一次. 因為 TResult 是泛型呀~~ , 無法做 + 運算 , 除非自己重載運算子 Orz.

#### 使用時機
1. 當集合元素類型為 Value Type , 使用建構式 1 取出集合總和
2. 當集合元素類型為 Reference Type , 使用建構式 2 取出集合中指定屬性的總和.
#### Sum 的用處
找出 list 中所有數字的總和 , 以及 people 中所有年紀的總和. 
```C#
public static List<int> GetList() => new List<int> { 5, 6, 9, 1, 3 };

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
     var list = GetList();
     var minNum = list.Sum(num => num);
     Console.WriteLine(minNum);

     var people = GetPeople();
     var minAge = people.Sum(item => item.age);
     Console.WriteLine(minAge);

     Console.ReadKey();
}
```

##### 輸出結果
1. 24
2. 120

#### 簡單實作自己的 Sum
實際上 LinQ 對於 Sum 的實作並沒有使用泛型 , 可能是考慮到效能吧?!
但我不想寫那麼多個 Orz.
```C#
// 實務上不建議用 dynamic 實作.
public static TSource MySum<TSource>(this IEnumerable<TSource> source)
{
     dynamic sum = 0;
     foreach (var item in source)
     {
          sum += item;
     }
     return sum;
}

public static TResult MySum<TSource, TResult>(this IEnumerable<TSource> source, Func<TSource, TResult> selector)
{
     dynamic sum = 0;
     foreach (var item in source)
     {
          sum += selector(item);
     }
     return sum;
}

// 實際上沒有這個形式
public static TResult MySum<TSource, TResult>(this IEnumerable<TSource> source, Func<TSource, TResult> selector, Func<TResult, TResult, TResult> add)
{
     TResult sum = default;
     foreach (var item in source)
     {
          sum = add(sum, selector(item));
     }
     return sum;
}

```
---

### [Average](https://docs.microsoft.com/zh-tw/dotnet/api/system.linq.enumerable.average?view=netframework-4.8)
回傳序列中指定項目的平均

我在MSDN文件也找不到泛型 TResult 的 API 格式 , 一樣不管 , 反正就記成只有下列兩個形式吧 XD

1. double Average<TSource>(this IEnumerable<TSource>)
2. double Average<TSource,TResult>(this IEnumerable<TSource>, Func<TSource,TResult>)
    - 雖然是這麼寫 , 但實際上此實作是將 TResult 可能的類型都實作一次. 因為 TResult 是泛型呀~~ , 無法做 + 運算 , 除非自己重載 Orz.


#### 使用時機
1. 當集合元素類型為 Value Type , 使用建構式 1 取出集合平均
2. 當集合元素類型為 Reference Type , 使用建構式 2 取出集合中指定屬性的平均.
#### Average 的用處
找出 list 中數字的平均 , 以及 people 中年紀的平均. 
```C#
public static List<int> GetList() => new List<int> { 5, 6, 9, 1, 3 };

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
     var list = GetList();
     var minNum = list.Average(num => num);
     Console.WriteLine(minNum);

     var people = GetPeople();
     var minAge = people.Average(item => item.age);
     Console.WriteLine(minAge);

     Console.ReadKey();
}
```
##### 輸出結果
1. 4.8
2. 40
#### 簡單實作自己的 Average
```C#
public static double MyAverage<TSource>(this IEnumerable<TSource> source)
{
     dynamic sum = 0;
     int count = 0;
     foreach (var item in source)
     {
          sum += item;
          count++;
     }
     return count > 0 ? 1.0 * sum / count : throw new Exception(" divide zero") ;
}

public static double MyAverage<TSource, TResult>(this IEnumerable<TSource> source, Func<TSource, TResult> selector)
{
     dynamic sum = 0;
     int count = 0;
     foreach (var item in source)
     {
          sum += selector(item);
          count++;
     }
     return count > 0 ? 1.0 * sum / count : throw new Exception(" divide zero") ;
}
```
---

### [Count](https://docs.microsoft.com/zh-tw/dotnet/api/system.linq.enumerable.count?view=netframework-4.8)
回傳序列中滿足指定條件的元素數量. **指定條件可以是無條件.**
1. int Count<TSource>(IEnumerable<TSource>)	
    - Returns the number of elements in a sequence.
2. int Count<TSource>(IEnumerable<TSource>, Func<TSource,Boolean>)	
    - Returns a number that represents how many elements in the specified sequence satisfy a condition.
#### 使用時機
1. **想知道集合中元素的數量.** 建議使用多載 1 . 另外若是集合的型態有 Count 屬性可以使用 ( 例如 List、Array ) , 則不建議使用 LinQ 的 Count 方法 , 原因是 Count 方法的結果是透過走訪所有集合成員得到.
2. **想要知道集合中滿足指定條件的元素的數量**. 可使用多載 2 .
#### Count 的用處
找出 list 中大於 5 的個數 , 以及 people 的成員個數. 
```C#
public static List<int> GetList() => new List<int> { 5, 6, 9, 1, 3 };

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
     var list = GetList();
     var minNum = list.Count(num => num > 5);
     Console.WriteLine(minNum);

     var people = GetPeople();
     var minAge = people.Count();
     Console.WriteLine(minAge);

     Console.ReadKey();
}
```

##### 輸出結果
1. **2** , 比 5 大的數字有 6 以及 9 兩個.
2. **3** , people 有三個人.


#### 簡單實作自己的 Count
```C#
public static int MyCount<TSource>(this IEnumerable<TSource> source)
{
     int count = 0;
     foreach (var item in source)
     {
          checked
          {
               count++;
          }
     }
     return count;
}

public static int MyCount<TSource>(this IEnumerable<TSource> source, Func<TSource, bool> predicate)
{
     int count = 0;
     foreach (var item in source)
     {
          checked
          {
               if (predicate(item))
               {
                    count++;
               }
          }
     }
     return count;
}
```
---

### 總結
LinQ 可以用很簡單的語法解決原本需要好幾次迴圈才能解決的問題.
另外硬要講缺點的話 , 大概是會一直點點點下去 , 會寫得很長.
我自己也還在想有沒有比較好的換行邏輯.

最後用兩個例子作為總結.

#### 想知道不同部門關於薪水的各項統計數據
##### 程式碼
```C#
var data = new[]
{
     new { DepartmentID = "屌炸天", Name = "Gamma", Salary = 42000 },
     new { DepartmentID = "酷到炸", Name = "jeff", Salary = 30000 },
     new { DepartmentID = "屌炸天", Name = "mary", Salary = 25000 },
     new { DepartmentID = "旋風爆", Name = "eric", Salary = 5000 },
     new { DepartmentID = "酷到炸", Name = "lisa", Salary = 65000 },
     new { DepartmentID = "旋風爆", Name = "city", Salary = 95000 },
     new { DepartmentID = "爆炸酷", Name = "linda", Salary = 12500 },
     new { DepartmentID = "爆炸酷", Name = "qoooo", Salary = 35600 },
     new { DepartmentID = "旋風爆", Name = "John", Salary = 53210 },
};

var query = data.GroupBy(item => item.DepartmentID).Select(group =>
                new
                {
                    DepartmentID = group.Key,
                    DepartCount = group.Count(),
                    TotalSalary = group.Sum(groupItem => groupItem.Salary),
                    AverageSalary = group.Average(groupItem => groupItem.Salary),
                    MaxSalary = group.Max(groupItem => groupItem.Salary),
                    MinSalary = group.Min(groupItem => groupItem.Salary),
                    EmployeeNames = group.Select(groupItem => groupItem.Name)
                }
            );

foreach (var item in query)
{
     Console.WriteLine($"部門編號 : {item.DepartmentID} , 部門人數 : {item.DepartCount}");
     Console.WriteLine($"薪水和 : {item.TotalSalary} , 平均薪水 : {item.AverageSalary} , 最高薪水 : {item.MaxSalary} , 最低薪水 : {item.MinSalary}");
     item.EmployeeNames.ToList().ForEach(name => Console.WriteLine($"員工姓名 : {name}  "));
     Console.WriteLine();
}
```
##### 輸出結果
![](https://i.imgur.com/88iMY5a.png)

---

#### 薪水的次數分配表
```C#
var data = new[]
{
     new { DepartmentID = "屌炸天", Name = "Gamma", Salary = 42000 },
     new { DepartmentID = "酷到炸", Name = "jeff", Salary = 30000 },
     new { DepartmentID = "屌炸天", Name = "mary", Salary = 25000 },
     new { DepartmentID = "旋風爆", Name = "eric", Salary = 5000 },
     new { DepartmentID = "酷到炸", Name = "lisa", Salary = 65000 },
     new { DepartmentID = "旋風爆", Name = "city", Salary = 95000 },
     new { DepartmentID = "爆炸酷", Name = "linda", Salary = 12500 },
     new { DepartmentID = "爆炸酷", Name = "qoooo", Salary = 35600 },
     new { DepartmentID = "旋風爆", Name = "John", Salary = 53210 },
};

var intervals = new double[] { 80000, 50000, 30000 };
var query = data.GroupBy(employee => intervals.FirstOrDefault(interval => employee.Salary >= interval))
                .Select( groupedDate => new 
                                        { 
                                            groupedDate.Key, 
                                            Count = groupedDate.Count(),
                                        }
                       ).OrderBy(item => item.Key);

foreach (var item in query)
{
     var label = item.Key != 0 ? $"薪水高於{item.Key.ToString()}的人數" : $"薪水低於{intervals[intervals.Length - 1]}的人數";
     Console.WriteLine($"{label} : { item.Count}");
}
```
##### 輸出結果
![](https://i.imgur.com/vxsFJYd.png)

---

### Thank you! 

You can find me on

- [GitHub](https://github.com/s0920832252)
- [Facebook](https://www.facebook.com/fourtune.chen)

若有謬誤 , 煩請告知 , 新手發帖請多包涵

# :100: :muscle: :tada: :sheep: 
