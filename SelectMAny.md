---
tags: LinQ , C# , Projection Operators
---

# Projection Operators - SelectMany

SelectMany 的功用與 Select 相同 , 都是依照我們設定的委派去對資料源做出處理 , 並輸出. 但是差異是 Select 只取得第一層的查詢結果放到輸出序列中 , 但是 SelectMany 若發現查詢結果回傳的是一個可以列舉的序列 , 則會再進一步把這個序列中的項目取出來 , 放到輸出序列中 , 也就是用 SelectMany 運算子 , 查詢出來的輸出序列之深度會比 Select 少一層.
SelectMany 通常是在處理巢狀集合的時候使用 , 可以替我們省去多層迴圈 .

一言以蔽之 , SelectMany 可以將集合中每個元素內的子集合**合併為一個新的集合**. 換句話說 , SelectMany 可以幫我們**把子集合展開成一個.**

### 使用時機
- 需要取出集合元素中的每一個子集合中的每一個元素.
  e.g. 學生物件有一個List<朋友> . 我們想要取出List<學生> 中所有的朋友物件.
- 需要對集合元素中的每一個子集合中的每一個元素做投影的動作.

你喜歡跑雙層迴圈取值 , 那這個函數可以當作沒看到 :laughing: 

### [多載形式](https://docs.microsoft.com/zh-tw/dotnet/api/system.linq.enumerable.selectmany?view=netframework-4.8)

1. SelectMany<TSource,TResult>(this IEnumerable<TSource>, Func<TSource,**IEnumerable<TResult>>**)
    * 將序列的每個項目都投影成 IEnumerable<T>，並將產生的序列簡化成單一序列。

2. SelectMany<TSource,TResult>(this IEnumerable<TSource>, Func<TSource,Int32,**IEnumerable<TResult>>**)
    * 將序列的每個項目都投影成 IEnumerable<T>，並將產生的序列簡化成單一序列。 各來源項目的索引是在該項目的投影表單中使用。

3. SelectMany<TSource,TCollection,TResult>(this IEnumerable<TSource>, Func<TSource,IEnumerable<TCollection>>, Func<TSource,TCollection,TResult>)
    * 將序列的每個項目投影為 IEnumerable<T>、將產生的序列簡化成單一序列，並對其中的每個項目叫用結果選取器函式。

4. SelectMany<TSource,TCollection,TResult>(this IEnumerable<TSource>, Func<TSource,Int32,IEnumerable<TCollection>>, Func<TSource,TCollection,TResult>)
    * 將序列的每個項目投影為 IEnumerable<T>、將產生的序列簡化成單一序列，並對其中的每個項目叫用結果選取器函式。 各來源項目的索引是在該項目的中繼投影表單中使用。

##### 解釋
- 3 & 4 與 1 & 2 相比
    - 多了一個 resultSelector 可以使用 , 它可以傳入TSource (原本集合的每一個元素)及 TCollection (子集合中的每一個元素) , 讓我們可以讓外層的資料跟內層的資料合為同一筆資料. 當然 , 因為有 TCollection 可以使用 , 自然也能再對 TCollection 做投影的處理.
- 使用 1 以及 2 將需要展開的集合平展開成一個集合
- **index 是 source 的索引.**
- Func **回傳 IEnumerable 型別**. 這代表你需要指定一個能走訪的集合回傳.

### 範例
有三個學生{小昂,小王,大名}
- 小昂有一個朋友 , 兩個手機.
- 小王有一個朋友 , 一個手機.
- 大名有兩個朋友 , 一個手機.

試不使用雙層迴圈來取出他們的資訊並列出
1. 僅取出號碼. 
2. 僅取出號碼 , 但同個人的號碼 , 需要用相同的編號.
3. 僅取出朋友姓名 , 但必須標註這是哪個學生的朋友.
4. 取出朋友的資訊 , 但必須與學生的資訊配對再一起. (湊數 , 我想不到好例子= =)

