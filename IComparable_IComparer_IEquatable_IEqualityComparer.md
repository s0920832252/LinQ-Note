---
tags: LinQ, LinQ基礎 , C#
---

# LINQ基礎 - IEquatable vs IEqualityComparer And IComparable vs IComparer 
這四個介面通常是作用在參考型別. 
- 介面 IEquatable 和 IEqualityComparer 是用來檢查兩個參考型別是否**相等**. 
    - 在取 Union , SequenceEqual , ToDictionary 等等的方法 , 我們可能會使用到它. 因為我們需要判兩個參考型別的物件是否相等. 
- 介面 IComparable 和 IComparer 是用來判斷兩個對象**誰大誰小**.
    - IComparable 和 IComparer 通常用於對一群參考型別的物件進行排序.
- 在 Linq 方法中實際上會用到的是 **IEqualityComparer** 以及 **IComparer**
    - 另外兩個是順便 :crystal_ball: 

---

### IEquatable
IEquatable 是在 C# 2.0 加入的. 它的主要用途是**提供實作此介面的類別物件具有判斷自己與另一個與自己相同類別的物件是否相等的能力**. 所以實現 IEquatable 介面的類別都必須實作 Equals 方法. ``` bool Equals(T other); ```


``` C#
class Employee : IEquatable<Employee>
{
     public int Id { get; set; }
     public string Name { get; set; }
     public bool Equals(Employee other)
     {
          return (Id == other.Id && Name == other.Name);
     }       
     // 可以不 override 
     public override bool Equals(object obj)
     {
          Employee employee = (Employee)obj;
          return (Id == employee.Id && Name == employee.Name);
     }
     // 可以不 override 
     public override int GetHashCode()
     {
           return Id.GetHashCode() ^ Name.GetHashCode();
     }
}

static void Main(string[] args)
{
     Employee ob1 = new Employee();
     Employee ob2 = new Employee();
     Console.WriteLine(ob1.Equals(ob2)); // true

     // 如果沒 override bool Equals(object obj) , 
     // 會使用Object.Equals - 依據記憶體位址是否相同(回傳 false)
     Employee a = new Employee();
     object b = new Employee();
     Console.WriteLine(a.Equals(b)); // true
     
     List<Employee> employees = new List<Employee> { 
         new Employee { Id = 1, Name = "王" 
     }};
     List<Employee> employees2 = new List<Employee> { 
         new Employee { Id = 3, Name = "綠" } , 
         new Employee { Id = 1, Name = "王" } 
     };
     foreach (var emp in employees.Union(employees2))
     {
          Console.Write(" " + emp.Name); // 輸出 : 王 綠
          // 若是沒有實作 bool Equals(Employee other) 會因為記憶體位址不同 , 
          // 被認為是不同實體 , 而輸出 : 王 綠 王
     }
}
```
微軟的文件建議若是我們實作 IEquatable 介面 , 則順便 override Equals() & GetHashCode(). 以避免未預期的結果發生.

---
### IEqualityComparer
IEqualityComparer是在 C# 2.0 加入的. 實作該介面的類別物件**具有判斷某相同類別的兩個物件是否相等的能力**. 實作IEquatable 的類別物件自身就能夠判斷自己是否與另外一個物件相等. 而實作 IEqualityComparer 的類別物件則是專門負責比較的比較器 , 通常是以不希望更改原類別為前提 , 但又希望可以比較該類別是否相等的狀況下使用.

IEqualityComparer 必須實作下列兩個方法 
1. bool Equals(T x, T y) : 
    - 判斷兩物件相等的邏輯
3. int GetHashCode(T obj) : 
    - 使用 hash 的方式加快比較相等的速度(在 key 相同的情況才會再進去 Equals去判斷兩物件是否相等)

```C#
class Employee
{
    public int Id { get; set; }
    public string Name { get; set; }
}

class EmployeeComparer : IEqualityComparer<Employee>
{
    public bool Equals(Employee x, Employee y)
    {
        return x.Name.Equals(y.Name) && x.Id.Equals(y.Id);
    }

    public int GetHashCode(Employee obj)
    {
        return obj.Name.GetHashCode() ^ obj.Id.GetHashCode();
    }
}

static void Main(string[] args)
{
    List<Employee> employees = new List<Employee> { 
                                  new Employee { Id = 1, Name = "王" } 
                               };
    List<Employee> employees2 = new List<Employee> { 
                                  new Employee { Id = 2, Name = "綠" }, 
                                  new Employee { Id = 1, Name = "王" } 
                                };
    foreach (var emp in employees.Union(employees2, new EmployeeComparer()))
    {
        Console.Write(" " + emp.Name);
        // 輸出 王 綠
        // 若沒有使用 EmployeeComparer 
        // 則輸出 王 綠 王
    }
    Console.ReadKey();
}
```
---
### IComparable
IComparable 必須實現 CompareTo 方法
```C#
int CompareTo(Object obj) // 比較邏輯
```
1. 當呼叫物件 < 參數物件obj , 回傳負數
2. 當呼叫物件 > 參數物件obj , 回傳正數
3. 當呼叫物件 = 參數物件obj , 回傳零

