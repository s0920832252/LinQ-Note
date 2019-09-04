---
tags: LinQ , C# , Restriction Operators
---

# Restriction Operators - Where

### [Where](https://docs.microsoft.com/zh-tw/dotnet/api/system.linq.enumerable.where?view=netframework-4.8#System_Linq_Enumerable_Where__1_System_Collections_Generic_IEnumerable___0__System_Func___0_System_Int32_System_Boolean__)
> Filters a sequence of values based on a predicate.

Where 在 LINQ 中就是篩選條件的方法 , 我們可以使用 Where **取得集合中符合描述的元素.**
其使用方式與 List.FindAll() 以及 static Array.FindAll() 類似 , 但差別在於 Where 的輸入參數以及回傳結果的型別皆為 IEnumerable , 而非 List 或是 Array . 因此因為 Where 具有延遲執行的優點 , 所以會建議使用 Where.

### 使用時機

- 從集合中取得所有符合特定條件的元素 (結果可能是複數).
- [示意圖](https://docs.microsoft.com/zh-tw/dotnet/csharp/programming-guide/concepts/linq/filtering-data)
  > ![](https://i.imgur.com/NWDT0nG.png)


### 多載型式

1. Where<TSource>(this IEnumerable<TSource> **source**, Func<TSource,Boolean> **predicate**)
    - Filters a sequence of values based on a predicate.
2. Where<TSource>(this IEnumerable<TSource> **source**, Func<TSource,**Int32**,Boolean> **predicate**)
    - Filters a sequence of values based on a predicate. Each element's index is used in the logic of the predicate function.

##### 解釋
1. this IEnumerable<TSource>
    - 代表此方法是擴充方法 , 也就是說 , IEnumerable<TSource> 型別的物件 , 都可以呼叫 Where() 方法. 例如 List<T> , T[] 等集合型別. 因為它們都有實現 IEnumerable<T> , 所以這些型別都可以直接呼叫 Where().
2. source 
    - 這個參數是我們要走訪的集合. 其泛型型別被命名為 TSource .
3. Func<TSource, bool> predicate
    - predicate 代表呼叫端希望過濾的條件. 因此這個委派會回傳 bool , 來判斷目前的 TSource 物件是否符合條件. ps : Func<TSource, bool> 其實與 Predicate<TSource> 無異. 
3. 回傳 IEnumerable<TSource>
     - 因為 Where() 是用來將符合條件的元素都篩選出來. 因此回傳型別與 source 型別會相同 , 都是 IEnumerable<TSource> . 然後因為回傳的是 IEnumerable<TSource> 型別 , 代表 Where 具有延遲執行的特性. 執行 Where , 並沒有馬上進行查詢.

### Where 的用處

#### 範例 - 從四個人中取出年紀大於六十歲的實體. 並印出它的資訊

```C#
public static IEnumerable<(string name, int age)> GetPeople()
{
     yield return (name: "德華", age: 87);
     yield return (name: "小王", age: 18);
     yield return (name: "老黃", age: 45);
     yield return (name: "阿高", age: 66);
}
```
##### 不使用 Where , 可能會這麼做
```C#
static void Main(string[] args)
{
     var people = GetPeople();
     foreach (var person in people)
     {
          if (person.age > 60)
          {
               Console.WriteLine(person.name + " " + person.age);
          }
     }
     Console.ReadKey();
}
```

##### 使用 Where
```C#
static void Main(string[] args)
{
     var people = GetPeople();
     var query = people.Where(person => person.age > 60);
     foreach (var person in query)
     {
           Console.WriteLine(person.name + " " + person.age);
     }

     Console.ReadKey();
}
```

輸出結果
![](https://i.imgur.com/FgZ45AP.png)

由上面的例子可以知道使用 Where 的優點如下
1. 寫法比較簡潔
2. 篩選條件是由**呼叫端透過委派決定** , 比較具有彈性
3. 能寫出較具有可讀性的程式 - 就 Where() 的寫法與在迴圈內加一個判斷式來比較的話

### 簡單實作自己的 Where
#### 沒有 index 的版本
```C#
public static IEnumerable<TSource> MyWhere<TSource>(this IEnumerable<TSource> source, Func<TSource, bool> predicate)
{
     foreach (var item in source)
     {
          if (predicate(item))
          {
               yield return item;
          }
     }
}
```
#### 有 index 的版本
```C#
public static IEnumerable<TSource> MyWhere<TSource>(this IEnumerable<TSource> source, Func<TSource, Int32, bool> predicate)
{
     int index = -1;
     foreach (var item in source)
     {
          checked
          {
               index++;
          }
          
          if (predicate(item, index))
          {
               yield return item;
          }
     }
}
```

### 補充範例
#### 使用條件式取代多個 Where -> 雖然結果相同但效能較好 & 較好讀
```C#
List<int> vs = new List<int> { 5, 9, 8, 7 };

var BadQuery = vs.Where(num => num > 5).Where(num => num < 9);

var GoodQuery = vs.Where(num => num > 5 && num < 9);
```

#### 結合Select , 找出 target 在 source 內的索引. 

```C#
static void Main(string[] args)
{
     List<string> source = new List<string> { "H", "Q", "T", "S" };
     List<string> target = new List<string> { "Q", "S" };
     var query = source.Select((num, index) => target.Contains(num) ? index : -1).Where(index => index != -1);
     foreach (var indexInSource in query)
     {
          Console.WriteLine(indexInSource);
     }

     Console.ReadKey();
}
```
輸出結果 
![](https://i.imgur.com/qNpJGP5.png)


### 參考
[Where的原碼探索](https://ithelp.ithome.com.tw/articles/10195658)
[Source Code](https://github.com/dotnet/corefx/blob/master/src/System.Linq/src/System/Linq/Where.cs)

---

### Thank you! 

You can find me on

- [GitHub](https://github.com/s0920832252)
- [Facebook](https://www.facebook.com/fourtune.chen)

若有謬誤 , 煩請告知 , 新手發帖請多包涵

# :100: :muscle: :tada: :sheep: 
