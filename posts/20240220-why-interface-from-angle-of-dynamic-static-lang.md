# 从动态/静态语言角度理解接口

> 发布日期：2024-02-20 19:03:35

[![pFN2Bon.webp](https://s11.ax1x.com/2024/02/22/pFN2Bon.webp)](https://imgse.com/i/pFN2Bon)

## 为什么需要接口

#### 什么是接口

在编程语言中，接口（interface）是一种规范或契约，用于定义类或对象应该提供哪些方法、属性或事件。接口提供了一种抽象的方式来描述类或对象的行为，使得不同的类或对象可以通过实现相同的接口来实现相同的功能。

接口通常包含一组方法和属性的声明，但不包含任何具体的实现。通过实现相同的接口，不同的类或对象可以实现相同的功能，从而减少了代码的重复，提高了代码的可重用性和可维护性。

在一些编程语言中，如 Java 和 Go，接口是一种独立的类型，可以被类或对象实现。在其他编程语言中，如 Python 和 Javascript，接口通常被视为约定或协议，而不是独立的类型。

#### 接口的作用

抽象类的行为：接口提供了一种抽象的方式来描述类或对象的行为，使得开发人员可以更加清晰地理解和设计类或对象的行为。

规范类或对象的行为：接口定义了类或对象应该提供哪些方法、属性或事件，从而规范了类或对象的行为。

提高代码的可重用性和可维护性：通过实现相同的接口，不同的类或对象可以实现相同的功能，从而减少了代码的重复，提高了代码的可重用性和可维护性。

降低代码的耦合性：通过接口，类或对象之间的依赖关系得到了解耦，从而提高了代码的灵活性和可扩展性。

## 不同编程语言中接口的表现

#### java 中的接口

java 属于静态语言，其变量的类型在编译时就已经确定。在实际的业务场景中，经常会有某个变量要根据实际的情况调用不同类型对象的方法，如果这个变量依赖于具体的类，那么是很难实现的。

因此在 java 中，相同场景的类一般都会继承同一个接口，这样变量的类型被定义成接口的类型，这个变量就依赖于抽象而不是具体，在真正调用的时候根据实际情况选择不同的实现类进行调用。

在下面的代码中 Desktop 和 Laptop 都实现了 Computer 这个接口并强制实现了接口定义的方法 getContent()。

```java
public interface Computer {
    public String getContent(String param);
}

public class Desktop implements Computer {
    @Override
    public String getContent(String param) {
        return "Get content from Desktop: " + param;
    }
}

public class Laptop implements Computer {
    @Override
    public String getContent(String param) {
        return "Get content from Desktop: " + param;
    }
}
```

在对 Desktop 和 Laptop 的实际使用中（见下面代码）。根据不同的业务需求通过工厂方法 getComputer 返回 Desktop 或 Laptop 的实例，变量 c 的类型是 Computer 接口类，也就是 c 依赖于抽象的接口而不是具体的 Desktop 或者 Laptop。这样提升了代码的灵活性，同时实现了代码的解耦。

```java
public class Calc {
    public static Computer getComputer(int cond) throws Exception {
        if (cond == 1) {
            return new Desktop();
        } else if (cond == 2) {
            return new Laptop();
        } else {
            throw new Exception("Wrong cond...");
        }
    }

    public static void main(String[] args) throws Exception {
        int cond = 1;
        Computer c = getComputer(1);
        String str = c.getContent("http://abc.com/");
        System.out.println(str);
    }
}
```

值得注意的是，在 java 中对接口的定义一般是从定义者出发的。在下面的代码中，main 方法中 c 需要的是一个实现了 Computer 接口的实现类，也就是说 Computer 是提前约定好的，c 只会接受实现了 Computer 接口的类，其他的类是不行的。

#### go 中的接口

go 也属于静态语言。如同 java 语言一样，也需要接口来提高程序的灵活性和依赖的解耦。

在下面代码中，定义 Desktop 和 Laptop 两个结构体，并都定义 GetContent 方法。

```go
type Laptop struct{}

func (Laptop) GetContent(param string) string {
	return "Get content from Laptop: " + param
}

type Desktop struct{}

func (Desktop) GetContent(param string) string {
	return "Get content from Desktop: " + param
}
```

在实际业务中（见下面代码），需要根据不同的场景来选择使用 Desktop 或者 Laptop 的 GetContent 方法。

与 java 不同的是，虽然 c 同样要依赖于抽象的接口而非具体的实现类，但是 Desktop 和 Laptop 这两个实现类并不需要同时实现定义出来的 ComputerIntl 接口。ComputerIntl 接口在这里更像是一种约定或协议，只要是满足这种协议的具体类就可以看作这个接口的“实现”。这种定义也叫做鸭子类型。

但是 go 的类型并不是真正的鸭子类型，是类鸭子类型。原因在于 go 的类型不是动态绑定的，而是编译时绑定的。

```go
type ComputerIntl interface {
	GetContent(string) string
}

func getComputer(cond int) ComputerIntl {
	if cond == 1 {
		return Desktop{}
	} else if cond == 2 {
		return Laptop{}
	} else {
		panic("Wrong cond...")
	}
}

func MainCalc() {
	cond := 1
	var c ComputerIntl = getComputer(cond)
	resStr := c.GetContent("http://abc.com/")
	fmt.Println(resStr)
}
```

除此之外，go 的接口定义和 java 不同的是，其接口类型的定义是从使用者角度出发的。由于在 go 中没有必要要求一组类（结构体实现）必须实现某个接口才表明这组类有相同的接口行为，使用者可以根据实际情况对一组含有相同行为的类定义接口，这更好的提高了程序的灵活性。

#### python 中的接口

python 属于动态语言，并且在 python 中没有接口定义的关键词。原因在于对于 python 语言来说，其接口的使用是约定性的。

在下面的代码中，我们同样定义 Desktop 和 Laptop 两个计算机类，并且定义一个工厂函数，这个工厂函数根据不同的业务条件来返回不同的计算机类。

```python
class Laptop:
    def get_content(self, param):
        return "Get content from Laptop: " + param


class Desktop:
    def get_content(self, param):
        return "Get content from Desktop: " + param


def get_computer(cond: int):
    if cond == 1:
        return Desktop()
    elif cond == 2:
        return Laptop()
    else:
        raise Exception("Wrong cond...")
```

在下面程序时候的时候，由于 python 是鸭子类型的语言，变量类型是动态绑定的，computer 变量可以接受任何类型的值，因此不需要显示的使用接口来定义变量的类型，至于 computer 是否可以真正的调用 get_content 方法，是开发者在开发的时候就通过约定或协议声明好的。

```python
if __name__ == "__main__":
    cond = 1
    computer = get_computer(cond)
    res = computer.get_content("https://abc.com/")
    print(res)
```

#### javascript 中的接口

和 python 一样 javascript 也是动态类型语言。在下面代码中同样定义 Desktop 和 Laptop 两个计算机类，并且定义一个工厂函数，这个工厂函数根据不同的业务条件来返回不同的计算机类。

```javascript
class Desktop {
  constructor() {}

  getContent(param) {
    return "Get content from Desktop: " + param;
  }
}

class Laptop {
  constructor() {}

  getContent(param) {
    return "Get content from Laptop: " + param;
  }
}

function getComputer(cond) {
  if (cond == 1) {
    return new Desktop();
  } else if (cond == 2) {
    return new Laptop();
  } else {
    throw new Error("Wrong cond...");
  }
}
```

由于 javascript 是鸭子类型的语言，变量类型是动态绑定的，computer 变量可以接受任何类型的值，因此不需要显示的使用接口来定义变量的类型，至于 computer 是否可以真正的调用 get_content 方法，是开发者在开发的时候就通过约定或协议声明好的。

```javascript
function main() {
  cond = 1;
  let c = getComputer(cond);
  let resStr = c.getContent("http://abc.com/");
  console.log(resStr);
}

main();
```

## 附录

#### 什么是鸭子类型

鸭子类型用来描述事物的外部行为而非内部结构。

比如有一只“橡皮鸭”，它是不是鸭子从使用者的角度来看，而不是定义者的角度。也就是说，从吃货角度看，这不是鸭子，因为橡皮鸭不能吃。但是从长相和行为角度，橡皮鸭长的像鸭子，并且能“游泳”，那么它就是鸭子。

#### 静态语言和动态语言

静态语言在编译期间进行类型检查，即编译器会检查代码中的类型错误，并在编译时发现并报告这些错误。这意味着在编译时就能够发现类型错误，而不需要在运行时进行类型检查。静态类型语言的优点是可以提前发现一些常见的编程错误，例如类型不匹配、变量未初始化等，并且可以在编译时进行优化，提高程序的性能。

动态语言在运行时进行类型检查，即在程序执行期间检查代码中的类型错误，并在运行时发现并报告这些错误。这意味着在运行时才能够发现类型错误，而不是在编译时。动态类型语言的优点是编写代码更加灵活、简单，因为不需要考虑类型检查的问题，可以更快地进行开发和测试。

## 参考文档：

N/A
