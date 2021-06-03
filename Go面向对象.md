## Go面向对象



## 简介

最近思考了一下这几年写的业务，都是用面向过程思路来实现。之所以用面向过程

一是因为大家都这么写，很少去思考是否有更好的实现

二是业务简单，使用面向过程编程可以很方便的实现

三是业务需要快速上线，面向过程的方法更快更直接

但是弊端也很明显，随着业务不断的积累，项目改动起来比较困难，需要不断梳理前人是怎么写的、有什么影响，测试需要做大量的测试，确保功能是正常的。有时候即使是相似或者迭代性的需求也无法保证快速上线。而且即使耗费了大量的人力，也很难保证没有遗漏。这时候就想起面向对象的好。

## 四大特性

面向对象编程有四大特性

1. 封装特性

   封装也叫作信息隐藏或者数据访问保护。类通过暴露有限的访问接口，授权外部仅能通过类提供的方式来访问内部信息或者数据。它需要编程语言提供权限访问控制语法来支持，例如Java中的private、protected、public关键字。封装特性存在的意义，一方面是保护数据不被随意修改，提高代码的可维护性;另一方面是仅暴露有限的必要接口，提高类的易用性。

2. 抽象特性

   封装主要讲如何隐藏信息、保护数据，那抽象就是讲如何隐藏方法的具体实现，让使用者只需要关心方法提供了哪些功能，不需要知道这些功能是如何实现的。抽象可以通过接口类或者抽象类来实现，但也并不需要特殊的语法机制来支持。抽象存在的意义，一方面是提高代码的可扩展性、维护性，修改实现不需要改变定义，减少代码的改动范围;另一方面，它也是处理复杂系统的有效手段，能有效地过滤掉不必要关注的信息。

3. 继承特性

   继承是用来表示类之间的is-a关系，分为两种模式:单继承和多继承。单继承表示一个子类只继承一个父类，多继承表示一个子类可以继承多个父类。为了实现继承这个特性，编程语言需要提供特殊的语法机制来支持。继承主要是用来解决代码复用的问题。

4. 多态特性

   多态是指子类可以替换父类，在实际的代码运行过程中，调用子类的方法实现。多态这种特性也需要编程语言提供特殊的语法机制来实现，比如继承、接口类、duck-typing。多态可以提高代码的扩展性和复用性，是很多设计模式、设计原则、编程技巧的代码实现基础。

## 实现

想要用面向对象编程来实现业务逻辑，必然需要掌握的是如何使用这门语言实现四大特性。在掌握了这份知识的基础上，才有可能灵活的使用设计模式，设计出好的架构。今后学习新的语言的时候，对于面向对象的实现，需要成为关注点之一。看下面的内容需要掌握基本的Go语言知识。

### 封装

Go中封装的实现方法为使用结构体，并给结构体添加相应的方法。

一般的实现步骤如下：

1. 将结构体、字段的首字母小写；
2. 给结构体所在的包提供一个工厂模式的函数，首字母大写，类似一个构造函数；
3. 提供一个首字母大写的Set方法（类似其它语言的public），用于对属性判断并赋值；
4. 提供一个首字母大写的Get方法（类似其它语言的public），用于获取属性的值。

举例：对于员工，不能随便查看年龄，工资等隐私，并对输入的年龄进行合理的验证。代码结构如下

```go
package model

import "fmt"

type person struct {
  Name string
  age  int //其它包不能直接访问..
  sal  float64
}

//写一个工厂模式的函数，相当于构造函数
func NewPerson(name string) *person {
  return &person{
     Name: name,
  }
}

//为了访问age 和 sal 我们编写一对SetXxx的方法和GetXxx的方法
func (p *person) SetAge(age int) {
  if age > 0 && age < 150 {
     p.age = age
  } else {
     fmt.Println("年龄范围不正确..")
     //给程序员给一个默认值
  }
}
func (p *person) GetAge() int {
  return p.age
}

func (p *person) SetSal(sal float64) {
  if sal >= 3000 && sal <= 30000 {
     p.sal = sal
  } else {
     fmt.Println("薪水范围不正确..")
  }
}

func (p *person) GetSal() float64 {
  return p.sal
}
```

关于这个代码，有两点需要说明

