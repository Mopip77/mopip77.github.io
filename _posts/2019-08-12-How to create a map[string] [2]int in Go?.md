---
layout: post
title: How to create a map[string] [2]int in Go?
tags:
- go
---

在编写go的时候碰到了个问题，我在创建`map[string][2]int`想做个key(string)，value(pair)的map时，发现创建没问题。当我使用`a["abc"][0] = 1`时，报错了。

Question:
```go
    a := map[string][5]int
    a[“bb”] = [5]int{1,2,3}
    a[“bb”][1] = 5. // ERROR, 如果value是struct的话也是无法直接修改里面的成员变量的
```
简而言之就是,map的value是无法寻址的,因为可能出现删除键后又放回的错误(不太懂),但是如果value值本身是引用类型则可以直接修改, 否则可以直接声明为map of pointer

```text
官方解释:
The two cases really are different.  Given a map holding a struct
    m[0] = s
is a write.
    m[0].f = 1
is a read-modify-write.  That is, we can implement
    m[0] = s
by passing 0 and s to the map insert routine.  That doesn't work for
    m[0].f
Instead we have to fetch a pointer, as you suggest, and assign through the pointer.
Right now maps are unsafe when GOMAXPROCS > 1 because multiple threads writing to the
map simultaneously can corrupt the data structure.  This is an unfortunate violation of
Go's general memory safeness.  This suggests that we will want to implement a
possibly-optional safe mode, in which the map is locked during access.  That will work
straightforwardly with the current semantics.  But it won't work with your proposed
addition.
That said, there is another possible implementation.  We could pass in the key, an
offset into the value type, the size of the value we are passing, and the value we are
passing.
A more serious issue: if we permit field access, can we justify prohibiting method
access?
    m[0].M()
But method access really does require addressability.
And let's not ignore
    m[0]++
    m[0][:]
```

参考链接：
https://stackoverflow.com/questions/38477567/how-to-create-a-mapstring-2int-in-go