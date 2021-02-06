---
title: "Template in C#"
date: 2018-03-05T05:43:58+08:00
categories: ["C#"]
---

泛型（参数化类型），将程序逻辑从具体的类型中分离出来，从而提高代码的复用性。
`.Net`的`System.Collections.Generic`命名空间中定义的大多数类型使用了泛型技术，比如`List<T>`，`Stack<T>`，`Tuple<T1, T2, T3, ...>`，`SortedList<TKey, TValue>`等，另外，委托类型也提供了泛型委托`Action`和`Function`。泛型接口定义了一种参数化类型的接口，保障程序的类型安全。


#### 泛型树节点接口

以树的遍历为例，不管树节点的类型和实现是什么，只要知道当前节点的深度和其子节点，就可以进行树的遍历。因此，定义泛型树节点接口`ITreeNode<T>`。

```cs
public interface ITreeNode<T> where T : class
{
    int Depth { get; set; } //深度
    List<T> GetChildren();	//获取子节点
}
```

#### 泛型树的约束

在泛型树的定义中添加约束`class`和`ITreeNode<T>`，这样，在编译期就可以确保类型安全，用未实现`ITreeNode<T>`接口的`class`来实例化`Tree<T>`时会编译出错。

```cs
public class Tree<T> where T : class, ITreeNode<T>
{
    //树根
    public T Root { get; set; }
    //最大深度
    public int MaxDepth { get; set; }

    public void Travel()
    {
        Travel(Root, MaxDepth);
    }

    public void Travel(int maxDepth)
    {
        Debug.Assert(maxDepth >= 0);
        Travel(Root, Math.Min(maxDepth, MaxDepth));
    }

    public void Travel(T parentNode, int maxDepth)
    {
        Debug.Assert(maxDepth <= MaxDepth, "maxdepth cannot be larger than tree Maxdepth");
        if (parentNode.Depth > maxDepth) return;
        Console.WriteLine(parentNode.ToString());
        var children = parentNode.GetChildren();
        if (children == null) return;
        foreach (var child in children)
        {
            Travel(child, maxDepth);
        }
    }
}
```

#### TreeNode实现


`MyTreeNode`实现了接口，通过静态属性`GetChildrenFunc`，具体的`GetChildren`可以由用户实现。若用户未定义`GetChildren`，则程序抛出异常。`GetChildrenFunc`也是通过泛型来确保编译期的类型安全。


```cs
public class MyTreeNode : ITreeNode<MyTreeNode>
{
    public int Content { get; set; }
    public static Func<MyTreeNode, List<MyTreeNode>> GetChildrenFunc { get; set; }

    #region constructor

    public MyTreeNode(int content = 0, int depth = 1)
    {
        Content = content;
        Depth = depth;
    }

    #endregion

    #region field in interface

    public int Depth { get; set; }

    public List<MyTreeNode> GetChildren()
    {
        if (GetChildrenFunc != null)
        {
            return GetChildrenFunc(this);
        }
        throw new NotImplementedException("GetChildrenFunc");
    }

    #endregion

    public override string ToString()
    {
        return string.Format("depth is {0}, content is {1}", Depth.ToString(), Content.ToString());
    }
}

//未实现接口，测试编译期类型安全
public class MyTreeNode2
{

}
```

#### 测试代码

```cs
public class TestTree
{
    public static void Test()
    {
        //无GetChildrenFunc的实现时，运行时抛出NotImplementedException
        MyTreeNode.GetChildrenFunc = (node => new List<MyTreeNode>()
        {
            new MyTreeNode(node.Content*2, node.Depth + 1),
            new MyTreeNode(node.Content*2 + 1, node.Depth + 1)
        });


        var tree1 = new Tree<MyTreeNode>() { Root = new MyTreeNode(1), MaxDepth = 3 };
        tree1.Travel();

        //未注释时，无法通过编译
        //var tree2 = new Tree<MyTreeNode2>();

    }
}
```