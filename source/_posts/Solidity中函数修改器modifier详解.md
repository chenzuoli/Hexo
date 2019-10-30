---
title: Solidity中函数修改器modifier详解
date: 2019-10-30 21:08:32
tags: [智能合约,solidity,modifier]
categories: 区块链
notebook: 区块链
---

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`modifier`即函数的修改器，可以用来改变一个函数的行为，控制函数的逻辑。修改器是一种合约属性，可以被继承和重写。

<img src="Solidity中函数修改器modifier详解/modifier.jpeg" width="500" height="300"/>

<!-- more -->

# 1.函数修改器

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;下面以代码为例进行介绍（代码来源于CryptoKitties项目KittyAccessControl.sol合约，详细代码可以查看<b><a>[<font color=#0099ff>https://github.com/dapperlabs/cryptokitties-bounty</font>](https://github.com/dapperlabs/cryptokitties-bounty)</b>)
```
modifier onlyCLevel() {
    require(
        msg.sender == cooAddress ||
        msg.sender == ceoAddress ||
        msg.sender == cfoAddress
    );
    _;
}
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;前面一段代码是一个修改器，声明了一个约束`onlyClevel`：仅当当前的地址为`ceoAddress`或者`cfoAddress`或者`cooAddress`时，可以执行后续代码。下划线_是一个占位符，代表了执行函数接下来的代码。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;有时你还会看到上面那段代码写成如下形式：
```
modifier onlyCLevel() {
    if(
        msg.sender != cooAddress &&
        msg.sender != ceoAddress &&
        msg.sender != cfoAddress
    ) throw;
    _;
}
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;其实两段代码是等价的，`if() throw`的写法是较为老式的写法，现在使用`require()`的写法较多。
```
function pause() external onlyClevel whenNotPaused {
        paused = true;
    }
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;接下来一段代码声明了一个函数`pause()`，用于暂停合约，这里使用了`onlyClevel`约束，表明该函数的执行必须要满足`onlyClevel`条件。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;此外函数修改器也支持传入参数，和函数的参数类似，例如：
```
pragma solidity ^0.4.0;

contract parameter{
    uint balance = 10;

    modifier lowerLimit(uint _balance, uint __withdraw){
        if( _withdraw < 0 || _withdraw > _balance) throw;
        _;
    }

    function f(uint withdraw) lowerLimit(balance, withdraw) returns (uint){
        return balance;
    }
}
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在上面这段代码中，修改器`lowerLimit`传入两个参数，执行修改器的逻辑。函数的执行与否取决于两个参数：`_withdraw`和`_balance`。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;函数的修改器参数支持表达式，例如：
```
pragma solidity ^0.4.0;

contract parameterExpression{
    modifier m(uint a){
        if(a > 0)
            _;
    }

    function add(uint a, uint b) private returns(uint){
        return a + b;
    }

    function f() m(add(1, 1)) returns(uint){
        return 1;
    }
}
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;对于上面的这段代码，修改器m的参数传入了一个表达式：`add(a+b)`，`add()`表达式在合约中定义了。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Return用在函数中表示返回值，如果函数有返回值标志`return`，但是由于修改器限制，判断不成功，无法执行函数体内的代码，那么将会返回返回值类型的默认值，例如：
```
pragma solidity ^0.4.0;

contract Return{
    modifier a(){
        if(false)
            _;
    }

    function uintReturn() A returns(uint){
        uint a = 0;
        return uint(1);
    }

    function stringReturn() A returns(string){
        return "Hello World";
    }
}
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;对于上面的代码，由于修改器A永远判断不成功，所以`uintReturn`和`stringReturn`两个函数的函数体永远无法执行，那么返回值分别是`uint`和`string`的默认值：0和空串。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;函数修改器允许使用`return`;来中断当前流程，但是不允许明确的`return`值，也就是说在修改器中只能存在`return`；
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;对于函数修改器，<b>下划线代表函数体</b>，当执行到下划线;这一行的时候，就跳到函数体，执行函数体内的语句，执行完函数体内语句其实还要回到函数修改器，执行下划线`_`;后面的语句，例如：
```
pragma solidity ^0.4.0;

contract processFlow{
    mapping(bool => uint) public mapp;

    modifier A(mapping(bool => uint) mapp){
        if(mapp[true] == 0){
            mapp[true] = 1;
            _;
            mapp[true] = 3;
        }
    }

    function f() A(mapp) returns(uint){
        mapp[true] = 2;
        return mapp[true];
    }
}
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;对于上面的函数`f()`，先运行修改器，判断权限，通过权限，则执行修改器判断语句后面的代码：`map[true] = 1;`，当运行到下划线;这一行的时候，跳到函数体内执行`map[true] = 2;`，然后`retrun map[true]`的值。执行完函数体的代码会回到函数修改器下划线；这一行后面的代码，这里还有代码，接着执行`map[true] = 3`。所以最终`map[true]`的值为3。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;对于一个函数可以有多个修改器限制，在函数定义的时候依次写上，并用加空格分隔，执行的时候也是依次执行。多个修改器是同时限制，也就是说必须满足所有修改器的权限，才可以执行函数体的代码，例如：
```
pragma solidity ^0.4.0;

contract multiModifier{
    address owner = msg.sender;

    modifier onlyOwner{
        if(msg.sender != owner) throw;
        _;
    }

    modifier inState(bool state){
        if(!state) throw;
        _;
    }

    function f(bool state) onlyOwner inState(state) returns(uint){
        return 1;
    }
}
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;上面这段代码中两个修改器`onlyOwner`和`inState`同时作用于函数`f()`，只有当两个修改器的权限同时满足的时候，才会执行函数体内的代码，`retrun 1`。

# 2.重写修改器

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;我们知道合约是可以继承的，在子合约中我们还可以对父合约中的修改器进行重写覆盖，例如：
```
pragma solidity ^0.4.0;

contract bank{
    modifier transferLimit(uint _withdraw){
        if(_withdraw > 100) throw;
        _;
    }
}

contract ModifierOverride is bank{
    modifier transferLimit(uint _withdraw){
        if(_withdraw > 10) throw;
        _;
    }

    function f(uint withdraw) transferLimit(withdraw) returns(uint){
        return withdraw;
    }
}
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;上面这段代码中，子合约`ModifierOverride`继承了父合约`bank`，那么同样有父合约中的`transferLimit`修改器，之后在子合约中再次定义`transferLimit`修改器，就重写了该修改器并覆盖了原修改器，在子合约中`transferLimit`的限制条件由子合约中重写的条件决定。



- - -
<b>Never regret a day in your life, good days give you happiness, bad days give you experiences.</b>