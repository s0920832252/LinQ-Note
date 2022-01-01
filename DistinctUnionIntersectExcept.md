---
tags: LinQ , C# , Set Operators
---

# Distinct & Union & Intersect & Except

### 前言
在 LinQ 中 , 關於集合的運算共有四個方法 , 每一個方法都基於不同的概念而有不同的結果.
1. Distinct - 轉成集合
2. Union - 轉成聯集
3. Intersect - 轉成交集
4. Except - 轉成差集

### [Distinct](https://docs.microsoft.com/zh-tw/dotnet/api/system.linq.enumerable.distinct?view=netframework-4.8)
Distinct 方法會回傳不包含重複值的資料集合. 換句話說 , Distinct 方法會移除資料集合中全部的重複值 , 並且將這個僅包含唯一值的資料集合回傳.

![gwlxjsW.png](https://github.com/s0920832252/LinQ-Note/blob/master/Resources/gwlxjsW.png?raw=true)


#### Distinct 的多載
```C#
public static IEnumerable<TSource> Distinct<TSource> (
                this IEnumerable<TSource> source
        );
```
```C#
public static IEnumerable<TSource> Distinct<TSource> (
                this IEnumerable<TSource> source, 
                IEqualityComparer<TSource> comparer
        );
```

#### Distinct 的範例
```C#
static void Main(string[] args)
{
    var duplicateNuns = new List<int>() { 1, 1, 2, 3, 1, 5, 3, 5 };
    IEnumerable<int> distinctQuery = duplicateNuns.Distinct();
    foreach (var num in distinctQuery)
    {
        Console.WriteLine(num);
    }
    Console.ReadKey();
}
```
##### 輸出結果
1     
2     
3     
5     


```C#
class EqualityComparer<TSource> : IEqualityComparer<TSource> where TSource : class
{
    private PropertyInfo[] _properties = null;
    public bool Equals(TSource x, TSource y)
    {
        if (_properties == null)
            _properties = x.GetType().GetProperties();
        return _properties.Aggregate(true, (result, item) => result && item.GetValue(x).Equals(item.GetValue(y)));
    }

    public int GetHashCode(TSource obj)
    {
        if (_properties == null)
            _properties = obj.GetType().GetProperties();
        return _properties.Aggregate(0, (result, item) => result ^ item.GetValue(obj).GetHashCode());
    }
}

public class Product
{
    public string Name { get; set; }
    public int Price { get; set; }
}

static void Main(string[] args)
{
    Product[] products =
    {
        new Product { Name = "apple", Price = 9 },
        new Product { Name = "orange", Price = 4 },
        new Product { Name = "apple", Price = 9 },
        new Product { Name = "lemon", Price = 12 },
        new Product { Name = "orange", Price = 4 },
    };

    var distinctProduct = products.Distinct(new EqualityComparer<Product>());

    foreach (var product in distinctProduct)
    {
        Console.WriteLine(product.Name + " " + product.Price);
    }

    Console.ReadKey();
}
```
##### 輸出結果
apple 9     
orange 4     
lemon 12     
#### 簡單實作自己的 Distinct
```C#
public static IEnumerable<TSource> Distinct<TSource>(this IEnumerable<TSource> source)
{
    if (source is null)
    {
        throw new Exception("source is null");
    }
    return DistinctIterator<TSource>(source, null);
}

public static IEnumerable<TSource> Distinct<TSource>(this IEnumerable<TSource> source, IEqualityComparer<TSource> comparer)
{
    if (source is null)
    {
        throw new Exception("source is null");
    }
    return DistinctIterator<TSource>(source, comparer);
}

private static IEnumerable<TSource> DistinctIterator<TSource>(IEnumerable<TSource> source, IEqualityComparer<TSource> comparer)
{
    // comparer 為 null , 會用 EqualityComparer<T>.Default;
    return new HashSet<TSource>(source, comparer); 
}
```
#### Distinct 的 Source Code
```C#
public static IEnumerable<TSource> Distinct<TSource>(this IEnumerable<TSource> source)
{
    if (source is null)
    {
        throw new Exception("source is null");
    }
    return DistinctIterator<TSource>(source, null);
}

public static IEnumerable<TSource> Distinct<TSource>(this IEnumerable<TSource> source, IEqualityComparer<TSource> comparer)
{
    if (source is null)
    {
        throw new Exception("source is null");
    }
    return DistinctIterator<TSource>(source, comparer);
}

private static IEnumerable<TSource> DistinctIterator<TSource>(IEnumerable<TSource> source, IEqualityComparer<TSource> comparer)
{
    var set = new Set<TSource>(comparer);
    foreach (var element in source)
    {
        if (set.Add(element))
        {
            yield return element;
        }
    }
}

internal class Set<TElement> 
{
    int[] buckets;
    Slot[] slots;
    int count;
    int freeList;
    readonly IEqualityComparer<TElement> comparer;

    public Set() : this(null) { }

    public Set(IEqualityComparer<TElement> comparer)
    {
        if (comparer == null) comparer = EqualityComparer<TElement>.Default;
        this.comparer = comparer;
        buckets = new int[7];
        slots = new Slot[7];
        freeList = -1;
    }

    // If value is not in set, add it and return true; otherwise return false
    public bool Add(TElement value)
    {
        return !Find(value, true);
    }

    // Check whether value is in set
    public bool Contains(TElement value)
    {
        return Find(value, false);
    }

    // If value is in set, remove it and return true; otherwise return false
    public bool Remove(TElement value)
    {
        int hashCode = InternalGetHashCode(value);
        int bucket = hashCode % buckets.Length;
        int last = -1;
        for (int i = buckets[bucket] - 1; i >= 0; last = i, i = slots[i].next)
        {
            if (slots[i].hashCode == hashCode && comparer.Equals(slots[i].value, value))
            {
                if (last < 0)
                {
                    buckets[bucket] = slots[i].next + 1;
                }
                else
                {
                    slots[last].next = slots[i].next;
                }
                slots[i].hashCode = -1;
                slots[i].value = default(TElement);
                slots[i].next = freeList;
                freeList = i;
                return true;
            }
        }
        return false;
    }

    bool Find(TElement value, bool add)
    {
        int hashCode = InternalGetHashCode(value);
        for (int i = buckets[hashCode % buckets.Length] - 1; i >= 0; i = slots[i].next)
        {
            if (slots[i].hashCode == hashCode && comparer.Equals(slots[i].value, value)) return true;
        }
        if (add)
        {
            int index;
            if (freeList >= 0)
            {
                index = freeList;
                freeList = slots[index].next;
            }
            else
            {
                if (count == slots.Length) Resize();
                index = count;
                count++;
            }
            int bucket = hashCode % buckets.Length;
            slots[index].hashCode = hashCode;
            slots[index].value = value;
            slots[index].next = buckets[bucket] - 1;
            buckets[bucket] = index + 1;
        }
        return false;
    }

    void Resize()
    {
        int newSize = checked(count * 2 + 1);
        int[] newBuckets = new int[newSize];
        Slot[] newSlots = new Slot[newSize];
        Array.Copy(slots, 0, newSlots, 0, count);
        for (int i = 0; i < count; i++)
        {
            int bucket = newSlots[i].hashCode % newSize;
            newSlots[i].next = newBuckets[bucket] - 1;
            newBuckets[bucket] = i + 1;
        }
        buckets = newBuckets;
        slots = newSlots;
    }

    internal int InternalGetHashCode(TElement value)
    {
        //Microsoft DevDivBugs 171937. work around comparer implementations that throw when passed null
        return (value == null) ? 0 : comparer.GetHashCode(value) & 0x7FFFFFFF;
    }

    internal struct Slot
    {
        internal int hashCode;
        internal TElement value;
        internal int next;
    }
}
```
### [Union](https://docs.microsoft.com/zh-tw/dotnet/api/system.linq.enumerable.union?view=netframework-4.8)
Union 方法會回傳聯集集合. 換句話說 , 它會合併兩個不同的資料集合為一個 , 並且此合併後的資料集合沒有任何的重複值.
![FVUvVjt.png](https://github.com/s0920832252/LinQ-Note/blob/master/Resources/FVUvVjt.png?raw=true)

#### Union 的多載
```C#
public static IEnumerable<TSource> Union<TSource> (
            this IEnumerable<TSource> first, 
            IEnumerable<TSource> second
    );
```
```C#
public static IEnumerable<TSource> Union<TSource> (
            this IEnumerable<TSource> first, 
            IEnumerable<TSource> second, 
            IEqualityComparer<TSource> comparer
    );
```

#### Union 的範例
```C#
static void Main(string[] args)
{
    var nums = new List<int> { 1, 1, 2, };
    var nums2 = new List<int> { 3, 1, 5, 3, 5 };
    IEnumerable<int> unionQuery = nums.Union(nums2);
    foreach (var num in unionQuery)
    {
        Console.WriteLine(num);
    }
    Console.ReadKey();
}
```
##### 輸出結果
1     
2     
3     
5     

```C#
class EqualityComparer<TSource> : IEqualityComparer<TSource> where TSource : class
{
    private PropertyInfo[] _properties = null;
    public bool Equals(TSource x, TSource y)
    {
        if (_properties == null)
            _properties = x.GetType().GetProperties();
        return _properties.Aggregate(true, (result, item) => result && item.GetValue(x).Equals(item.GetValue(y)));
    }

    public int GetHashCode(TSource obj)
    {
        if (_properties == null)
            _properties = obj.GetType().GetProperties();
        return _properties.Aggregate(0, (result, item) => result ^ item.GetValue(obj).GetHashCode());
    }
}

public class Product
{
    public string Name { get; set; }
    public int Price { get; set; }
}

static void Main(string[] args)
{
    Product[] products =
    {
        new Product { Name = "apple", Price = 9 },
        new Product { Name = "orange", Price = 4 },
        new Product { Name = "apple", Price = 9 },
    };
    Product[] products2 =
    {
        new Product { Name = "lemon", Price = 12 },
        new Product { Name = "orange", Price = 4 },
    };

    var unionProduct = products.Union(products2, new EqualityComparer<Product>());
    foreach (var product in unionProduct)
    {
        Console.WriteLine(product.Name + " " + product.Price);
    }
    Console.ReadKey();
}
```
##### 輸出結果
apple 9     
orange 4     
lemon 12     

#### 簡單實作自己的 Union
```C#
public static IEnumerable<TSource> MyUnion<TSource>(this IEnumerable<TSource> first, IEnumerable<TSource> second)
{
    if (first is null || second is null)
    {
        throw new Exception("null exception");
    }
    return MyUnion(first, second, null);
}

public static IEnumerable<TSource> MyUnion<TSource>(this IEnumerable<TSource> first, IEnumerable<TSource> second, IEqualityComparer<TSource> comparer)
{
    if (first is null || second is null)
    {
        throw new Exception("null exception");
    }
    var set = new HashSet<TSource>(comparer);
    foreach (var item in first)
    {
        if (set.Add(item))
        {
            yield return item;
        }
    }

    foreach (var item in second)
    {
        if (set.Add(item))
        {
            yield return item;
        }
    }
}
```
#### Union 的 Source Code
```C#
public static IEnumerable<TSource> Union<TSource>(this IEnumerable<TSource> first, IEnumerable<TSource> second) {
    if (first == null) throw Error.ArgumentNull("first");
    if (second == null) throw Error.ArgumentNull("second");
    return UnionIterator<TSource>(first, second, null);
}

public static IEnumerable<TSource> Union<TSource>(this IEnumerable<TSource> first, IEnumerable<TSource> second, IEqualityComparer<TSource> comparer)
{
    if (first == null) throw Error.ArgumentNull("first");
    if (second == null) throw Error.ArgumentNull("second");
    return UnionIterator<TSource>(first, second, comparer);
}

static IEnumerable<TSource> UnionIterator<TSource>(IEnumerable<TSource> first, IEnumerable<TSource> second, IEqualityComparer<TSource> comparer)
{
    Set<TSource> set = new Set<TSource>(comparer);
    foreach (TSource element in first)
        if (set.Add(element)) yield return element;
    foreach (TSource element in second)
        if (set.Add(element)) yield return element;
}

internal class Set<TElement> 
{
    int[] buckets;
    Slot[] slots;
    int count;
    int freeList;
    readonly IEqualityComparer<TElement> comparer;

    public Set() : this(null) { }

    public Set(IEqualityComparer<TElement> comparer)
    {
        if (comparer == null) comparer = EqualityComparer<TElement>.Default;
        this.comparer = comparer;
        buckets = new int[7];
        slots = new Slot[7];
        freeList = -1;
    }

    // If value is not in set, add it and return true; otherwise return false
    public bool Add(TElement value)
    {
        return !Find(value, true);
    }

    // Check whether value is in set
    public bool Contains(TElement value)
    {
        return Find(value, false);
    }

    // If value is in set, remove it and return true; otherwise return false
    public bool Remove(TElement value)
    {
        int hashCode = InternalGetHashCode(value);
        int bucket = hashCode % buckets.Length;
        int last = -1;
        for (int i = buckets[bucket] - 1; i >= 0; last = i, i = slots[i].next)
        {
            if (slots[i].hashCode == hashCode && comparer.Equals(slots[i].value, value))
            {
                if (last < 0)
                {
                    buckets[bucket] = slots[i].next + 1;
                }
                else
                {
                    slots[last].next = slots[i].next;
                }
                slots[i].hashCode = -1;
                slots[i].value = default(TElement);
                slots[i].next = freeList;
                freeList = i;
                return true;
            }
        }
        return false;
    }

    bool Find(TElement value, bool add)
    {
        int hashCode = InternalGetHashCode(value);
        for (int i = buckets[hashCode % buckets.Length] - 1; i >= 0; i = slots[i].next)
        {
            if (slots[i].hashCode == hashCode && comparer.Equals(slots[i].value, value)) return true;
        }
        if (add)
        {
            int index;
            if (freeList >= 0)
            {
                index = freeList;
                freeList = slots[index].next;
            }
            else
            {
                if (count == slots.Length) Resize();
                index = count;
                count++;
            }
            int bucket = hashCode % buckets.Length;
            slots[index].hashCode = hashCode;
            slots[index].value = value;
            slots[index].next = buckets[bucket] - 1;
            buckets[bucket] = index + 1;
        }
        return false;
    }

    void Resize()
    {
        int newSize = checked(count * 2 + 1);
        int[] newBuckets = new int[newSize];
        Slot[] newSlots = new Slot[newSize];
        Array.Copy(slots, 0, newSlots, 0, count);
        for (int i = 0; i < count; i++)
        {
            int bucket = newSlots[i].hashCode % newSize;
            newSlots[i].next = newBuckets[bucket] - 1;
            newBuckets[bucket] = i + 1;
        }
        buckets = newBuckets;
        slots = newSlots;
    }

    internal int InternalGetHashCode(TElement value)
    {
        //Microsoft DevDivBugs 171937. work around comparer implementations that throw when passed null
        return (value == null) ? 0 : comparer.GetHashCode(value) & 0x7FFFFFFF;
    }

    internal struct Slot
    {
        internal int hashCode;
        internal TElement value;
        internal int next;
    }
}
```
### [Intersect](https://docs.microsoft.com/zh-tw/dotnet/api/system.linq.enumerable.intersect?view=netframework-4.8)
Intersect 方法回傳交集. 換句話說 , 回傳的資料集合元素必定可在兩個資料集合中各找到一份找到相同的元素.
![NiBWfO4.png](https://github.com/s0920832252/LinQ-Note/blob/master/Resources/NiBWfO4.png?raw=true)


#### Intersect 的多載
```C#
public static IEnumerable<TSource> Intersect<TSource> (
            this IEnumerable<TSource> first,
            IEnumerable<TSource> second
    );
```
```C#
public static IEnumerable<TSource> Intersect<TSource> (
            this IEnumerable<TSource> first,
            IEnumerable<TSource> second, 
            IEqualityComparer<TSource> comparer
    );
```
#### Intersect 的範例
```
static void Main(string[] args)
{
    var nums = new List<int> { 1, 1, 2, };
    var nums2 = new List<int> { 3, 1, 5, 3, 5 };
    IEnumerable<int> intersectQuery = nums.Intersect(nums2);
    foreach (var num in intersectQuery)
    {
        Console.WriteLine(num);
    }
    Console.ReadKey();

    Console.ReadKey();
}
```
##### 輸出結果
1        

```C#
class EqualityComparer<TSource> : IEqualityComparer<TSource> where TSource : class
{
    private PropertyInfo[] _properties = null;
    public bool Equals(TSource x, TSource y)
    {
        if (_properties == null)
            _properties = x.GetType().GetProperties();
        return _properties.Aggregate(true, (result, item) => result && item.GetValue(x).Equals(item.GetValue(y)));
    }

    public int GetHashCode(TSource obj)
    {
        if (_properties == null)
            _properties = obj.GetType().GetProperties();
        return _properties.Aggregate(0, (result, item) => result ^ item.GetValue(obj).GetHashCode());
    }
}

public class Product
{
    public string Name { get; set; }
    public int Price { get; set; }
}

static void Main(string[] args)
{
    Product[] products =
    {
        new Product { Name = "apple", Price = 9 },
        new Product { Name = "orange", Price = 4 },
        new Product { Name = "apple", Price = 9 },
        new Product { Name = "lemon", Price = 12 },
        new Product { Name = "orange", Price = 4 },
    };
    Product[] products2 =
    {
        new Product { Name = "lemon", Price = 12 },
        new Product { Name = "orange", Price = 4 },
    };

    var intersectProduct = products.Intersect(products2, new EqualityComparer<Product>());

    foreach (var product in intersectProduct)
    {
        Console.WriteLine(product.Name + " " + product.Price);
    }
    Console.ReadKey();
}
```
##### 輸出結果
orange 4     
lemon 12     

#### 簡單實作自己的 Intersect
```C#
public static IEnumerable<TSource> MyIntersect<TSource>(this IEnumerable<TSource> first, IEnumerable<TSource> second)
{
    if (first is null || second is null)
    {
        throw new Exception("null Exception");
    }
    return MyIntersect(first, second, null);
}

public static IEnumerable<TSource> MyIntersect<TSource>(this IEnumerable<TSource> first, IEnumerable<TSource> second, IEqualityComparer<TSource> comparer)
{
    if (first is null || second is null)
    {
        throw new Exception("null Exception");
    }
    var set = new HashSet<TSource>(second, comparer);
    foreach (var item in first)
    {
        if (set.Remove(item))
        {
            yield return item;
        }
    }
}
```
#### Intersect 的 Source Code
```
public static IEnumerable<TSource> Intersect<TSource>(this IEnumerable<TSource> first, IEnumerable<TSource> second) {
    if (first == null) throw Error.ArgumentNull("first");
    if (second == null) throw Error.ArgumentNull("second");
    return IntersectIterator<TSource>(first, second, null);
}

public static IEnumerable<TSource> Intersect<TSource>(this IEnumerable<TSource> first, IEnumerable<TSource> second, IEqualityComparer<TSource> comparer)
{
    if (first == null) throw Error.ArgumentNull("first");
    if (second == null) throw Error.ArgumentNull("second");
    return IntersectIterator<TSource>(first, second, comparer);
}

static IEnumerable<TSource> IntersectIterator<TSource>(IEnumerable<TSource> first, IEnumerable<TSource> second, IEqualityComparer<TSource> comparer)
{
    Set<TSource> set = new Set<TSource>(comparer);
    foreach (TSource element in second) set.Add(element);
    foreach (TSource element in first)
        if (set.Remove(element)) yield return element;
}

internal class Set<TElement> 
{
    int[] buckets;
    Slot[] slots;
    int count;
    int freeList;
    readonly IEqualityComparer<TElement> comparer;

    public Set() : this(null) { }

    public Set(IEqualityComparer<TElement> comparer)
    {
        if (comparer == null) comparer = EqualityComparer<TElement>.Default;
        this.comparer = comparer;
        buckets = new int[7];
        slots = new Slot[7];
        freeList = -1;
    }

    // If value is not in set, add it and return true; otherwise return false
    public bool Add(TElement value)
    {
        return !Find(value, true);
    }

    // Check whether value is in set
    public bool Contains(TElement value)
    {
        return Find(value, false);
    }

    // If value is in set, remove it and return true; otherwise return false
    public bool Remove(TElement value)
    {
        int hashCode = InternalGetHashCode(value);
        int bucket = hashCode % buckets.Length;
        int last = -1;
        for (int i = buckets[bucket] - 1; i >= 0; last = i, i = slots[i].next)
        {
            if (slots[i].hashCode == hashCode && comparer.Equals(slots[i].value, value))
            {
                if (last < 0)
                {
                    buckets[bucket] = slots[i].next + 1;
                }
                else
                {
                    slots[last].next = slots[i].next;
                }
                slots[i].hashCode = -1;
                slots[i].value = default(TElement);
                slots[i].next = freeList;
                freeList = i;
                return true;
            }
        }
        return false;
    }

    bool Find(TElement value, bool add)
    {
        int hashCode = InternalGetHashCode(value);
        for (int i = buckets[hashCode % buckets.Length] - 1; i >= 0; i = slots[i].next)
        {
            if (slots[i].hashCode == hashCode && comparer.Equals(slots[i].value, value)) return true;
        }
        if (add)
        {
            int index;
            if (freeList >= 0)
            {
                index = freeList;
                freeList = slots[index].next;
            }
            else
            {
                if (count == slots.Length) Resize();
                index = count;
                count++;
            }
            int bucket = hashCode % buckets.Length;
            slots[index].hashCode = hashCode;
            slots[index].value = value;
            slots[index].next = buckets[bucket] - 1;
            buckets[bucket] = index + 1;
        }
        return false;
    }

    void Resize()
    {
        int newSize = checked(count * 2 + 1);
        int[] newBuckets = new int[newSize];
        Slot[] newSlots = new Slot[newSize];
        Array.Copy(slots, 0, newSlots, 0, count);
        for (int i = 0; i < count; i++)
        {
            int bucket = newSlots[i].hashCode % newSize;
            newSlots[i].next = newBuckets[bucket] - 1;
            newBuckets[bucket] = i + 1;
        }
        buckets = newBuckets;
        slots = newSlots;
    }

    internal int InternalGetHashCode(TElement value)
    {
        //Microsoft DevDivBugs 171937. work around comparer implementations that throw when passed null
        return (value == null) ? 0 : comparer.GetHashCode(value) & 0x7FFFFFFF;
    }

    internal struct Slot
    {
        internal int hashCode;
        internal TElement value;
        internal int next;
    }
}
```
### [Except](https://docs.microsoft.com/zh-tw/dotnet/api/system.linq.enumerable.except?view=netframework-4.8)
Except 方法回傳差集. 換句話說 , 回傳的資料集合元素內容是以集合一為基礎 , 並且扣除與資料集合二中相同元素後的結果.
![PiiyGa4.png](https://github.com/s0920832252/LinQ-Note/blob/master/Resources/PiiyGa4.png?raw=true)
#### Except 的多載
```C#
public static IEnumerable<TSource> Except<TSource> (
            this IEnumerable<TSource> first, 
            IEnumerable<TSource> second
    )
```
```C#
public static IEnumerable<TSource> Except<TSource> (
            this IEnumerable<TSource> first,
            IEnumerable<TSource> second, 
            IEqualityComparer<TSource> comparer
    );
```
#### Except 的範例
```C#
static void Main(string[] args)
{
    var nums = new List<int> { 1, 1, 2, };
    var nums2 = new List<int> { 3, 1, 5, 3, 5 };
    IEnumerable<int> exceptQuery = nums.Except(nums2);
    foreach (var num in exceptQuery)
    {
        Console.WriteLine(num);
    }
    Console.ReadKey();
}
```
##### 輸出結果
2
```C#
class EqualityComparer<TSource> : IEqualityComparer<TSource> where TSource : class
{
    private PropertyInfo[] _properties = null;
    public bool Equals(TSource x, TSource y)
    {
        if (_properties == null)
            _properties = x.GetType().GetProperties();
        return _properties.Aggregate(true, (result, item) => result && item.GetValue(x).Equals(item.GetValue(y)));
    }

    public int GetHashCode(TSource obj)
    {
        if (_properties == null)
            _properties = obj.GetType().GetProperties();
        return _properties.Aggregate(0, (result, item) => result ^ item.GetValue(obj).GetHashCode());
    }
}

public class Product
{
    public string Name { get; set; }
    public int Price { get; set; }
}

static void Main(string[] args)
{
    Product[] products =
    {
        new Product { Name = "apple", Price = 9 },
        new Product { Name = "orange", Price = 4 },
        new Product { Name = "apple", Price = 9 },
        new Product { Name = "lemon", Price = 12 },
        new Product { Name = "orange", Price = 4 },
    };
    Product[] products2 =
    {
        new Product { Name = "lemon", Price = 12 },
        new Product { Name = "orange", Price = 4 },
    };

    var exceptProduct = products.Except(products2, new EqualityComparer<Product>());
    foreach (var product in exceptProduct)
    {
        Console.WriteLine(product.Name + " " + product.Price);
    }
    Console.ReadKey();
}
```
##### 輸出結果
apple 9     

#### 簡單實作自己的 Except
```C#
public static IEnumerable<TSource> MyExcept<TSource>(this IEnumerable<TSource> first, IEnumerable<TSource> second)
{
    if (first is null || second is null)
    {
        throw new Exception("null exception");
    }

    return MyExcept(first, second, null);
}

public static IEnumerable<TSource> MyExcept<TSource>(this IEnumerable<TSource> first, IEnumerable<TSource> second, IEqualityComparer<TSource> comparer)
{
    if (first is null || second is null)
    {
        throw new Exception("null exception");
    }
    var set = new HashSet<TSource>(second, comparer);
    foreach (var item in first)
    {
        if (set.Add(item))
        {
            yield return item;
        }
    }
}
```
#### Except 的 Source Code
```C#
public static IEnumerable<TSource> Except<TSource>(this IEnumerable<TSource> first, IEnumerable<TSource> second)
{
    if (first == null) throw Error.ArgumentNull("first");
    if (second == null) throw Error.ArgumentNull("second");
    return ExceptIterator<TSource>(first, second, null);
}

public static IEnumerable<TSource> Except<TSource>(this IEnumerable<TSource> first, IEnumerable<TSource> second, IEqualityComparer<TSource> comparer)
{
    if (first == null) throw Error.ArgumentNull("first");
    if (second == null) throw Error.ArgumentNull("second");
    return ExceptIterator<TSource>(first, second, comparer);
}

static IEnumerable<TSource> ExceptIterator<TSource>(IEnumerable<TSource> first, IEnumerable<TSource> second, IEqualityComparer<TSource> comparer) {
    Set<TSource> set = new Set<TSource>(comparer);
    foreach (TSource element in second) set.Add(element);
    foreach (TSource element in first)
        if (set.Add(element)) yield return element;
}

internal class Set<TElement> 
{
    int[] buckets;
    Slot[] slots;
    int count;
    int freeList;
    readonly IEqualityComparer<TElement> comparer;

    public Set() : this(null) { }

    public Set(IEqualityComparer<TElement> comparer)
    {
        if (comparer == null) comparer = EqualityComparer<TElement>.Default;
        this.comparer = comparer;
        buckets = new int[7];
        slots = new Slot[7];
        freeList = -1;
    }

    // If value is not in set, add it and return true; otherwise return false
    public bool Add(TElement value)
    {
        return !Find(value, true);
    }

    // Check whether value is in set
    public bool Contains(TElement value)
    {
        return Find(value, false);
    }

    // If value is in set, remove it and return true; otherwise return false
    public bool Remove(TElement value)
    {
        int hashCode = InternalGetHashCode(value);
        int bucket = hashCode % buckets.Length;
        int last = -1;
        for (int i = buckets[bucket] - 1; i >= 0; last = i, i = slots[i].next)
        {
            if (slots[i].hashCode == hashCode && comparer.Equals(slots[i].value, value))
            {
                if (last < 0)
                {
                    buckets[bucket] = slots[i].next + 1;
                }
                else
                {
                    slots[last].next = slots[i].next;
                }
                slots[i].hashCode = -1;
                slots[i].value = default(TElement);
                slots[i].next = freeList;
                freeList = i;
                return true;
            }
        }
        return false;
    }

    bool Find(TElement value, bool add)
    {
        int hashCode = InternalGetHashCode(value);
        for (int i = buckets[hashCode % buckets.Length] - 1; i >= 0; i = slots[i].next)
        {
            if (slots[i].hashCode == hashCode && comparer.Equals(slots[i].value, value)) return true;
        }
        if (add)
        {
            int index;
            if (freeList >= 0)
            {
                index = freeList;
                freeList = slots[index].next;
            }
            else
            {
                if (count == slots.Length) Resize();
                index = count;
                count++;
            }
            int bucket = hashCode % buckets.Length;
            slots[index].hashCode = hashCode;
            slots[index].value = value;
            slots[index].next = buckets[bucket] - 1;
            buckets[bucket] = index + 1;
        }
        return false;
    }

    void Resize()
    {
        int newSize = checked(count * 2 + 1);
        int[] newBuckets = new int[newSize];
        Slot[] newSlots = new Slot[newSize];
        Array.Copy(slots, 0, newSlots, 0, count);
        for (int i = 0; i < count; i++)
        {
            int bucket = newSlots[i].hashCode % newSize;
            newSlots[i].next = newBuckets[bucket] - 1;
            newBuckets[bucket] = i + 1;
        }
        buckets = newBuckets;
        slots = newSlots;
    }

    internal int InternalGetHashCode(TElement value)
    {
        //Microsoft DevDivBugs 171937. work around comparer implementations that throw when passed null
        return (value == null) ? 0 : comparer.GetHashCode(value) & 0x7FFFFFFF;
    }

    internal struct Slot
    {
        internal int hashCode;
        internal TElement value;
        internal int next;
    }
}
```
### 參考
[Set Operations in LINQ](https://www.tutorialspoint.com/linq/linq_set_operations.htm)     
[LINQ | Set Operator | Distinct](https://www.geeksforgeeks.org/linq-set-operator-distinct/)     
[LINQ | Set Operator | Union](https://www.geeksforgeeks.org/linq-set-operator-union/)     
[LINQ | Set Operator | Intersect](https://www.geeksforgeeks.org/linq-set-operator-intersect/)     
[LINQ | Set Operator | Except](https://www.geeksforgeeks.org/linq-set-operator-except/)     





### Thank you! 

You can find me on

- [GitHub](https://github.com/s0920832252)
- [Facebook](https://www.facebook.com/fourtune.chen)

若有謬誤 , 煩請告知 , 新手發帖請多包涵

# :100: :muscle: :tada: :sheep: 
