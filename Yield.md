---
tags: LinQ, LinQ基礎 , C#
---

# LINQ基礎 - Yield

### 閒話543
C#的語法糖真的是多到一個爆炸呀:laughing: 

---

### [Iterators](https://docs.microsoft.com/zh-tw/dotnet/csharp/programming-guide/concepts/iterators) & [yield](https://docs.microsoft.com/zh-tw/dotnet/csharp/language-reference/keywords/yield)

> yield 在 .Net 2.0 時提供

> 當編譯器偵測到迭代器時，它會**自動產生**IEnumerator 或 IEnumerator<T> 介面的 Current、MoveNext 和 Dispose 方法。

> 迭代器方法使用 yield return 陳述式，一次傳回一個項目。 當到達 yield return 陳述式時，系統會記住程式碼中的目前位置。下次呼叫迭代器函式時，便會從這個位置重新開始執行。

> 可以使用 yield break 陳述式結束反覆項目

> 每次反覆運算foreach迴圈 (或直接呼叫 IEnumerator.MoveNext)，下一個迭代器程式碼主體都會在上一個 yield return 陳述式之後繼續。 然後繼續執行至下一個 yield return 陳述式，直到達到迭代器主體結尾，或遇到 yield break 陳述式為止。


#### 範例1 - 使用 yield 完成走訪功能

```C#
public class DaysOfTheWeek : IEnumerable
{
     readonly string[] m_Days = { "Sun", "Mon", "Tue", "Wed", "Thr", "Fri", "Sat" };
     public IEnumerator GetEnumerator()
     {
          for (int i = 0; i < m_Days.Length; i++)
          {
               yield return m_Days[i];
          }
     }
}
```

```C#
static void Main(string[] args)
{
     DaysOfTheWeek week = new DaysOfTheWeek();

     foreach (string day in week)
     {
          Console.Write(day + " ");
     }

     Console.WriteLine();

     var emumerator = week.GetEnumerator();
     while (emumerator.MoveNext())
     {
          Console.Write(emumerator.Current + " ");
     }

     Console.ReadKey();
}
```