1. 关于大小写

2. - 小写字母开头的函数只在本包内可见，大写字母开头的函数才能被其他包使用。这个规则也适用于类型和变量的可见性。
   - 大小写影响了可见性，大写字母开头等同于public，小写字母开头等同于private，这种做法不仅免除了public、private关键字，更重要的是统一了命名风格。

3. 结构体方法参数

4. - 如果要求对象必须以指针传递，这有时会是个额外成本，因为对象有时很小（比如4字节），用指针传递并不划算。
   - 只有在你需要修改对象的时候，才必须用指针。

### 继承

Go语言根本就不支持面向对象思想中的继承语法。

从另一个维度而言，Go语言也提供了继承，但是采用了组合的文法，所以我们将其称为匿名组合。

举例：

```go
package main

import "fmt"

type Base struct {
  Name string
}

func (b *Base) SetName(name string) {
  b.Name = name
}

func (b *Base) GetName() string {
  return b.Name
}

// 组合，实现继承
type Child struct {
  base Base // 这里保存的是Base类型
}

// 重写GetName方法
func (c *Child) GetName() string {
  c.base.SetName("modify...")
  return c.base.GetName()
}

// 实现继承，但需要外部提供一个Base的实例
type Child2 struct {
  base *Base // 这里是指针
}

//
type Child3 struct {
  Base
}

type Child4 struct {
  *Base
}

func main() {
  c := new(Child)
  c.base.SetName("world")
  fmt.Println(c.GetName())

  c2 := new(Child2)
  c2.base = new(Base) // 因为Child2里面的Base是指针类型，所以必须提供一个Base的实例
  c2.base.SetName("ccc")
  fmt.Println(c2.base.GetName())

  c3 := new(Child3)
  c3.SetName("1111")
  fmt.Println(c3.GetName())

  c4 := new(Child4)
  c4.Base = new(Base)
  c4.SetName("2222")
  fmt.Println(c4.GetName())
}
```

关于这段代码，有几点需要说明

1. 可以使用匿名组合和非匿名组合，如果使用非匿名组合，则调用的时候需要显示指定变量名称。如果使用匿名组合，则无需显示指定变量名称，当然也可以显示调用，如c3.Base.Name。
2. 在Go语言中，还可以以指针方式从一个类型“派生”：如Child4，这段Go代码仍然有“派生”的效果，只是Child4创建实例的时候，需要外部提供一个Base类实例的指针。
3. 在“派生类”Child3没有改写“基类”Base的成员方法时，相应的方法就被“继承”，例如在上面的例子中，调用c3.GetName()和调用c3.Base.GetName()效果一致。在“派生类”Child改写“基类”Base的成员方法，c.GetName()会调用派生类的方法，如果想调用基类的方法，可以显示调用。
4. 如果“派生类”和“基类”有相同的变量名Name，所有的派生类的Name成员的访问都只会访问到最外层的那个Name变量，基类的Name变量相当于被覆盖了，可以用显示引用。

### 多态

在Go语言中，一个类只需要实现了接口要求的所有函数，我们就说这个类实现了该接口。如果类实现了接口，便可将对象实例赋值给接口。

举例：

```go
package main

import "fmt"

type Money interface {
  show() string
}

type OldMoney struct {
}

func (oldMoney *OldMoney) show() string {
  return "I am old money"
}

type NewMoney struct {
}

func (newMoney *NewMoney) show() string {
  return "I am new money"
}

func PrintMoney(l []Money) {
  for _, item := range l {
     fmt.Println(item.show())
  }
}

func main() {
  moneyList := []Money{new(OldMoney), new(NewMoney), new(OldMoney)}
  PrintMoney(moneyList)
}
```

关于这段代码，有几点需要说明：

1. 接口赋值并不要求两个接口必须等价。如果接口A的方法列表是接口B的方法列表的子集，那么接口B可以赋值给接口A。

1. Go语言可以根据下面的函数：func (oldMoney OldMoney) show() string自动生成一个新的show()方法：func (oldMoney *OldMoney) show() string，所以赋值时使用引用还是对象，需要考虑函数参数的类型。

## 结论

简单梳理了一下Go语言的特性，使用Go做面向对象编程及其简单。