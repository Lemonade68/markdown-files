# { HiveConfig } 用户使用说明文档

***

[TOC]



***

## 整体说明

​		**hiveConfig命名空间**是用于封装与配置(config)相关的所有操作的一个工具命名空间。在该命名空间下，最常使用的功能为：**普适的配置设置**、**从xml文件中载入配置**，进而可以进一步搭建场景等。

### CHiveConfig

​		该命名空间中最为重要的一个类为`CHiveConfig`类，该类是后面我们要进行所有操作的前提。下面给出该类的声明以及常用的方法：

```c++
// namespace hiveConfig
	class CONFIG_DECLSPEC CHiveConfig : public hiveCommon::IProduct  
	{
	public:
		CHiveConfig();
		~CHiveConfig() {}

		EConfigValidity validate() const;

		bool isAttributeExisted(const std::string& vAttributeName) const;
		bool setAttribute(const std::string& vAttributeName, std::any vAttributeValue);  //this function cannot overwrite existed non-default attribute value 
		bool overwriteAttribute(const std::string& vAttributeName, std::any vAttributeValue);
		bool appendString(const std::string& vAttributeName, const std::string& vAppendValue);
		bool isAnonSubconfig() const noexcept;

		void setAllowAutonomousAttributeHint() noexcept;
		void setName(const std::string& vName);
		void loadDefaultConfig();
		void extractSpecifiedSubconfigsRecursively(const std::string& vSubconfigTypeName, std::vector<CHiveConfig*>& vioSubconfigSet) const noexcept;
		void clearContent();

		const std::string& getName() const noexcept;
		const std::string& getSubconfigType() const noexcept;
		const CHiveConfig* getParentConfig() const noexcept;
		const CHiveConfig* getSubconfigAt(unsigned int vIndex) const;
		const CHiveConfig* getNextSubconfig() const noexcept;

		CHiveConfig* findSubconfigByName(const std::string& vConfigName) const;
		CHiveConfig* fetchSubconfigAt(unsigned int vIndex) const;
		CHiveConfig* fetchSpecifiedSubconfigAt(const std::string& vSubconfigTypeName, unsigned int vIndex) const;
		
		[[nodiscard]] CHiveConfig* addSubconfig(const CHiveConfig *vConfig);

		std::size_t getNumSubconfigs() const noexcept;
		std::size_t getNumAttributes() const noexcept;
		std::size_t getNumSpecifiedSubconfigs(const std::string& vSubconfigTypeName) const noexcept;
		std::size_t getNumSpecifiedSubconfigsRecursively(const std::string& vSubconfigTypeName) const noexcept;

		std::pair<std::string, std::any> getAttributeAt(std::size_t vIndex) const;

		std::string generateConfigPathString() const;

		EConfigDataType getDataType(const std::string& vAttributeName) const noexcept;

		template<class T>
		std::optional<T> getAttribute(const std::string& vAttributeName) const;
		
        // ...
	};
```



### CStaticOffspringConfigIterator

​		`CStaticOffspringConfigIterator`类是为了方便对一个配置对象的子配置进行管理而诞生的，可以视为一个链表上的一个指针，每次可以获取下一个子配置对象。

```c++
class CStaticOffspringConfigIterator
{
public:
	CStaticOffspringConfigIterator(void) = default;
	~CStaticOffspringConfigIterator(void) = default;

	CHiveConfig* fetchFirstOffspringConfig(const CHiveConfig* vRootConfig);
	CHiveConfig* fetchFirstOffspringConfig(const CHiveConfig* vRootConfig, const std::string& vSubconfigTypeName);
	CHiveConfig* fetchNextOffspringConfig();
	//...
}
```

​		其中，`static`的含义为：在第一次使用`fetchFirstOffspringConfig`方法后，子配置的列表就已经固定了，此时如果再次加入新的子配置，则不能被`fetchNextOffspringConfig`获取。（如果需要使用新的子配置的话则需要重新使用`fetchFirstOffspringConfig`）



* 下面将结合上述的代码，对提到的两大点功能进行简单的使用说明。



***

## 普适的配置设置

### 自定义配置的创建

​		为了构建自定义的configuration，首先我们需要创建一个继承于`CHiveConfig`的子类，假设子类名为`CMyConfiguration`。

```c++
class CMyConfiguration : public hiveConfig::CHiveConfig
{
public:
	CMyConfiguration();
	~CMyConfiguration() = default;

private:
	void __defineAttributesV() override;
	void __loadDefaultConfigV() override;
};
```

​		`CMyConfiguration`的构造函数举例如下：

