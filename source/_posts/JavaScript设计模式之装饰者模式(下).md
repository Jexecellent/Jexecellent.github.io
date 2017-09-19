---
title: JavaScript设计模式之装饰者模式(下)
tags: ['js','设计模式','decorator']
categories : JS
---

### AOP 的应用实例

用 AOP 装饰函数的技巧在实际开发中非常有用。不论是业务代码的编写，还是在框架层面，我们都可以把行为依照职责分成粒度更细的函数，随后通过装饰把它们合并到一起，这有助于我们编写一个`松耦合`和`高复用性`的系统。

<!-- more -->
下面介绍几个例子，带大家进一步理解装饰函数的威力。

#### 数据统计上报

分离业务代码和数据统计代码，无论在什么语言中，都是AOP的经典应用之一。在项目开发的结尾阶段难免要加上很多统计数据的代码，这些过程可能让我们被迫改动早已封装好的函数。

比如页面中有一个登录 button，点击这个button会弹出登录浮层，与此同时要进行数据上报，来统计有多少用户点击了这个登录 button：

```
<html>
 <button tag="login" id="button">点击打开登录浮层</button>
 <script>
    var showLogin = function(){
        console.log( '打开登录浮层' );
        log( this.getAttribute( 'tag' ) );
    }
    var log = function( tag ){
        console.log( '上报标签为: ' + tag );
        // (new Image).src = 'http:// xxx.com/report?tag=' + tag; // 真正的上报代码略
    }
    document.getElementById( 'button' ).onclick = showLogin;
 </script>
</html>
```

我们看到在 showLogin函数里，既要负责打开登录浮层，又要负责数据上报，这是两个层面的功能，在此处却被耦合在一个函数里。使用 AOP分离之后，代码如下：

```
<html>
<button tag="login" id="button">点击打开登录浮层</button>
<script>
    Function.prototype.after = function( afterfn ){
        var __self = this;
        return function(){
            var ret = __self.apply( this, arguments );
            afterfn.apply( this, arguments );
            return ret;
        }
    };
    var showLogin = function(){
        console.log( '打开登录浮层' );
    }
    var log = function(){
        console.log( '上报标签为: ' + this.getAttribute( 'tag' ) );
    }
    showLogin = showLogin.after( log ); // 打开登录浮层之后上报数据
    document.getElementById( 'button' ).onclick = showLogin;
</script>
</html> 
```


#### 用AOP动态改变函数的参数

观察 Function.prototype.before 方法：

```
Function.prototype.before = function( beforefn ){
    var __self = this;
    return function(){
        beforefn.apply( this, arguments ); // (1)
        return __self.apply( this, arguments ); // (2)
    }
} 
```

从这段代码的(1)处和(2)处可以看到，beforefn和原函数`__self`共用一组参数列表`arguments`，当我们在beforefn的函数体内改变arguments的时候，原函数`__self`接收的参数列表自然也会变化。

下面的例子展示了如何通过Function.prototype.before方法给函数func的参数param动态地添加属性b：

```
var func = function( param ){
    console.log( param ); // 输出： {a: "a", b: "b"}
}
func = func.before( function( param ){
    param.b = 'b';
});
func( {a: 'a'} ); 
```

现在有一个用于发起ajax请求的函数，这个函数负责项目中所有的ajax异步请求：

```
var ajax = function( type, url, param ){
 console.dir(param);
 // 发送 ajax 请求的代码略
};
ajax( 'get', 'http:// xxx.com/userinfo', { name: 'sven' } ); 
```

上面的伪代码表示向后台 cgi 发起一个请求来获取用户信息，传递给 cgi 的参数是
`{ name:'sven' }` 。

ajax 函数在项目中一直运转良好，跟 cgi 的合作也很愉快。直到有一天，我们的网站遭受了`CSRF`攻击。解决`CSRF`攻击最简单的一个办法就是在`HTTP`请求中带上一个`Token`参数。

假设我们已经有一个用于生成 Token 的函数：

```
var getToken = function(){
 return 'Token';
} 

```

现在的任务是给每个 ajax 请求都加上 Token 参数：

```
var ajax = function( type, url, param ){
 param = param || {};
 Param.Token = getToken(); // 发送 ajax 请求的代码略...
}; 
```

虽然已经解决了问题，但我们的 ajax 函数相对变得僵硬了，每个从 ajax 函数里发出的请求都自动带上了Token参数，虽然在现在的项目中没有什么问题，但如果将来把这个函数移植到其他项目上，或者把它放到一个开源库中供其他人使用，Token参数都将是多余的。

也许另一个项目不需要验证Token，或者是Token的生成方式不同，无论是哪种情况，都必须重新修改 ajax 函数。

为了解决这个问题，先把 ajax 函数还原成一个干净的函数：

```
var ajax= function( type, url, param ){
 console.log(param); // 发送 ajax 请求的代码略
}; 
```

然后把 Token 参数通过`Function.prototyte.before`装饰到`ajax`函数的参数`param`对象中：

```
var getToken = function(){
 return 'Token';
}
ajax = ajax.before(function( type, url, param ){
 param.Token = getToken();
});
ajax( 'get', 'http:// xxx.com/userinfo', { name: 'sven' } ); 

```