```C#
class Employee : IComparable<Employee>
{
     public int Id { get; set; }
     public string Name { get; set; }

     public int CompareTo(Employee other)
     {
          return Id.CompareTo(other.Id);
     }
}

static void Main(string[] args)
{
     List<Employee> employees = new List<Employee> { 
             new Employee { Id = 3, Name = "王" }, 
             new Employee { Id = 1, Name = "綠" }, 
             new Employee { Id = 2, Name = "王" }, 
     };
     employees.Sort(); // 沒實作 IComparable 會跳例外XD
     foreach (var employee in employees)
     {
          Console.WriteLine($"{employee.Id} {employee.Name}");
     }
     Console.ReadKey();
}
```
##### 輸出結果
Id = 1 Name = 綠           
Id = 2 Name = 王           
Id = 3 Name = 王           

---

### IComparer
IComparer 使用在參考型態的自定義排序. (在 C# 3.5 前 , 可能在 Sort 方法的時候會使用)
但在 LinQ 的 ThenBy , OrderBy ...出來後 , 使用機率會少很多. 
LinQ 仍然有接受 IComparer 作為參數的方法.



```C#
int Compare(T x, T y);
```
- 若 x < y 回傳 -1
- 若 x == y 回傳 0
- 若 x > y 回傳 1

```C#
class Employee
{
        public int Id { get; set; }
        public string Name { get; set; }
}

class EmployeeCompare : IComparer<Employee>
{
     public int Compare(Employee x, Employee y)
     {
          return x.Id.CompareTo(y.Id);
     }
}

static void Main(string[] args)
{
     List<Employee> employees = new List<Employee> { 
                                 new Employee { Id = 3, Name = "王" }, 
                                 new Employee { Id = 1, Name = "綠" }, 
                                 new Employee { Id = 2, Name = "王" } 
                                 };
     employees.Sort(new EmployeeCompare());
     foreach (var employee in employees)
     {
          Console.WriteLine($"{employee.Id} {employee.Name}");
     }
     Console.ReadKey();
}
```
##### 輸出結果
1. 綠
2. 王
3. 王

---

### 與 LinQ 的關聯
一些 LinQ 的方法可能會需要知道兩個項目是否相等或者需要比較兩個項目的大小.
- 需要知道兩個項目是否相等
ToDictionary , ToHashSet , ToLookup , Contains , Distinct , SequenceEqual , Except , GroupBy , GroupJoin , Intersect , Join , Union
- 需要比較兩個項目的大小
OrderBy , OrderByDescending , ThenBy , ThenByDescending , 

---
### 參考
1. [Equals vs IEqualityComparer, IEquatable<T>, IComparable, IComparer](https://yetanotherchris.dev/csharp/equals-vs-iequatable-iequalitycomparer-icomparable-icomparer/)
1. [Comparing Complex Type With ==, Equals, IEquatable and IComparable in C#](https://www.c-sharpcorner.com/UploadFile/78607b/comparing-complex-type-with-equals-iequatable-and-icomp/)
1. [IComparable, IComparer And IEquatable Interfaces In C#](https://www.c-sharpcorner.com/UploadFile/80ae1e/icomparable-icomparer-and-iequatable-interfaces-in-C-Sharp/)
1. [使用原則(5) - IComparable<T>, IEquatable<T>](https://ithelp.ithome.com.tw/articles/10187848)
1. [What is the difference between IEqualityComparer<T> and IEquatable<T>?](https://stackoverflow.com/questions/9316918/what-is-the-difference-between-iequalitycomparert-and-iequatablet)
1. [What's the difference between IComparable & IEquatable interfaces?](https://stackoverflow.com/questions/2410101/whats-the-difference-between-icomparable-iequatable-interfaces)
1. [[C#] IComparable 和 IComparer](http://jengting.blogspot.com/2015/02/c-icomparable-icomparer.html)
1. [Implementing IComparable and IEquatable safely and fully](https://www.codeproject.com/Tips/408711/Implementing-IComparable-and-IEquatable-safely-and)
1. [談談C#的相等性(1)](https://blog.opasschang.com/2019/02/19/equality_in_csharp_1/)
1. [The Right Way to do Equality in C#](http://www.aaronstannard.com/overriding-equality-in-dotnet/)

---

### 補充 : 製造泛型的 IEqualityComparer
```
class EqualityComparer<TSource> : IEqualityComparer<TSource> where TSource : class
{
     private PropertyInfo[] properties = null;
     public new bool Equals(TSource x, TSource y)
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
```


---
### Thank you! 

You can find me on

- [GitHub](https://github.com/s0920832252)
- [Facebook](https://www.facebook.com/fourtune.chen)

若有謬誤 , 煩請告知 , 新手發帖請多包涵

# :100: :muscle: :tada: :sheep: 
