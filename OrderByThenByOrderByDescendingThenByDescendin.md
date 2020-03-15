---
tags: LinQ , C# , Ordering Operators 
---

# OrderBy & ThenBy & OrderByDescending & ThenByDescending 使用介紹

### 前言
LinQ 與排序有關的方法一共有四個方法 , 分別是 OrderBy , ThenBy , OrderByDescending 以及 ThenByDescending. 透過 OrderBY 或是 OrderByDescending 創造實作  IOrderedEnumerable 類別 , 由此類別負責資料的排序. 之後的 ThenBy/ThenByDescending 則是修改此類別在排序上的方式(比較大小的方式). 是一個很漂亮的 Flutter 設計. 正常人大概會用建造者模式去實作 OrderBy 吧XD. 總之排序章節預計用兩篇的幅度來介紹 , 第一篇用來介紹如何使用 , 第二篇才來探討其方法實作的細節.

### [排序資料 (C#)](https://docs.microsoft.com/zh-tw/dotnet/csharp/programming-guide/concepts/linq/sorting-data)
> 排序會根據一個或多個屬性來排序序列的項目。 第一個排序方式會執行元素的主要排序； 您可以藉由指定第二個排序方式來繼續排序每一個主要排序群組內的元素.
> 下圖顯示對一系列字元執行字母順序排序作業的結果：
> ![](https://i.imgur.com/B3CbEQK.png)
> ```C#
> // 程式碼範例
> char[] Source = new char[] { 'G', 'C', 'F', 'E', 'B', 'A', 'D' };
> IOrderedEnumerable<char> Results = Source.OrderBy(c => c);
> 
> foreach(char result in Results){
>     Console.Write($"{result} ");
> }
> Console.WriteLine();
> // output: A B C D E F G
> ```
由以上的例子為遞增排序 , 但排序還是可能有以下的狀況
1. 可以遞減排序嗎？
    - A : 使用 OrderByDescending / ThenByDescending
2. 可以指定多個排序欄位嗎？
    - A : 再使用 Order / OrderByDescending 後接著使用 ThenBy / ThenByDescending
3. 可以同時使用多個欄位進行遞增和遞減排序嗎？
    - A : 可以 , 交錯使用 LinQ 排序的方法即可.
4. 可以自訂排序邏輯嗎？
    - A : 傳入實作 ICompare 的比較器 


#### 以下為 LinQ 排序資料的方法
1. OrderBy
    - 設定初始排序方式為**遞增排序**.
2. OrderByDescending
    - 設定初始排序方式為**遞減排序**.
3. ThenBy
    - 在維持原本的排序方式下**繼續設定第二個或以後的排序方式為遞增排序**
4. ThenByDescending
    - 在維持原本的排序方式下**繼續設定第二個或以後的排序方式為遞減排序**

觀察以上四個方法 , 其實可以發現 , 它們其實只有遞增或是遞減兩種方式而已 , 其本質都是相同的.

### [OrderBy](https://docs.microsoft.com/zh-tw/dotnet/api/system.linq.enumerable.orderby?view=netframework-4.8)
#### OrderBy 的多載
1. ```C#
    public static IOrderedEnumerable<TSource> OrderBy<TSource,TKey> (
                this IEnumerable<TSource> source, 
                Func<TSource,TKey> keySelector
        );
    ```
2. ```C#
    public static IOrderedEnumerable<TSource> OrderBy<TSource,TKey> (
                this IEnumerable<TSource> source, 
                Func<TSource,TKey> keySelector, 
                IComparer<TKey> comparer
        );
    ```
#### 名詞解釋 
- keySelector : 指定哪一個欄位需要排序
- comparer : 自定義比較器
- IOrderedEnumerable : LINQ 與排序有關的四個方法都會回傳此型別

OrderBy有兩個方法 , 其差別在於要不要傳入自定義比較器. 其回傳值為 IOrderedEnumerable , 此型別繼承 IEnumerable. 該型別的責任為排序資料 , 因此之後若使用 ThenBy 或是 ThenByDescending 再增加排序條件 , IOrderedEnumerable 也有這個能力可以處理.

因為 **IOrderedEnumerable 繼承 IEnumerable** , 所以 OrderBy 方法可以再接 OrderBy , 甚至 ThenBy 後也可以再接 OrderBy 但再最後一個 OrderBy 之前所設定的排序方法皆會被忽略. 初始排序會以最後一個 OrderBy 設定的排序方法為初始排序方法.

#### OrderBy 的使用範例
```C#
static void Main(string[] args)
{
    var pets = new List<(string Name, int Age)>()
    {
        ("小黑",8),
        ("小笨貓",4),
        ("喵喵",1),
    };

    // 延遲執行 - 指定年齡遞增排序
    var orderQuery = pets.OrderBy(pet => pet.Age);

    foreach (var (name, age) in orderQuery)
    {
        Console.WriteLine($"寵物 {name} is {age} 歲");
    }

    Console.ReadKey();
}
```
##### 輸出結果
寵物 喵喵 is 1 歲     
寵物 小笨貓 is 4 歲     
寵物 小黑 is 8 歲     

### [OrderByDescending](https://docs.microsoft.com/zh-tw/dotnet/api/system.linq.enumerable.orderbydescending?view=netframework-4.8)
#### OrderByDescending 的多載
1. ```C#
        public static IOrderedEnumerable<TSource> OrderByDescending<TSource,TKey> (
                his IEnumerable<TSource> source,
                Func<TSource,TKey> keySelector
        );
    ```
2. ```C#
    public static IOrderedEnumerable<TSource> OrderByDescending<TSource,TKey> (
                this Enumerable<TSource> source, 
                Func<TSource,TKey> keySelector, 
                IComparer<TKey> comparer
            );
    ```
OrderByDescending 也有兩個多載方法 , 差別再是否有自定義比較器.
大致上與 OrderBy 差不多. 但其排序方式與 OrderBy 相反.
是遞減排序.

#### OrderByDescending 的使用範例
```C#
static void Main(string[] args)
{
    var pets = new List<(string Name, int Age)>()
    {
        ("喵喵",1),
        ("小笨貓",4),
        ("小黑",8),
    };

    // 延遲執行 - 使用年紀遞減排序
    var orderQuery = pets.OrderByDescending(pet => pet.Age);

    foreach (var (name, age) in orderQuery)
    {
        Console.WriteLine($"寵物 {name} is {age} 歲");
    }

    Console.ReadKey();
}
```
##### 輸出結果
寵物 小黑 is 8 歲     
寵物 小笨貓 is 4 歲     
寵物 喵喵 is 1 歲     

### [ThenBy](https://docs.microsoft.com/zh-tw/dotnet/api/system.linq.enumerable.thenby?view=netframework-4.8)

#### ThenBy 的多載
1. ```C#
    public static IOrderedEnumerable<TSource> ThenBy<TSource,TKey> (
                this IOrderedEnumerable<TSource> source, 
                Func<TSource,TKey> keySelector
        )
    ```
2. ```C#
    public static IOrderedEnumerable<TSource> ThenBy<TSource,TKey> (
                this IOrderedEnumerable<TSource> source, 
                Func<TSource,TKey> keySelector,
                IComparer<TKey> comparer
        )
    ```

ThenBy 是用來設定第二個或者第 N 個 排序方式. 也就是說是再設定初始排序方式之後. 所以 ThenBy 一定是接再 OrderBy 之後.
再介紹 OrderBy 時 , 有介紹到其回傳值型態為 IOrderedEnumerable. 而 **ThenBy 的輸入參數型態為 IOrderedEnumerable , 回傳值型態也是 IOrderedEnumerable**
1. 這代表**你在一個方法回傳值型態為 IEnumerable 後面是不能接 ThenBy 的 , 要先接OrderBy才能用ThenBy.**
2. **ThenBy 後面可以繼續接 ThenBy 系列** , 因為其回傳值與輸入參數型態相同

ThenBy 的排序方式為**遞增排序**

#### ThenBy 的使用範例
```C#
static void Main(string[] args)
{
    var pets = new List<(string Name, int Age)>()
    {
        ("喵喵2",1),
        ("喵喵",1),
        ("小笨貓",4),
        ("小笨貓2",4),        
        ("小黑2",8),
        ("小黑",8),
    };

    // 延遲執行 - 先用年紀遞增排序 , 再此排序下若有相同值 , 再以性別遞增排序
    var orderQuery = pets.OrderBy(pet => pet.Age).ThenBy(pet => pet.Name);

    foreach (var (name, age) in orderQuery)
    {
        Console.WriteLine($"寵物 {name} is {age} 歲");
    }

    Console.ReadKey();
}
```
##### 輸出結果
寵物 喵喵 is 1 歲     
寵物 喵喵2 is 1 歲     
寵物 小笨貓 is 4 歲     
寵物 小笨貓2 is 4 歲     
寵物 小黑 is 8 歲     
寵物 小黑2 is 8 歲     

### [ThenByDescending](https://docs.microsoft.com/zh-tw/dotnet/api/system.linq.enumerable.thenbydescending?view=netframework-4.8)

#### ThenByDescending 的多載
1. ```C#
    public static IOrderedEnumerable<TSource> ThenByDescending<TSource,TKey> (
                this IOrderedEnumerable<TSource> source, 
                Func<TSource,TKey> keySelector
        );
    ```
2. ```C#
    public static IOrderedEnumerable<TSource> ThenByDescending<TSource,TKey> (
                this IOrderedEnumerable<TSource> source, 
                Func<TSource,TKey> keySelector, 
                IComparer<TKey> comparer
        );
    ```

基本上同 ThenBy , 只是其排序方式為**遞減排序**.

#### ThenByDescending 的使用範例
```C#
static void Main(string[] args)
{
    var pets = new List<(string Name, int Age)>()
    {
        ("喵喵2",1),
        ("喵喵",1),
        ("小笨貓",4),
        ("小笨貓2",4),
        ("小黑2",8),
        ("小黑",8),
    };

    // 延遲執行 - 先用年紀遞增排序 , 再此排序下若有相同值 , 再以性別'遞減'排序
    var orderQuery = pets.OrderBy(pet => pet.Age).ThenByDescending(pet => pet.Name);

    foreach (var (name, age) in orderQuery)
    {
        Console.WriteLine($"寵物 {name} is {age} 歲");
    }

    Console.ReadKey();
}
```
##### 輸出結果
寵物 喵喵2 is 1 歲     
寵物 喵喵 is 1 歲     
寵物 小笨貓2 is 4 歲     
寵物 小笨貓 is 4 歲     
寵物 小黑2 is 8 歲     
寵物 小黑 is 8 歲     

### Summary 
1. 屬於延遲執行的方法，回傳的只是走訪器物件，要等到執行 GetEnumerator() 或是 foreach 才會觸發走訪.
2. OrderBy / OrderByDescending 回傳值不是 IEnumerable 而是 IOrderedEnumerable , 其目的是要為了讓 ThenBy 及 ThenByDescending 可以接續其原本的排序方式繼續設定其他的排序方式
3. 可以實作 IComparer 來自定義比較器
4. OrderBy / OrderByDescending 會排在第一個 , 用來設定初始排序方法 , 而 ThenBy / ThenByDescending 則是排在第 2 ~  n 個 , 用來設定第 2 ~  n 個的排序方法
5. 有 Descending 後墜字的方法是用來設定遞減排序 , 反之則是用來設定遞增排序.


### Thank you! 

You can find me on

- [GitHub](https://github.com/s0920832252)
- [Facebook](https://www.facebook.com/fourtune.chen)

若有謬誤 , 煩請告知 , 新手發帖請多包涵

# :100: :muscle: :tada: :sheep: 
