> swift 中的闭包类似于 OC 中的 Block，但是使用比 Block 更广泛

# 闭包的简单说明

1. 保存提前准备好的代码
2. 在需要的时候执行
3. 可以当做参数传递
4. 在 OC 中 Block 是匿名的函数
5. 在 Swift 中闭包是特殊的函数

# 闭包的使用场景

1. 异步执行完成回调
2. 控制器间回调
3. 自定义视图回调
4. 回调的特点
    - 以参数回调处理结果
    - 返回值为 Void

# 闭包的简单使用

1.  无参数无返回值

    ```swift
     //最简单的闭包（无返回值无参数）
     // ( ) -> ( )
     let bibao1 = {
         print("hello, world")
     }
     //调用闭包
     bibao1()
    ```
2.  有参数无返回值
    - 在闭包中，参数、返回值、实现代码都可以写在 { } 中
    - 使用一个关键字 `in` 分割定义和实现
    - { 形参列表 -> 返回值类型 in   //实现代码 }

    ```swift
    //带参数无返回值的闭包
    // ( Int ) -> ( )
    let bibao2 = {
        (x: Int) -> () in

        print(x)
    }
    bibao2(10)
    ```
3.  有参数有返回值

    ```swift
      //带参数带返回值的闭包
      // (Int) -> Int
      let bibao3 = {
          (x: Int) -> Int in
      
          return x + 10
      }
      print(bibao3(10))    
    ```

## 闭包作为参数传递

- 闭包和 Block 一样可以作为参数进行传递

    ```swift
    //闭包作为参数传递
    // (Int, Int) -> Int
    let bibao4 = {
    (x: Int, y: Int) -> Int in
    
        return x + y
    }

    func test(x: Int, y: Int, bibao: (Int, Int) -> Int) {
        bibao(x, y)
    }
    test(x: 10, y: 20, bibao: bibao4)
    ```

## 尾随闭包
- 如果函数的最后一个参数是闭包，函数参数可以提前结束，最后的一个参数使用 { } 来包装闭包的代码
- 使用尾随闭包对上述的参数传递代码进行调整，可以使用以下方式进行修改

    ```swift
    //尾随闭包
    func test(x: Int, y: Int, bibao: (Int, Int) -> Int) {
        bibao(x, y)
    }
    
    test(x: 10, y: 20) {
        (x, y) -> Int in
    
        return x + y
    }
    ```

## 逃逸闭包
- 在 Swift 中 闭包默认是非逃逸的，不能被其他对象引用
- @escaping 修饰的就是逃逸闭包，可以被其他对象引用

    ```swift
    func test(a: Int, aa: ()->()) {
        aa()
    }

    func getData(result: @escaping ([String]) -> ()) {

        test(a: 10, aa: {
            result(["1", "2"])
        })
    }
    ```

## 闭包的循环引用
- 由于 { } 的作用域，在使用闭包的同时要注意循环引用的问题
- 在 OC 中可以使用 __weak 和 __unsafe_unretained  两种方式
- 在 Swift 中主要使用 weak 和 unowned 两种方式来解决

1.  `weak`
    - Swift 中推荐使用的方法
    - 需要注意解包问题
    - 修饰的 self 都是弱引用

    ```swift
    class Person {
        var num: Int = 0
        var bb: (() -> ())?

        // weak 方式一
        func test1() {
            weak var weakSelf = self
            bb = {
                print("test1", weakSelf!)
                let pr = weakSelf?.num
                pr! + 10
            }
            bb!()
        }
        
        // weak 方式二
        func test2() {
            bb = {
                //标识，在这个闭包里使用的所有 self 都是弱引用
                [weak self]
                () -> () in
            
                print(self!)
                print(self!)
            }
            bb!()
        }
    }
    ```
2.  `unowned`
    - 修饰的 self 都是 assign 的，不会强引用
    - 如果对象释放，指针地址不会变化
    - 若被释放之后继续调用，会出现野指针问题

    ```swift
    class Person {
        var num: Int = 0
        var bb: (() -> ())?

        // unowned 方式一
        func test1() {
            unowned let weakSelf = self
            bb = {
                print("test1", weakSelf)
                let pr = weakSelf.num
                pr + 10
            }
            bb!()
        }
        
        // unowned 方式二
        func test2() {
            bb = {
                //标识，在这个闭包里使用的所有 self 都是弱引用
                [unowned self]
                () -> () in
            
                print(self)
                print(self)
            }
            bb!()
        }
    }
    ```

