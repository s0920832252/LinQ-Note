---
tags: LinQ , C# , Projection Operators
---

# Projection Operators - Select

### [Select](https://docs.microsoft.com/en-us/dotnet/api/system.linq.enumerable.select?view=netframework-4.8)

> Projects each element of a sequence into a new form.

上述參考使用了Projects一詞 , 代表我們可以根據資料來源序列中的項目 , 建立相對應的輸出序列. 換句話說就是 , 將集合中的每一個元素以新的形式輸出. 我覺得使用以前高中教的函數的概念可能會更好理解. 亦即
```C# 
TResult y = selector(TSource x);
```
Select 會將 source 集合中的元素一一透過 selector() 這個委派去做轉換, 並在轉換後回傳結果.

### 使用時機
1. 調用一個物件與調用一個變數所消耗的成本是不同的. 假設我們只需要物件中的某幾項資訊 , 那麼我們其實可以僅取出我們需要的資訊來操作就好.
2. 我們需要對集合內全部元素做某項轉換後 , 再做操作.

### 多載形式
	
1. Projects each element of a sequence into a new form.
    - Select<TSource,TResult>(this IEnumerable<TSource>, **Func<TSource,TResult>**)
    
2. Projects each element of a sequence into a new form by incorporating the element's **index**.
    - Select<TSource,TResult>(this IEnumerable<TSource>, **Func<TSource,Int32,TResult>**)

##### 解釋
1. this IEnumerable<TSource>
    - LinQ 的方法基本上都是在走訪集合元素的時候 , 對集合元素做出操作. 所以會對 IEnumerable<TSource> 進行擴充. 也就是透過 GetEnumerator() 與 iterator.MoveNext(), iterator.Current 來巡覽.
2. Func<TSource,TResult>
    - 使用兩個泛型 , 分別代表投影前後的結果. 
    - Select會再走訪集合元素的時候 , 使用 Func 對元素做轉換.
3. Func<TSource,Int32,TResult>
    - 多了一個 index 這個參數可以使用. 其意義就是集合內元素的 index.
    - Select會再走訪集合元素的時候 , 使用 Func 對元素做轉換.

---

### Select 的用處
#### 範例 - 需要取出一群人的 BMI 數值 以及 Age

```C#
- 取出人群的函數
public static List<(string name, int age, int weight, int height)> GetPeople()
{
     return new List<(string name, int age, int weight, int height)>()
     {
          ("小王", 19, 75,172),
          ("小明", 29, 70,182),
          ("老黃", 69, 80,196),
     };
}
```

##### 不使用 Select 可能會這麼寫 !?
```C#
- 取出 BMI
public static IEnumerable<double> GetBMI()
{
     var people = GetPeople();
     foreach (var (name, age, weight, height) in people)
     {
          yield return weight / Math.Pow(height / 100.0, 2);
     }
}
```
```C#
- 取出 Age
public static IEnumerable<double> GetBMI()
{
     var people = GetPeople();
     foreach (var (name, age, weight, height) in people)
     {
          yield return age;
     }
}
```

##### 使用 Select
```C#
- 取出 BMI
public static IEnumerable<double> GetBMI()
{
     var people = GetPeople();
     return people.Select(person => person.weight / Math.Pow(person.height / 100.0, 2));
}
```
```C#
- 取出 Age
public static IEnumerable<double> GetBMI()
{
     var people = GetPeople();
     return people.Select(person => person.age);
}
```
我們可以透過使用 Select 很輕易的從集合中**僅**取出我們所需要的資訊.

由上面是否使用 Select 的例子可以知道 , Select 替我們走訪了集合成員 , 並且透過傳入的委派 , 去對集合成員做出相對應的處理. 因此我們不必自行實作 foreach 的部分 , 僅需要傳入對應的委派即可.

### 簡單實作自己的Select

作法 
1. MySelect : 負責對參數做出檢查或是判斷
2. MySelectIterator : 負責回傳 IEnumerable

將 Iterator 拆出到 MySelectIterator 去執行的原因是.
- yield return 會將函式標註為 iterator block , 也就是說會延遲執行. 但我們希望可以在設定查詢式的時候 , 立刻做出一些檢查或是判斷. e.g. 參數的檢查 , 集合Type 是 List or Array .

#### 沒有 index 的版本

```C#
public static IEnumerable<TResult> MySelect<TSource, TResult>(this IEnumerable<TSource> source, Func<TSource, TResult> selector)
{
     if (source is null || selector is null)
         throw new Exception("null exception");
     return MySelectIterator(source, selector);
}

private static IEnumerable<TResult> MySelectIterator<TSource, TResult>(IEnumerable<TSource> source, Func<TSource, TResult> selector)
{
     foreach (var item in source)
     {
          yield return selector(item);
     }
}
```
##### 測試程式
```C#
static void Main(string[] args)
{
     List<int> nums = new List<int> { 5, 9, 8, 7 };
     foreach (var num in nums.MySelect(num => num + 10))
     {
          Console.WriteLine(num);
     }
     Console.ReadKey();
}
```
##### 輸出結果
![](https://i.imgur.com/OMJQevR.png)

#### 有 index 的版本

```C#
public static IEnumerable<TResult> MySelect<TSource, TResult>(this IEnumerable<TSource> source, Func<TSource, Int32, TResult> selector)
{
     if (source is null || selector is null)
         throw new Exception("null exception");
     return MySelectIterator(source, selector);
}

private static IEnumerable<TResult> MySelectIterator<TSource, TResult>(IEnumerable<TSource> source, Func<TSource, Int32, TResult> selector)
{
     int index = -1;
     foreach (var item in source)
     {
          checked //C# 關鍵字 , 用來檢查 index 是否 overflow
          {
               index++;
          }
          yield return selector(item, index);
     }
}
```
##### 測試程式
```C#
static void Main(string[] args)
{
     List<int> nums = new List<int> { 5, 9, 8, 7 };
     var query = nums.MySelect((num, index) => num + index);
     foreach (var num in query)
     {
          Console.WriteLine(num);
     }
     Console.ReadKey();
}
```
##### 輸出結果
![](https://i.imgur.com/TuuB97D.png)

### 參考
[Select的原碼探險](https://ithelp.ithome.com.tw/articles/10194885)
[Source Code](https://github.com/dotnet/corefx/blob/master/src/System.Linq/src/System/Linq/Select.cs)

### 補充 - 使用匿名型別
#### 若是希望取出複數值?!
```C#
static void Main(string[] args)
{
     List<int> nums = new List<int> { 5, 9, 8, 7 };
     var query = nums.MySelect((num, index) => new { MyNum = num, MyIndex = index });
     foreach (var num in query)
     {
          Console.WriteLine($"{num.MyIndex} - {num.MyNum}");
     }
     Console.ReadKey();
}
```
##### 輸出結果
![](https://i.imgur.com/3vhJPx7.png)


---

### Thank you! 

You can find me on

- [GitHub](https://github.com/s0920832252)
- [Facebook](https://www.facebook.com/fourtune.chen)

若有謬誤 , 煩請告知 , 新手發帖請多包涵

# :100: :muscle: :tada: :sheep: 