##### 輸出結果
![](https://i.imgur.com/NGActqW.png)


#### 範例2 - 不使用 yield 完成走訪功能

```C#
public class DaysOfTheWeek : IEnumerable
{
     readonly string[] m_Days = { "Sun", "Mon", "Tue", "Wed", "Thr", "Fri", "Sat" };
     public IEnumerator GetEnumerator()
     {
          return new DaysOfTheWeekEnumerator(m_Days);
     }
}
```

```C#
public class DaysOfTheWeekEnumerator : IEnumerator
{
     private int _index = -1;
     private string[] _Days;

     public DaysOfTheWeekEnumerator(string[] m_Days) => _Days = m_Days;
     public object Current => _Days[_index];
     public bool MoveNext() => ++_index < _Days.Length;
     public void Reset() => _index = -1;
}
```
```C#
static void Main(string[] args)
{
     DaysOfTheWeek week = new DaysOfTheWeek();

     foreach (string day in week)
     {
          Console.Write(day + " ");
     }

     Console.WriteLine();

     var emumerator = week.GetEnumerator();
     while (emumerator.MoveNext())
     {
          Console.Write(emumerator.Current + " ");
     }

     Console.ReadKey();
}
```
##### 輸出結果
![](https://i.imgur.com/5LnI64S.png)


由上面兩個範例 我們可以知道
- yield 可幫助自動產生走訪器 , 因此我們**不必自行定義一個實現 IEnumerator 的類別**.

#### 執行順序

##### 範例 - 走訪 list 內成員

```C#
public class CityManager
{
     public static IEnumerable<int> GetEnumerable(List<int> _numbers)
     {
          foreach (var num in _numbers)
          {
               Console.WriteLine($"執行yield return前, 數值為:{num}");
               if (num == 3)
               {
                    Console.WriteLine("數值為3, 呼叫yield break");
                    yield break;
               }
               yield return num;
               Console.WriteLine($"執行yield return後的下一行, 數值為:{num}");
          }
     }
}
```
```C#
static void Main(string[] args)
{
     var enumerable = CityManager.GetEnumerable(new List<int>(){ 6, 5, 3, 13, 9, 8, 7 });

     Console.WriteLine($"執行了foreach之前");
     foreach (var num in enumerable)
     {
          Console.WriteLine($"foreach - 現在的值是{num}");
          Console.WriteLine($"數字:{num}處理完畢.跳下一個數字");
          Console.WriteLine();
     }

     Console.WriteLine($"------------分隔線----------------");

     var enumerator = enumerable.GetEnumerator();
     Console.WriteLine($"執行了while之前");
     while (enumerator.MoveNext())
     {
          Console.WriteLine($"while - 現在的值是{enumerator.Current}");
          Console.WriteLine($"數字:{enumerator.Current}處理完畢.跳下一個數字");
          Console.WriteLine();
     }

     Console.ReadKey();
}
```

##### 輸出結果

![](https://i.imgur.com/Js81Jn4.png)




由上面的範例 我們可以知道執行順序是
1. foreach & in 在執行時會呼叫 MoveNext() , 然後取出 Current 的值
1. 取出 Current 的值後 , 執行 foreach 主體.
1. foreach 要取下一個 Item 時 , 會呼叫 MoveNext() , 此時會從剛剛的 yield return 處下一行開始執行.
1. 上述三個動作會重複執行 , 直到走訪完畢或是碰到 yield break 為止.

---


### 總結

- 使用 yield return 可以輕鬆建立一個 IEnumerable<T> 的資料集合.
- 執行 yield return 後 , 下一次被呼叫時 , 會繼續從上一次的 yield return 後開始執行.
- 呼叫 yield break 後 , 會離開 foreach 主體.
- 如果一個區塊(block)中有 yield 陳述式，則此區塊就叫做 Iterator Block
    - 一個方法的區塊如果是 Iterator Block , 則它的回傳值會是 IEnumerable 或是IEnumerator. 請參考[C# 語言規格-類別](https://docs.microsoft.com/zh-tw/dotnet/csharp/language-reference/language-specification/classes#iterators)
> Iterator 會產生一系列的值，這些都是相同的型別。 這個型別稱為 iterator 的yield 型別。
> - 傳回IEnumerator object或IEnumerable的反覆運算器產生類型為 object。
> - 傳回IEnumerator<T> T或IEnumerable<T>的反覆運算器產生類型為 T。
> - ![](https://i.imgur.com/fHr22mu.png)

    
    

---

### 補充 - 不使用 yield 實作走訪 list 內成員
```C#
public class CityManger
{
     public static IEnumerable<T> GetEnumerable<T>(List<T> _numbers) => new Enumerable<T>(_numbers);
}

public class Enumerable<T> : IEnumerable<T>
{
     private readonly List<T> _list;
     public Enumerable(List<T> list) => _list = list;

     IEnumerator<T> IEnumerable<T>.GetEnumerator() => new Enumerator<T>(_list);
     IEnumerator IEnumerable.GetEnumerator() => throw new NotImplementedException();
}

public class Enumerator<T> : IEnumerator<T>
{
     private List<T> _list;
     private int _index = -1;
     public Enumerator(List<T> list) => _list = list;

     T IEnumerator<T>.Current => _list[_index];
     public object Current => throw new NotImplementedException();

     public bool MoveNext()
     {
          if (_index != -1 && _list[_index].Equals(3))
               return false;
          return ++_index < _list.Count;
     }

     public void Reset() => _index = -1;
     public void Dispose() => _list = null;
}
```

![](https://i.imgur.com/F5BCyIJ.png)


---






### Thank you! 

You can find me on

- [GitHub](https://github.com/s0920832252)
- [Facebook](https://www.facebook.com/fourtune.chen)

若有謬誤 , 煩請告知 , 新手發帖請多包涵

# :100: :muscle: :tada: :sheep: 
