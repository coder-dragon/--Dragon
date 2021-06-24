### 理解闭包函数

--- 

首先我们来看一个普通的函数作用域的实现

```
--普通的变量定义
    function a()
        local flag = 1
        print("打印结果："..flag)
    end
```

```打印结果：1```

这段函数会在他执行执行完这个方法后,flag的作用域的失效了。

---

下面让我们来看这段基本的函数内部的闭包实现

```
    --普通的变量定义
    function a()
        local flag = 1
        local func = function(n)
            flag = flag + n
            return flag
        end
        
        print("打印结果："..func(1))
    end
```

```打印结果：2```

首先这个函数的flag，func变量都在这个函数的作用域中，在函数内部func可以共享定义在func之前的局部变量定义，所以当我们求值时，可以取到flag的值，进而获得正确的结果

---

那么闭包函数是否可以在闭包函数内访问自身呢，让我们来看下面这段定义

```
    --普通的变量定义
    function a()
        local flag = 1
        local func = function(n)
            if flag == 5 then
                return flag
            end
            flag = flag + func(1)
        end
        
        print("打印结果："..func(1))
    end
```

```打印结果就不说了，因为这段代码会报错```

思考，为什么会报错呢，实际上因为func闭包定义的时候，这个闭包func的局部变量func还没在a方法中被定义，可以把整个方法赋值给func看做是一行代码变量定义。这样就好理解多拉。

那么我们改进一下写法。

```
    --普通的变量定义
    function a()
        local flag = 1
        local func
        func = function(n)
            if flag == 5 then
                return flag
            end
            flag = flag + func(1)
        end
        
        print("打印结果："..func(1))
    end
```

```打印结果：5```

这时候程序就能正常运行，因为**func定义被提前了**，他作为a的局部变量可以被下面的闭包共享作用域，虽然被定义的时候func是空值，但是当我们执行方法的时候**他已经被赋值了**，也就是当前这个匿名方法

再让我们看一段代码，更进一步

```
    --普通的变量定义
    function a()
        local func
        func = function()
            return func   
        end
        return func
    end

    local b = a()
    local c = a()()
```

让我们思考一下上面这段代码的各个变量的作用域，很显然 **local func** 的时候他在作用域被固定在了方法a的方法内，那么再让我们**思考**最后这个 **return func**的作用域呢，这个变量的作用域存在在a方法返回后持有他这个func变量的变量定义的作用域，也就是 **local b 的作用域**，但是我们的**闭包方法又返回了方法本身，那么这个函数的作用域在哪呢，什么时候被释放呢，实际上代码中已经给出了答案，也就是 local c ，参考 local b ,这样解释大家是不是对闭包和闭包函数作用域有了更深的了解了呢**

---

那么下面就让我们讲本文章的重点吧，如果利用**闭包函数**和函数的**作用域**来实现优雅的编程

### 函数的柯里化的应用

---

普通的加法实现

```
    function add(a, b)
        return a + b
    end

    add(1, 2)
```

当我们已经有了上面那个方法定义后，那么当我们需要相加多个数字的时候需要怎么做呢，可能有人会这么写

```
    add(add(1, 2), 3)
```

或者重新定义方法，

```
    function add(a, b, c)
        return a + b + c
    end
```

但是如果我们不希望他们马上相加出结果，因为中间可能有其他的判断逻辑，甚至有一些异步操作的情况下,下面是普通的代码实现逻辑

```

    function add(a, b)
        return a + b
    end

    local result = 0

    -- 第一个判断分支
    if true then
        result = add(result, 1)
    end
 
    -- 第二个判断分支
    if false then
        result = add(result, 2)
    end

    -- 第三个判断分支
    if true then
        result = add(result, 3)
    end


    -- 这是一个异步调用，会对result进行操作，但是却没有返回result
    other = async(result):next()

    -- result = ????????????
```

上面是普通的方法实现逻辑，一些逻辑判断，是否累加，或者进行异步调用

**从这段代码的上下文我们没有办法直接获取result最后确切的值，可能需要从其他地方获取，或者让异步调用增加返回值，而且代码的实现也不够美观，我们需要每次对result进行实时计算，并得出结果，传给下面的步骤，但是可能在中间过程的时候我们并不关心result是什么状态，我们只想要在最后一步的时候，知道result的计算结果**

**当当当，这时候我们的函数柯里化的作用就来了，让我们假设有这么一个函数，可以让
 add(1, 2) 的调用方式变成add(1)(2),或者add(1),add(2),并且，并且只有在最后调用()的时候才会返回计算结果，那么调用方式就变成了add(1), add(2), add(),或者add(1)(2)()**

**让我们来看如果我们有这么一个方法以后上面的代码可以怎么优化吧**

```
    --这是一个可以让函数实现柯里化的函数
    function curry(fn)
        ...
    end

    function add(a, b)
        return a + b
    end

    local curry_add = curry(add)

    -- 第一个判断分支
    if true then
        curry_add(1)
    end
 
    -- 第二个判断分支
    if false then
        curry_add(2)
    end

    -- 第三个判断分支
    if true then
        curry_add(3)
    end


    -- 这是一个异步调用，会对result进行操作，但是却没有返回result
    other = async(curry_add):next()

    -- 我们终于可以在上下文中拿到result的结果了
    result = curry_add()
```

是不是很酷，感觉写代码都不那么累了，不需要写那些多余的字符，并且可以在最后直接拿到result的结果

---

### 函数的柯里化的实现

那么下面就贴上代码的实现，如果有不对的地方大佬可以指出错误

```
-- 柯里化函数调用
-- fn(a, b) 操作方法
-- 如果调用()则返回结果并则清除缓存
function curry(fn)
    local cache = {}
    local count, result, ret
    ret = function(arg)
        if arg == nil then
            if count == 0 or count == nil then
                cache, count = nil, nil
                return result
            end
            for i = 1, #cache do
                count = count - 1
                if cache[i+1] then
                    result = fn(result or cache[i], cache[i+1])
                else
                    if result then
                        cache, count = nil, nil
                        return result
                    else
                        result = cache[i]
                        cache, count = nil, nil
                        return result
                    end
                end
            end
            cache, count = nil, nil
            return result
        else
            table.insert(cache, arg)
            count = #cache
            return ret
        end
    end
    return ret
end
```

```

function add(a, b)
    return a + b
end

local curry_add = curry(add)

add(1)
add(2)
add(3)

print("打印结果："..add())

```

```
打印结果：6
```

```
add(1)(2)(3)()
```

```
打印结果：6
```

### 结语

那么今天就教程就到这里结束了，如果觉得我说的有用的话就在我的[>>>>>> 戳这里 github项目<<<<<<](https://github.com/coder-dragon/Dragon)点上一个小小的star吧，咖啡就不用请我喝了，屑屑(比心)


