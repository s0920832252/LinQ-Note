---
tags: LinQ , C# , Ordering Operators 
---

# Reverse

### 前言
字面上意思 , 反轉序列中項目的排序方向. 因此會反過來輸出.

### [Reverse 的多載](https://docs.microsoft.com/zh-tw/dotnet/api/system.linq.enumerable.reverse?view=netframework-4.8)
```C#
public static IEnumerable<TSource> Reverse<TSource> (
            this IEnumerable<TSource> source
    );
```

### Reverse 的使用範例
```C#
static void Main(string[] args)
{
    IEnumerable<(string Name, int Age)> pets = new List<(string Name, int Age)>
    {
        ("小黑",8),
        ("小笨貓",4),
        ("喵喵",1),
    };

    var reverseQuery = pets.Reverse();
    foreach (var (name, age) in reverseQuery)
    {
        Console.WriteLine($"{name} {age}");
    }

    Console.ReadKey();
}
```
##### 輸出結果
喵喵 1     
小笨貓 4     
小黑 8     

### 簡單實作自己的 Reverse
```C#
public static IEnumerable<TSource> MyReverse<TSource>(
                this IEnumerable<TSource> sources
        ) => new Stack<TSource>(sources);
```

### Reverse 的 Source Code
```C#
public static IEnumerable<TSource> Reverse<TSource>(this IEnumerable<TSource> source)
{
    if (source is null)
    {
        throw new Exception("source is null");
    }

    return ReverseIterator<TSource>(source);
}

private static IEnumerable<TSource> ReverseIterator<TSource>(IEnumerable<TSource> source)
{
    var buffer = new Buffer<TSource>(source);
    for (var i = buffer.Count - 1; i >= 0; i--)
    {
        yield return buffer.Items[i];
    }
}

internal struct Buffer<TElement>
{
    internal TElement[] Items { get; }
    internal int Count { get; }

    internal Buffer(IEnumerable<TElement> source)
    {
        TElement[] items = null;
        var count = 0;
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
            foreach (var item in source)
            {
                if (items == null)
                {
                    items = new TElement[4];
                }
                else if (items.Length == count)
                {
                    var newItems = new TElement[checked(count * 2)];
                    Array.Copy(items, 0, newItems, 0, count);
                    items = newItems;
                }

                items[count] = item;
                count++;
            }
        }

        this.Items = items;
        this.Count = count;
    }
}
```




### Thank you! 

You can find me on

- [GitHub](https://github.com/s0920832252)
- [Facebook](https://www.facebook.com/fourtune.chen)

若有謬誤 , 煩請告知 , 新手發帖請多包涵

# :100: :muscle: :tada: :sheep: 
