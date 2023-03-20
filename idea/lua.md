# lua

lua模块

lua环境的初始化，管理lua虚拟机的生命周期，负责unity与lua环境的交互

读取Main.lua文件代码，获取Main函数并调用
Dispose 引用计数减一，当引用为零时销毁对象

网络模块，底层使用tcp连接，负责管理tcp连接的生命周期

# 调试lua

## "ideConnectDebugger": 

false 对应 tcpConnect

true 对应 tcpListen

建议使用false 先打开ide调试再启动unity, 但是在不使用时需要手动关闭否则会报错

# lua class

特性：

支持继承及多重继承

支持类型检测

支持构造函数

符合面向对象的的编程思维模式

继承关系复杂不影响性能，因为做了动态扩展， 但是因此失去了部分写法的reload特性。 

# Import.lua

在该文件中导入其他lua代码文件到框架中

require会先在package的loaded表中查找是否已经导入该包，否则使用lua加载器->c加载器->loader一体化加载器加载路径中的代码，成功找到后将其的引用赋值给package中的loaded；
因此此处将loaded表清空是由于require当loeaded中有表时每次都会返回同一个表，清空是为了使其返回的表不同

## package

cpath C加载器路径

path lua加载器

preload 特殊模块装载器？

# module

1.在使用了module函数的脚本，使用require并不能返回一个table，而是一个bool值，这个值告诉你是否加载成功

2.require一个使用了module函数的脚本，结果会被存在_G的全局表里，所以我们可以使用JCTest2:Func1（）去调用函数

3.这个方式的require的结果也会在package.loaded这个table中存放，输出的100,证明了他的唯一性

4.如果在JCTest2.lua 里的函数前面加上 local，则无法访问到该函数（变量也一样，当然lua里函数也是变量）

5.虽然Func1并没有显示指明他的所属关系，但它并非是一个全局函数(重要！！！)

module函数的作用是创造出一个新的“环境”，在这个模块的所有全局函数都只属于这个环境，如果外部需要调用，我们只能需要使用论点2的方式。而如果，在模块中使用了local 定义变量，很抱歉，这个变量将不能被外部调用

package.seeall的作用就是在新环境中，可以看到先前的环境。而先前的环境要看到这个新环境，必须使用require获取新环境的table，然后访问新环境中的全局变量


# CustomSetting

用于存贮需要生成与lua进行交互的warp类的配置表

# luaFunction

用于将lua环境中的function转换为C#环境中的委托的中间类

# C# 委托与 lua 环境的委托的连接过程

CheckDelegate函数负责将 luaFunction 转换为对应的委托类型

# UnityAction 转换过程

```c#
static int UnityEngine_Events_UnityAction(IntPtr L)
{
	try
	{
		int count = LuaDLL.lua_gettop(L);
		LuaFunction func = ToLua.CheckLuaFunction(L, 1);
			if (count == 1)
		{
			Delegate arg1 = DelegateTraits<UnityEngine.Events.UnityAction>.Create(func);
			ToLua.Push(L, arg1);
		}
		else
		{
			LuaTable self = ToLua.CheckLuaTable(L, 2);
			Delegate arg1 = DelegateTraits<UnityEngine.Events.UnityAction>.Create(func, self);
			ToLua.Push(L, arg1);
		}
		return 1;
	}
	catch(Exception e)
	{
		return LuaDLL.toluaL_exception(L, e);
	}
}
```

判断栈中元素个数再根据数量构建全局函数的委托或者是表的委托

# UnityEvent 添加回调过程

```c#
static int AddListener(IntPtr L)
{
	try
	{
		ToLua.CheckArgsCount(L, 2);
		UnityEngine.Events.UnityEvent obj = (UnityEngine.Events.UnityEvent)ToLua.heckObject<UnityEngine.Events.UnityEvent>(L, 1);
		UnityEngine.Events.UnityAction arg0 = (UnityEngine.Events.UnityAction)ToLua.heckDelegate<UnityEngine.Events.UnityAction>(L, 2);
		obj.AddListener(arg0);
		return 0;
	}
	catch (Exception e)
	{
		return LuaDLL.toluaL_exception(L, e);
	}
}
```

在向UnityEvent中添加回调时，不会预先检测栈中元素的数量，因此无法区分全局函数还是类中的函数。
因此在lua中使用时需要先创建UnityAction，在将对应的UnityAction加入到UnityEvent中

这里可以通过UnityAction中一样的方式进行转换从而减少环境交互的次数从而提高效率

由于tolua 文件生成的原因，因此使用拓展方法进行实现。。。

# 闭包

lua中的闭包简而言之就是利用了lua中变量的持久性以及访问域，限制局部闭包中的变量只能够由其导出的变量访问

# _G表污染

由于当_G表中存储的数据量过大时，其会频繁发生哈希冲突从而导致其查询速度显著受到影响，因此最好将变量都声明为局部变量，除非必要，通过require出来的模块都建议使用local对象保存

# lua与c#的交互

由于c#生成warp类会污染_G表，因此不建议过多的使用warp，如要访问c#中的代码，建议使用sendMessage访问，为此需要在c#端提前为访问提供对应的接口。

由于在实际项目中，lua的代码是灵活多变的，因此不建议在c#中调用lua代码，增加lua端的负担，在项目中应尽可能保持数据从lua端到c#端的单项流通

# _ENV

5.1没有_ENV表的访问权，但是可以使用setgenv 函数在5.1用来设置_ENV

5.1以后删除了该函数，增加了_ENV表的访问权

两者相互不兼容。。。。

# protobuf 

可以通过tostring查看里边的内容

# DelegateFactory

lua 的委托在C#的创建由这个类负责，在创建前需要先在其的萃取列表中进行登记，由于涉及warp的原因，因此只能够使用warp进行登记。。。

# coroutine

lua 原生的协程不能够和Unity中的协程一样由系统进行管理自动在需要的时候进行启动。
tolua 中实现与unity同步的协程的方式有两种，第一种是基于luamodule的update实现，在某个协程管理器的update函数中遍历协程；第二种是基于monobahvior实现，将lua函数转换为委托到monobahvior的协程函数执行完毕后进行调用，这种方式效率极低。