```c++
CMyConfiguration::CMyConfiguration()
{
  _overwriteProductSig("MY_CONFIG");
  __defineAttributesV();
}
```
​		在上述构造函数中，我们需要使用继承自`IProduct`类的`_overwriteProductSig`函数给我们的配置起一个签名。另外还要重写继承自`CHiveConfig`的`__defineAttributesV()`函数，该函数用于将属性名称和对应的数据类型关联起来。举一个简单的例子：

```c++
void CMyConfiguration::__defineAttributesV()
{
  _defineAttribute("INT",    hiveConfig::EConfigDataType::ATTRIBUTE_INT);
  _defineAttribute("VECSTRING", hiveConfig::EConfigDataType::ATTRIBUTE_VECSTRING);
  //...
}
```



### 自定义配置的属性

​		这里假设已经创建了一个自定义配置的对象`Config`。

#### 设置配置的属性

​		使用继承自`CHiveConfig`的`setAttribute`函数给对应的属性设置值：

```c++
CMyConfiguration Config;
Config.setAttribute("INT", 4);
```


#### 获取配置的属性

​		使用`getAttribute`模板函数得到对应属性的值：

```c++
auto t0 = Config.getAttribute<int>("INT").value();
```


#### 修改配置的属性

​		使用`overwriteAttribute`函数（不可以使用`setAttribute`函数进行修改），注意覆盖时新属性值的数据类型必须和属性数据类型严格一致，即使二者数据类型存在合理的数据转换也不行：

```c++
Config.overwriteAttribute("INT", 5)
```
​		注：当属性的值的类型为字符串数组时，允许使用`appendString`函数给属性的值添加额外的字符串：

```c++
std::vector<std::string> StringVec = { "str1", " str2", " s t r 3 " };

Config.setAttribute("VECSTRING", StringVec);
Config.appendString("VECSTRING", std::string("append"));
```


#### 默认属性的加载

​		当我们的配置有默认属性时，可以重载继承自`CHiveConfig`的`__loadDefaultConfigV`函数，并在创建`CMyConfiguration`对象后，直接加载默认配置：

```c++
// 在CMyConfiguration.h中添加
class CMyConfiguration : public hiveConfig::CHiveConfig
{
    //...
	void __loadDefaultConfigV() override;
    //...
}

// 在CMyConfiguration.cpp中添加
void CMyConfiguration::__loadDefaultConfigV()
{
    // 默认属性
	setAttribute("INT", 10);
    //...
}

// 然后就可以直接加载默认配置
CMyConfiguration Config;
Config.loadDefaultConfig();
// 并且默认值可以直接用setAttribute来进行修改，之后需使用overwriteAttribute来进行修改
Config.setAttribute("INT", 50);
Config.overwriteAttribute("INT", 100);
```





### 自定义配置的层级结构

​		在自定义配置中，好的层级结构可以用来更好地对配置的各个部分进行管理，因此有了父配置和子配置的说法。

#### 子配置对象的声明与添加

```c++
CMyConfiguration m_Subconfig1;
// 使用setName函数给子配置设置一个名称
m_Subconfig1.setName("Subconfig1");
// 使用addSubconfig函数绑定子配置，在这之前必须要给子配置起一个名称，返回的都是CHiveConfig对象，因此后面类似的都要用dynamic_cast转成自定义的子类型
CMyConfiguration *pSubconfig = dynamic_cast<CMyConfiguration*>(m_Config.addSubconfig(&m_Subconfig1));
```
​				注1：得到的`pSubconfig`并不指向`m_Subconfig1`，但是会拷贝`m_Subconfig1`的内容。

​				注2：当使用`overwriteAttribute`函数修改`m_Subconfig1`对象时，不会影响到`m_Config`的子配置。



#### 子配置对象的获取

```c++
// 绑定完成后，可以使用getSubconfigAt函数来获取指定索引的子配置指针，它的地址与pSubconfig相同
pSubconfig == m_Config.getSubconfigAt(0);  // True
// 获取父配置的子配置数量
size_t num = m_Config.getNumSubconfigs();
// 通过子配置获取父配置
auto pParentConfig = dynamic_cast<CMyConfiguration*>(pSubconfig->getParentConfig());
```


#### 复数子配置对象的设置

​		对于一个父配置，可以设置多个子配置：

```c++
CMyConfiguration m_Subconfig2;
m_Subconfig2.setAttribute("VEC2F", std::make_tuple(2.0, -5.0));
m_Subconfig2.setName("subconfig2");
pSubconfig = dynamic_cast<CMyConfiguration*>(m_Config.addSubconfig(&m_Subconfig2));
pSubconfig == m_Config.getSubconfigAt(1) // True
```
​		另外，可以直接从父配置中直接获取子配置的属性值，形式为：在父配置中调用`getAttribute`方法，并通过“子配置名|属性名”的格式来获取具体的子配置属性：
```c++
std::tuple<double, double> t4 = m_Config.getAttribute<std::tuple<double, double>>("Subconfig2|VEC2F").value();
```
​		注意：同级子配置不能同名，如果新设置了同名的子配置，`addSubconfig`函数将会返回`nullptr`