#### 結構
```C#
public class Student
{
     public string Name { get; set; }
     public List<Friend> Friends { get; set; }
     public List<string> Phones { get; set; }
}

public class Friend
{
     public string Name { get; set; }
}

public static List<Student> GetStudents()
{
     return new List<Student>
     {
          new Student{Name = "小昂", Friends=new List<Friend>{ new Friend {Name="V" } } ,Phones=new List<string>{ "0989782268","0920755420" }},
          new Student{Name = "小王", Friends=new List<Friend>{ new Friend {Name="Q" } } ,Phones=new List<string>{ "1234567888" }},
          new Student{Name = "大名", Friends=new List<Friend>{ new Friend {Name="T" }, new Friend { Name = "S" } } ,Phones=new List<string>{ "6666666666" }},
     };
}
```
#### 測試程式
```C#
 static void Main(string[] args)
{
     var students = GetStudents();
     var queryPhones = students.SelectMany((student) => student.Phones);
     foreach (var phone in queryPhones)
     {
          Console.WriteLine(phone);
     }

     Console.WriteLine();

     var queryPhonesWithIndex = students.SelectMany((student, index) => student.Phones.Select(phone => $"編號 {index}. 的 {phone}")); ;
     foreach (var phone in queryPhonesWithIndex)
     {
          Console.WriteLine(phone);
     }

     Console.WriteLine();

     var queryFriends = students.SelectMany(student => student.Friends, (student, friend) => $"{student.Name} 有朋友叫做 {friend.Name}");
     foreach (var item in queryFriends)
     {
          Console.WriteLine(item);
     }

     Console.WriteLine();

     var queryFriendsWithIndex = students.SelectMany((student, index) => student.Friends.Select(friend => new { ID = index, friend.Name }), (student, friend) => new { student, friend });
     foreach (var item in queryFriendsWithIndex)
     {
          Console.WriteLine($"編號{item.friend.ID}號 的{item.student.Name} 有朋友叫做 {item.friend.Name}");
     }
     Console.ReadKey();
}
```
#### 結果
![6LsbQDz.png](https://github.com/s0920832252/LinQ-Note/blob/master/Resources/6LsbQDz.png?raw=true)

---

### 簡單實作自己的SelectMany

作法
- MySelectMany : 負責對參數做出檢查或是判斷
- MySelectManyIterator : 負責回傳 IEnumerable
    - 其實就是走訪指定的 Source
- resultSelector : 將 subItem 轉換成結果.
```C#
public static IEnumerable<TResult> MySelectMany<TSource, TResult>(this IEnumerable<TSource> source, Func<TSource, int, IEnumerable<TResult>> selector)
{
     if (source is null || selector is null)
         throw new Exception("Exception");
     return MySelectManyIterator(source, selector);
}

public static IEnumerable<TResult> MySelectManyIterator<TSource, TResult>(IEnumerable<TSource> source, Func<TSource, int, IEnumerable<TResult>> selector)
{
     int index = -1;
     foreach (var item in source)
     {
          checked
          {
               index++;
          }
          foreach (var subItem in selector(item, index))
          {
               yield return subItem;
          }
     }
}

public static IEnumerable<TResult> MySelectMany<TSource, TCollection, TResult>(this IEnumerable<TSource> source, Func<TSource, int, IEnumerable<TCollection>> collectionSelector, Func<TSource, TCollection, TResult> resultSelector)
{
     if (source is null || collectionSelector is null || resultSelector is null)
          throw new Exception("Exception");
     return MySelectManyIterator(source, collectionSelector, resultSelector);
}

private static IEnumerable<TResult> MySelectManyIterator<TSource, TCollection, TResult>(IEnumerable<TSource> source, Func<TSource, int, IEnumerable<TCollection>> collectionSelector, Func<TSource, TCollection, TResult> resultSelector)
{
     int index = -1;
     foreach (var item in source)
     {
          checked
          {
               index++;
          }
          foreach (var subItem in collectionSelector(item, index))
          {
               yield return resultSelector(item, subItem);
          }
     }
}
```


---

參考
[Source Code](https://github.com/dotnet/corefx/blob/master/src/System.Linq/src/System/Linq/SelectMany.cs)

---

### Thank you! 

You can find me on

- [GitHub](https://github.com/s0920832252)
- [Facebook](https://www.facebook.com/fourtune.chen)

若有謬誤 , 煩請告知 , 新手發帖請多包涵

# :100: :muscle: :tada: :sheep: 
