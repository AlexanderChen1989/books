用接口的思想来理解graphql
接口是行为的集合，定义了更系统交互的界面。
行为的实体是函数，函数包括定义和实现，而接口只关注函数的定义。
函数定义包括传人参数和返回值，包含了参数名称和类型，包含了返回值的类型。
下面我们用Go语言定义了一个简单的类型和接口。
```go
// 定义的User类型
type User struct {
    ID int
    Name string
    Age int
    Posts []Post
}

type Post struct {
    ID int 
    Title string
    Owner User
}

// 定义的查询接口
type Query interface {
    FetchUsers() []User
    FetchPosts(userID int) []Post
    GetUserById(id int) *User // 定义了函数的参数和返回值
}

// 定义的修改接口
type Mutation interface {
    CreateUser(name string, age int) User
    DeleteUser(id int) *User
}
```

graphql可以看成是一种接口，里面包括了参数和返回值的定义。
graphql把接口分成两类。
一种是用来查询的接口，叫做Query，这种接口不改变后台数据。
一类是用来修改是接口，叫做Mutation, 这种接口成调用会改变后台数据。
这里的Query和Mutation只是一种分类方式，本身不约束接口实现，也就是说，你可以在一个查询结合中进行数据修改，当然这是不对的。

当然如果graphql只是这些，那就没什么意思了。
我们知道，接口如果成功调用，会返回一个完整的数据。比如，GetUserById成功调用后会返回一个完整的User给我们。
这里包括大量User的全部属性, 形如User {ID: 0, Name: "Alex", Age: 28}。
但是如果我们带宽太窄，只需要User的Name信息？
我们喜欢活动形如 User {Name: "Alex"} 的数据？
这里就是graphql跟接口不同的地方。使用graphql你可以声明你需要什么属性。
```graphql
{
    GetUserById(id: 1) {
        Name
    }
}
```
接口，你调用一个函数，返回一个结果。你返回多种结果，调用多次函数。
使用graphql，你能够把多个对数据的要求放在一个请求里面。
```graphql
{
    GetUserById(id: 1) {
        Name
    }
    
    FetchPosts(userId: 1) {
        title
    }
}
```
graphql支持顺着类型定义嵌套的抓取信息。
上面的查询也可以写成下面的形式。
```graphql
{
    GetUserById(id: 1) {
        Posts {
            title
        }
    }
}
```
当然你也可以丧心病狂的不断嵌套抓取数据。
```graphql
{
    GetUserById(id: 1) {
        Posts {
            title
            Owner {
                Name {
                    Posts {
                        title
                        ...
                    }
                }
            }
        }
    }
}
```
这种按照接口的类型定义，不断嵌套的抓取数据正是graphql中graph的由来，表达了graph + ql = 按图 + 索骥。
当然这种变态的抓取是非常恐怖的，如果有坏人故意发这种请求搞死我们，我们就完了，所以必须有一种方式来限制这种行为。
好在，我们可以对用户的请求进行ast语法分析，限制嵌套的深度，比如，我们可以让你最多嵌套3层。


总结

graphql本质上是一种对接口的扩展，包括了接口的严格类型定义，同时又扩展是接口的使用方式，满足不同端对数据的要求。