#### 多级子配置对象的设置

​		我们还可以为父配置嵌套添加多级子配置：

```c++
m_Subconfig1.setName("Subconfig1");
CMyConfiguration *pSubconfig = dynamic_cast<CMyConfiguration*>(m_Config.addSubconfig(&m_Subconfig1));

m_Subconfig2.setName("subconfig2");
pSubconfig = dynamic_cast<CMyConfiguration*>(pSubconfig->addSubconfig(&m_Subconfig2));
```
​		也可以从父配置中直接获取多级子配置的属性值，形式改为“子配置名|子子配置名|...|属性名”：
```c++
std::tuple<double, double> t4 = m_Config.getAttribute<std::tuple<double, double>>("Subconfig1|Subconfig2|VEC2F").value();
```



#### 子配置对象的名称查找与遍历

​		我们可以通过使用`findSubconfigByName`函数来通过名字得到子配置：

```c++
m_Subconfig1.setName("Subconfig1");
CMyConfiguration *pSubconfig = dynamic_cast<CMyConfiguration*>(m_Config.addSubconfig(&m_Subconfig1));
m_Subconfig2.setName("subconfig2");
pSubconfig = dynamic_cast<CMyConfiguration*>(m_Config.addSubconfig(&m_Subconfig2));
m_Subconfig3.setName("Subconfig3");
pSubconfig = dynamic_cast<CMyConfiguration*>(m_Config.addSubconfig(&m_Subconfig3));

pSubconfig = dynamic_cast<CMyConfiguration*>(m_Config.findSubconfigByName("Subconfig1"));
```

​		为了方便，我们还可以使用迭代器遍历所有的子配置：
```c++

m_Subconfig1.setName("Subconfig1");
CMyConfiguration *pSubconfig1 = dynamic_cast<CMyConfiguration*>(m_Config.addSubconfig(&m_Subconfig1));
m_Subconfig2.setName("subconfig2");
CMyConfiguration *pSubconfig2 = dynamic_cast<CMyConfiguration*>(pSubconfig1->addSubconfig(&m_Subconfig2));
m_Subconfig3.setName("Subconfig3");
CMyConfiguration *pSubconfig3 = dynamic_cast<CMyConfiguration*>(pSubconfig2->addSubconfig(&m_Subconfig3));
m_Subconfig4.setName("Subconfig4");
CMyConfiguration *pSubconfig4 = dynamic_cast<CMyConfiguration*>(m_Config.addSubconfig(&m_Subconfig4));

hiveConfig::CStaticOffspringConfigIterator itr;
CMyConfiguration *pOffspringConfig = dynamic_cast<CMyConfiguration*>(itr.fetchFirstOffspringConfig(&m_Config));
```
​		在`fetchFirstOffspringConfig`函数中，我们使用队列实现了一个层序遍历，将所有的子配置存放在了`m_OffspringConfigSet`中。所以，在使用`fetchNextOffspringConfig`函数进行遍历时，得到的子配置的顺序为：1->4->2->3



***

## 从XML文件中读取配置

​		每次都从头进行配置的手动设置太繁琐了，因此我们可以选择使用形式更简单的诸如xml，json的文件来记录配置的所有信息，然后将其导入到我们自己的配置类中。这里以xml为例：

### hiveParseConfig：读取配置的核心方法

​		从xml文件中读取配置的核心函数的声明如下：

```c++
EParseResult hiveConfig::hiveParseConfig(const std::string& vConfigFileName, EConfigType vConfigType, CHiveConfig* voConfig);
```

其中：

* `EParseResult`有`SUCCEED`，`SKIP_SOME_ITEMS`，`ONLY_LOAD_DEFAULT_CONFIG`，`FAIL`四个状态，对应不同的构造结果
* `vConfigType`有`XML`，`INI`，`JSON`，`INFO`四个状态，其中xml是默认值
* `voConfig`在函数使用后会被更新为文件中预定的配置状态
* `vConfigFileName`为要装载的xml文件的名字，xml文件可以放置在以下位置：
  * 当前运行程序的同级目录
  * 当前运行程序的上一级目录
  * 使用`hiveUtility`命名空间下的`CFileLocator`中`addFileSearchPath`中添加过的路径