从 ajax 函数打印的 log 可以看到，Token 参数已经被附加到了 ajax 请求的参数中：

```
{name: "sven", Token: "Token"} 
```

明显可以看到，用AOP的方式给ajax函数动态装饰上Token参数，保证了ajax函数是一个相对纯净的函数，提高了ajax函数的可复用性，它在被迁往其他项目的时候，不需要做任何
修改。

#### 插件式的表单验证

我们很多人都写过许多表单验证的代码，在一个 Web 项目中，可能存在非常多的表单，如
注册、登录、修改用户信息等。在表单数据提交给后台之前，常常要做一些校验，比如登录的时候需要验证用户名和密码是否为空，代码如下：

```
<html>
<body>
    用户名：<input id="username" type="text"/> 
    密码： <input id="password" type="password"/>
    <input id="submitBtn" type="button" value="提交">
</body>
<script>
    var username = document.getElementById( 'username' ),
    password = document.getElementById( 'password' ),
    submitBtn = document.getElementById( 'submitBtn' );
    var formSubmit = function(){
        if ( username.value === '' ){
            return alert ( '用户名不能为空' );
        }
        if ( password.value === '' ){
            return alert ( '密码不能为空' );
        }
        var param = {
            username: username.value,
            password: password.value
        }
        ajax( 'http:// xxx.com/login', param ); // ajax 具体实现略
    }
    submitBtn.onclick = function(){
        formSubmit();
    }
</script>
</html> 
```

formSubmit函数在此处承担了两个职责，除了提交ajax请求之外，还要验证用户输入的合法性。这种代码一来会造成函数臃肿，职责混乱，二来谈不上任何可复用性。

我们的目的是分离校验输入和提交 ajax请求的代码，我们把校验输入的逻辑放到validata
函数中，并且约定当 validata 函数返回 false 的时候，表示校验未通过，代码如下：

```
var validata = function(){
    if ( username.value === '' ){
        alert ( '用户名不能为空' );
        return false;
    }
    if ( password.value === '' ){
        alert ( '密码不能为空' );
        return false;
    }
}
var formSubmit = function(){
    if ( validata() === false ){ // 校验未通过
        return;
    }
    var param = {
        username: username.value,
        password: password.value
    }
    ajax( 'http:// xxx.com/login', param );
}
submitBtn.onclick = function(){
    formSubmit();
} 

```

现在的代码已经有了一些改进，我们把校验的逻辑都放到了validata函数中，但formSubmit函数的内部还要计算 validata 函数的返回值，因为返回值的结果表明了是否通过校验。

接下来进一步优化这段代码，使validata和formSubmit完全分离开来。首先要改写Function.prototype.before，如果beforefn的执行结果返回false，表示不再执行后面的原函数，代码如下：

```
Function.prototype.before = function( beforefn ){
    var __self = this;
    return function(){
        if ( beforefn.apply( this, arguments ) === false ){
            // beforefn 返回 false 的情况直接 return，不再执行后面的原函数
            return;
        }
        return __self.apply( this, arguments );
    }
}
var validata = function(){
    if ( username.value === '' ){
        alert ( '用户名不能为空' );
        return false;
    }
    if ( password.value === '' ){
        alert ( '密码不能为空' );
        return false;
    }
}
var formSubmit = function(){
    var param = {
        username: username.value,
        password: password.value
    }
    ajax( 'http:// xxx.com/login', param );
}
formSubmit = formSubmit.before( validata );
submitBtn.onclick = function(){
    formSubmit();
} 

```

在这段代码中，校验输入和提交表单的代码完全分离开来，它们不再有任何耦合关系，
`formSubmit = formSubmit.before( validata )`这句代码，如同把校验规则动态接在 formSubmit 函数之前，validata成为一个即插即用的函数，它甚至可以被写成配置文件的形式，这有利于我们分开维护这两个函数。再利用策略模式稍加改造，我们就可以把这些校验规则都写成插件的形式，用在不同的项目当中。

值得注意的是，因为函数通过 `Function.prototype.before` 或者 `Function.prototype.after` 被装饰之后，返回的实际上是一个新的函数，如果在原函数上保存了一些属性，那么这些属性会丢失。代码如下：

```
var func = function(){
 alert( 1 );
}
func.a = 'a';
func = func.after( function(){
 alert( 2 );
});
alert ( func.a ); // 输出：undefined 
```

另外，这种装饰方式也叠加了函数的作用域，如果装饰的链条过长，性能上也会受到一些
影响。

### 小结

通过数据上报、统计函数的执行时间、动态改变函数参数以及插件式的表单验证这4个例子，我们了解了装饰函数，它是JavaScript中独特的装饰者模式。这种模式在实际开发中非常有用，除了上面提到的例子，它在框架开发中也十分有用。作为框架作者，我们希望框架里的函数提供的是一些稳定而方便移植的功能，那些个性化的功能可以在框架之外动态装饰上去，这可以避免为了让框架拥有更多的功能，而去使用一些if、else语句预测用户的实际需要。 