---
tags: LinQ , C# , Generation Operators
---

# Repeat  & Range & Empty & DefaultIfEmpty
Generation Operators 可以幫助我們快速產生常見樣式的資料集合. 也就是說這些方法不需要提供類似型式的資料集合作為方法的輸入參數 , 並且回傳會 IEnumerable<T> 型態的資料集合. 例如 : 產生一個陣列後不需要手動地透過迴圈去塞值 , 而是可以透過一個方法產生這樣的陣列.

隸屬於 Generation Operators 的方法有
1. Repet ( Enumerable 類別內的**靜態**方法 )
1. Range ( Enumerable 類別內的**靜態**方法 )
1. Empty ( Enumerable 類別內的**靜態**方法 )
1. DefaultIfEmpty ( Enumerable 類別內的擴充方法 )

### Repeat 
Repeat 方法可以產生指定數量重複值的序列.
```C#
public static IEnumerable<TResult> Repeat(TResult element, int count);
```
- element : 需要重複的值
- count : 重複的數量

#### 使用時機
- 需要快速產生具有重複值的序列.
    - 例如 : 快速初始化一個裡面全部塞某個值的陣列.
        - var array = new int[]{1,1,1,1,1}
        - 雖然我們可以透過上述語法初始化一個陣列 , 但卻必須自己手動的填入內容值. 但使用 Repeat 方法卻僅需要填入兩個參數即可 (雖然型態是 Enumerable , 之後需要再使用 toArray() ) , 並且陣列大小將是動態決定的 , 由第二個參數 count 來決定.
#### Repeat 的用法
```C#
// 印出 I am City ~~ 四次
static void Main(string[] args)
{
     IEnumerable<string> strings = Enumerable.Repeat("I am City ~~", 4);
     foreach (var str in strings)
     {
          Console.WriteLine(str);
     }
     Console.ReadKey();
}
```
##### 輸出結果
I am City ~~    
I am City ~~    
I am City ~~    
I am City ~~

```C#
// 印出5層的小星星
static void Main(string[] args)
{
     for (var i = 1; i <= 5; i++)
     {
          // Join 使用 "" 作為每一個 * 的分隔符號. 將集合成員合併為一個字串
          string star = string.Join("", Enumerable.Repeat("*", i));
          Console.WriteLine(star);
     }
     Console.ReadKey();
}
```
##### 輸出結果
\*    
\**    
\***    
\****    
\*****    

#### 簡單實作自己的 Repeat
```C#
public static IEnumerable<TResult> MyRepeat<TResult>(TResult element, int count)
{
     if (count < 0)
     {
          throw new Exception("count is less than zero");
     }
     return MyRepeatIterator(element, count);
}

private static IEnumerable<TResult> MyRepeatIterator<TResult>(TResult element, int count)
{
     for (int i = 0; i < count; i++)
     {
          yield return element;
     }
}
```

---

### Range
相對於 Repeat 能夠產生以某個值作為重複值的序列 , Range 能夠產生的序列是由**遞增排序的數字**組成.

```C#
public static IEnumerable<int> Range(int start, int count);
```
- start : 序列的起始值
- count : **序列的數量** (不是終止值 :scream: )

#### 使用時機
- 需要快速產生具有遞增排序的數字的序列
    - 通常之後會再搭配 select 去將數字轉成對應的值.
        - ```C#
            // 製造 pow .
            var pow = Enumerable.Range(1,5).Select(n => n * n).ToList()
          ```
- 可以用來取代基本形式的 for 迴圈
    - 與 select 搭配 , 甚至可以取代 for 迴圈. 有點像是 Python 的感覺. 至於哪個好 , 就見仁見智了XD
      
        - ```C#
            // 取出位置 2,4,6 的值
            var arr = new int[8];
            var list = Enumerable.Range(1,3).Select(n => arr[n * 2]).ToList()
          ```

#### Range 的用法
```C#
// 印出 5 層的小星星
static void Main(string[] args)
{
     var stars = Enumerable.Range(1, 5).Select(i => string.Join("", Enumerable.Repeat("*", i)))
                 .ToList();
     stars.ForEach(star => Console.WriteLine(star))            
     Console.ReadKey();
}
```
##### 輸出結果
\*    
\**    
\***    
\****    
\*****  


#### 簡單實作自己的 Range
```C#
public static IEnumerable<int> MyRange(int start, int count)
{
     long max = (long)count + start - 1;
     if (count < 0 || max > int.MaxValue)
     {
          throw new Exception("out of range");
     }
     return MyRangeIterator(start, count);
}

private static IEnumerable<int> MyRangeIterator(int start, int count)
{
     for (int i = 0; i < count; i++)
     {
          yield return start + i;
     }
}
```


---