经过上述配置后，即可将`voConfig`视为自己已经配置好了的对象，从而使用第一节中的操作。



### 读取元素中心结构的xml文件

```xml
<!-- 元素中心结构的xml示例 -->
<STRING> sdF23f3f </STRING>
<FLOAT> 2.343 </FLOAT>
<DOUBLE> 2.343 </DOUBLE>

<Subconfig>
  Subconfig24
  <VEC2I> 2 3 </VEC2I>
  <VECSTRING> sdfa </VECSTRING>
  <Subconfig>
    SubConfig100
  </Subconfig>
</Subconfig>

<Subconfig>
  Subconfig22
  <CONFIG_OBJECT_SIGNATURE>MY_CONFIG</CONFIG_OBJECT_SIGNATURE>
  <VEC4I>3 4 6 -87 </VEC4I>
  <VEC2F>3 34.3 </VEC2F>
  <Subconfig> 
    Subconfig3
    <BOOL> TRUE </BOOL>
  </Subconfig>
  <AnotherSubconfig>
    SubConfig4
    <DOUBLE> 3 </DOUBLE>
  </AnotherSubconfig>
</Subconfig>

```

**在上面的xml文件解析后，对于常规操作的结果，由下面的代码中注释给出：**

```c++
// 1. 构造完成后获取属性示例
Config.getAttribute<std::string>("STRING").value();  //值为: "sdF23f3f"
Config.getAttribute<float>("FLOAT").value();   		//值为：2.343 ± 浮点数允许的误差范围
Config.getAttribute<double>("DOUBLE").value();	    //同上

Config.getNumSubconfigs();  //值为：2，该方法只会考虑直接的子配置个数

// 2. 通过索引来找第一个子配置subConfig24
// 注意subconfig24的类型为CHiveConfig，而不是CMyConfiguration，原因见subconfig22那里
const hiveConfig::CHiveConfig *pSubconfig = Config.getSubconfigAt(0);
pSubconfig->getName();     //值为："Subconfig24"
pSubconfig->getAttribute<std::tuple<int, int>>("VEC2I").value();  //值为：{2, 3}

// 3. 通过名字来找第二个子配置subConfig22
// 注意subConfig22的类型为CHiveConfig，因为xml中给定了"MY_CONFIG"的签名，而24没有
pSubconfig = Config.findSubconfigByName("subconfig22");

// 4. 下面展示是否递归搜索的区别：递归搜索后，子配置的子配置也会被算入其中
Config.getNumSpecifiedSubconfigs("Subconfig");  //值为：2
Config.getNumSpecifiedSubconfigsRecursively("Subconfig");   //值为：4

// 5. 递归获取子配置信息时，是通过深度优先的方式，而非层级的方式
std::vector<hiveConfig::CHiveConfig*> SubconfigSet;  
Config.extractSpecifiedSubconfigsRecursively("subconfig", SubconfigSet);
SubconfigSet.size();         //值为：4
SubconfigSet[0]->getName();  //值为："Subconfig24"
SubconfigSet[1]->getName();  //值为："SubConfig100"
SubconfigSet[2]->getName();  //值为："Subconfig22"
SubconfigSet[3]->getName();  //值为："Subconfig3"
```

**注意事项：对于有误的xml文档，错误的属性将不会被读入。**



### 读取属性中心结构的xml文件

```xml
<!-- 属性中心结构的xml示例 -->
<?xml version="1.0" encoding="utf-8"?>
<Subconfig>
  <IF class="IfConfigClass" enabled="true" integer_value="2" float_value="3.3" strvec_value="12, sdf, Sf">
    <VEC2I> 2 3</VEC2I>
  </IF>
</Subconfig>
```

**在上面的xml文件解析后，对于常规操作的结果，由下面的代码中注释给出：**

```c++
const hiveConfig::CHiveConfig* pConfig = Config.getSubconfigAt(0);       // subconfig
const hiveConfig::CHiveConfig* pSubconfig = pConfig->getSubconfigAt(0);  // IF

pSubconfig->getAttribute<std::string>("class").value();  //值为："IfConfigClass");
pSubconfig->getAttribute<bool>("enabled").value());      //值为：true
pSubconfig->getAttribute<int>("integer_value").value();  //值为：2
pSubconfig->getAttribute<float>("float_value").value();  //值为：3.3 ± 浮点数误差
std::vector<std::string> t = pSubconfig->getAttribute<std::vector<std::string>>("strvec_value").value();
// t[0] = std::string("12")
// t[1] = std::string("sdf")
// t[2] = std::string("Sf")

std::tuple<int, int> t1 = pSubconfig->getAttribute<std::tuple<int, int>>("VEC2I").value();  //值为{2, 3}

```