### Empty
回傳一個 IEnumerable<TResult> 型態的空序列.
```C#
public static IEnumerable<TResult> Empty<TResult>();
```
#### 使用時機
1. 臨時需要一個 IEnumerable<T> 型態的空序列.
2. 採用獨體模式(Singleton Pattern)的設計 , 因此每次呼叫都是同一個實體. 所以即使有十個變數 , 仍只會 new 一次. 可以減少 GC 的介入. 提高效能.
   - ```C#            
     var set = Enumerable.Empty<string>();
     var set2 = Enumerable.Empty<string>();
     Console.WriteLine($"{set == set2} {set.Equals(set2)} {ReferenceEquals(set, set2)}");
     ```
   - ![](https://i.imgur.com/96V8c3N.png)
  
#### [Empty 的用法](https://docs.microsoft.com/zh-tw/dotnet/api/system.linq.enumerable.empty?view=netframework-4.8)
```C# 
string[] names1 = { "Hartono, Tommy" };
string[] names2 = { "Adams, Terry", "Andersen, Henriette Thaulow",
                      "Hedlund, Magnus", "Ito, Shu" };
string[] names3 = { "Solanki, Ajay", "Hoeing, Helge",
                      "Andersen, Henriette Thaulow",
                      "Potra, Cristina", "Iallo, Lucio" };

List<string[]> namesList =
    new List<string[]> { names1, names2, names3 };

// Only include arrays that have four or more elements
IEnumerable<string> allNames =
    namesList.Aggregate(Enumerable.Empty<string>(),
    (current, next) => next.Length > 3 ? current.Union(next) : current);

foreach (string name in allNames)
{
    Console.WriteLine(name);
}

```
##### 輸出結果
 Adams, Terry    
 Andersen, Henriette Thaulow    
 Hedlund, Magnus    
 Ito, Shu    
 Solanki, Ajay    
 Hoeing, Helge    
 Potra, Cristina    
 Iallo, Lucio    
 
#### 簡單實作自己的 Empty
```CE        
public static IEnumerable<TResult> MyEmpty<TResult>() => Array.Empty<TResult>();
```

#### 參考資源
1. [Enumerable.Empty() vs new ‘IEnumerable’() – what’s better?](https://agirlamonggeeks.com/2019/03/22/enumerable-empty-vs-new-ienumerable-whats-better/)
1. [Is it better to use Enumerable.Empty<T>() as opposed to new List<T>() to initialize an IEnumerable<T>?](https://stackoverflow.com/questions/1894038/is-it-better-to-use-enumerable-emptyt-as-opposed-to-new-listt-to-initial)
1. [new object[] {} vs Array.Empty<object>()](https://stackoverflow.com/questions/53071157/new-object-vs-array-emptyobject)
1. [In C#, should I use string.Empty or String.Empty or “” to intitialize a string?](https://stackoverflow.com/questions/263191/in-c-should-i-use-string-empty-or-string-empty-or-to-intitialize-a-string)

---

### DefaultIfEmpty
如果序列是空的 , 則傳回塞入一個預設值的序列.

1. DefaultIfEmpty<TSource>(this IEnumerable<TSource> source)	
    - 傳回指定之序列的項目；如果序列是空的，則傳回單一集合中型別參數的預設值。
1. DefaultIfEmpty<TSource>(this IEnumerable<TSource> source, TSource defaultValue)
    - 傳回指定之序列的項目；如果序列是空的，則傳回單一集合中型別參數的預設值。

- 這兩個過載方法的差異在預設值是使用 TSource 的預設值 還是由使用者指定.

#### 使用時機
- 需要序列必定不為空的時候 
- 需要依據不同情況給予預設值
#### DefaultIfEmpty 的用法
```C#
static void Main(string[] args)
{
     List<(int id, string name)> vs = new List<(int id, string name)>();
     List<(int id, string name)> vs2 = new List<(int id, string name)>() { (5, "大王"), (1, "小黃") };
     var collect = vs.DefaultIfEmpty(); // int 預設值 => 0 , string 預設值 => ""
     var collect2 = vs.DefaultIfEmpty<(int id, string name)>((-1, "測試"));
     var collect3 = vs2.DefaultIfEmpty<(int id, string name)>((-1, "測試"));

     collect.ToList().ForEach(coll => Console.WriteLine($"{coll.id} {coll.name}"));
     Console.WriteLine();
     collect2.ToList().ForEach(coll => Console.WriteLine($"{coll.id} {coll.name}"));
     Console.WriteLine();
     collect3.ToList().ForEach(coll => Console.WriteLine($"{coll.id} {coll.name}"));
     Console.ReadKey();
}
```
##### 輸出結果
![](https://i.imgur.com/wU9yMfy.png)

#### 簡單實作自己的 DefaultIfEmpty
```C#
public static IEnumerable<TSource> MyDefaultIfEmpty<TSource>(this IEnumerable<TSource> source)
{
     return MyDefaultIfEmpty(source, default);
}

public static IEnumerable<TSource> MyDefaultIfEmpty<TSource>(this IEnumerable<TSource> source, TSource defaultValue)
{
     if (source == null) throw new Exception("source is null");
     return MyDefaultIfEmptyIterator(source, defaultValue);
}

public static IEnumerable<TSource> MyDefaultIfEmptyIterator<TSource>(IEnumerable<TSource> source, TSource defaultValue)
{
     var enumerator = source.GetEnumerator();
     bool hasItem = enumerator.MoveNext();
     if (hasItem)
     {
          do
          {
               yield return enumerator.Current;
          } while (enumerator.MoveNext());
     }
     else
     {
          yield return defaultValue;
     }
}
```
### Summary
- 如果想獲得一個特定型式的序列 , 也許可以使用
    - Enumerable.Repeat
    - Enumerable.Range
- 如果想獲得一個空序列 , 使用 Enumerable.Empty<T>
- 如果想给空序列一個預設值，使用 Enumerable.DefaultIfEmpty







---

### Thank you! 

You can find me on

- [GitHub](https://github.com/s0920832252)
- [Facebook](https://www.facebook.com/fourtune.chen)

若有謬誤 , 煩請告知 , 新手發帖請多包涵

# :100: :muscle: :tada: :sheep: 
