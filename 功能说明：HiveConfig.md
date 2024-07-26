**文档说明：**`HiveConfig`命名空间中包括四个类、四个枚举以及一个对外接口，本文为功能文档，会对每个类、每个枚举以及接口进行内部的描述，涵盖成员、方法等，可以使用 Ctrl + F 来快速索引到想要找的内容。

***

[TOC]

---

# 四个类
## CHiveConfig
### 公有成员函数

```c++
EConfigValidity validate() const;
```

**作用：**验证配置的有效性，返回 `EConfigValidity` 枚举值，指示配置是否有效。  

```c++
bool isAttributeExisted(const std::string& vAttributeName) const;
```

**作用：**检查指定属性是否存在。  

```c++
bool setAttribute(const std::string& vAttributeName, std::any vAttributeValue);
```

**作用：**设置指定属性的值，**注意：**

* 不能覆盖已有的**非默认**属性值；
* 当没有使用过`setAllowAutonomousAttributeHint()`方法时，必须设置的是配置类中使用过`_defineAttribute()`定义的能够接收的属性（详情见下面`setAllowAutonomousAttributeHint()`）。

```c++
bool overwriteAttribute(const std::string& vAttributeName, std::any vAttributeValue);
```

**作用：**显式覆盖指定已经set过的属性的值。  

```c++
bool appendString(const std::string& vAttributeName, const std::string& vAppendValue);
```

**作用：**将字符串追加到指定属性的值，注意修改的属性一定要是**VECSTRING类型**。  

```c++
bool isAnonSubconfig() const noexcept;
```

**作用：**检查是否为匿名子配置。  

```c++
void setAllowAutonomousAttributeHint() noexcept;
```

**作用：**使得自定义配置允许使用匿名属性（即配置中没有预定义过的属性类型）

* 默认情况下只允许用户使用`_defineAttribute()`(可以在自定义配置的`__defineAttributesV()` 方法中) 来确定配置可以定义的属性，而不能直接使用`setAttribute`等方法来直接设置没有预定义过的属性名称。例如：如果在`_defineAttribute()`中没有定义`INT`属性，就不能直接使用`setAttribute`来设置一个新的`INT`的值。

* 而当使用过这个函数时，就可以使用上面的情况。

```c++
void setName(const std::string& vName);
```

**作用：**设置配置的名称。

```c++
void loadDefaultConfig();
```

**作用：**加载默认配置。

```c++
void extractSpecifiedSubconfigsRecursively(const std::string& vSubconfigTypeName, std::vector<CHiveConfig*>& vioSubconfigSet) const noexcept;
```

**作用：**递归提取指定类型的子配置，由于是递归，因此可以提取子配置的子配置，以及类似情况。

```c++
void clearContent();
```

 **作用：**清空配置内容，包括父亲配置指针，子配置表，属性，名称。

```c++
const std::string& getName() const noexcept;
```

 **作用：**获取配置名称。

```c++
const std::string& getSubconfigType() const noexcept;
```

**作用：**获取子配置类型名称。

```c++
const CHiveConfig* getParentConfig() const noexcept;
```

**作用：**获取父配置指针。

```c++
const CHiveConfig* getNextSubconfig() const noexcept;
```

**作用：**获取下一个子配置，按照层级的顺序。

```c++
CHiveConfig* findSubconfigByName(const std::string& vConfigName) const;
```

**作用：**通过名称来查找子配置，并返回该子配置的指针。

```c++
const CHiveConfig* getSubconfigAt(unsigned int vIndex) const;
```

**作用：**获取指定索引处的子配置，索引下标从0开始，返回常量指针（不能修改`CHiveConfig`或自定义配置的内容）。

```c++
CHiveConfig* fetchSubconfigAt(unsigned int vIndex) const;
```

 **作用：**获取指定索引处的子配置，索引下标从0开始，返回指针（可以修改`CHiveConfig`或自定义配置的内容）。

```c++
CHiveConfig* fetchSpecifiedSubconfigAt(const std::string& vSubconfigTypeName, unsigned int vIndex) const;
```

 **作用：**获取指定类型和索引处的子配置。

```c++
[[nodiscard]] CHiveConfig* addSubconfig(const CHiveConfig *vConfig);
```

**作用：**添加子配置，其中`vConfig`是指向子配置的指针，返回的是克隆后的子配置的指针，**注意：**实际添加的是子配置的克隆，这样当子配置被释放时。

```c++
std::size_t getNumSubconfigs() const noexcept;
```

作用：获取子配置数量（不会考虑子配置的子配置这种情况，即直接子配置）。

```c++
std::size_t getNumAttributes() const noexcept;
```

**作用：**获取属性数量（）。

```c++
std::size_t getNumSpecifiedSubconfigs(const std::string& vSubconfigTypeName) const noexcept;
```

**作用：**获取指定类型的子配置数量（不包含子配置的子配置）。

```c++
std::size_t getNumSpecifiedSubconfigsRecursively(const std::string& vSubconfigTypeName) const noexcept;
```

**作用：**递归获取指定类型的子配置数量（包含子配置的子配置）。

```c++
std::pair<std::string, std::any> getAttributeAt(std::size_t vIndex) const;
```

**作用：**获取指定索引处的属性，返回<属性名，属性值>。

```c++
std::string generateConfigPathString() const;
```

**作用：**生成配置路径字符串。

```c++
EConfigDataType getDataType(const std::string& vAttributeName) const noexcept;
```

**作用：**获取指定属性的类型`EConfigDataType`。

```c++
std::optional<T> getAttribute(const std::string& vAttributeName) const;
```

**作用：**获取指定类型的指定名称的属性的值，并以optional包装后返回。

### 保护成员函数

```c++
void _defineAttribute(const std::string& vAttributeName, EConfigDataType vAttributeDataType);
```

**作用：**定义一个属性及其数据类型，常被用于对配置的预定义中属性的定义。

```c++
void _setSubconfigTypeName(const std::string& vType) noexcept;
```

**作用：**设置子配置的类型（仅当当前的配置是子配置的时候有效）。

```c++
virtual EConfigValidity _verifyConfigV() const;
```

**作用：**虚函数，用于验证配置的有效性（检查如果是子配置的话是否设置了子配置的类型，即上面的），子类可以重载此函数。

### 私有成员变量

1. `bool m_OnlyAcceptPredefinedAttribute = true;`

指示是否仅接受预定义的属性（上面提到的）。

2. `bool m_LoadingDefaultConfig = false;`

指示是否加载默认配置（函数中自己定义的）。

3. `std::string  m_Name;`

配置的名称。

4. `std::string  m_SubconfigTypeName;`

子配置的类型名称，仅在对象为子配置时有效。

5. `CHiveConfig *m_pParentConfig = nullptr;`

父配置指针。

6. `std::vector<std::shared_ptr<CHiveConfig>> m_SubconfigSet;`

子配置的集合。

7. `std::map<std::string, std::pair<std::any, bool>, hiveCommon::SInsensitiveStrLess> m_AttributesMap;`

属性映射，存储属性名称、值及是否为默认值的对。

8. `std::map<std::string, EConfigDataType, hiveCommon::SInsensitiveStrLess> m_AttributeDataTypeMap;`

属性数据类型映射。
### 私有成员函数

```c++
CHiveConfig* __cloneConfig() const;
```

**作用：**克隆配置，用于上面的添加子配置的操作中等。

```c++
void __addSubconfig(CHiveConfig *vSubconfig);
```

**作用：**添加子配置。

```c++
virtual void __defineAttributesV();
```

**作用：**虚函数，用于定义属性，子类可以重载。

```c++
virtual void __onLoadedV();
```

**作用：**虚函数，配置加载完成时调用，子类可以重载。

```c++
virtual void __loadDefaultConfigV();
```

**作用：**虚函数，用于加载默认配置，子类可以重载。

```c++
virtual bool __onProductCreatedV(const CHiveConfig* vConfig);
```

**作用：**虚函数，产品创建时调用，子类可以重载。

```c++
virtual bool __onProductCreatedV();
```

**作用：**虚函数，产品创建时调用，子类可以重载。

### 友元类
`CConfigParser、CAttributeExtractor、hiveCommon::CProductCreator`

## CStaticOffspringConfigIterator
### 公有成员函数

```c++
CHiveConfig* fetchFirstOffspringConfig(const CHiveConfig* vRootConfig);
```

**作用：**获取第一个子配置。

```c++
CHiveConfig* fetchFirstOffspringConfig(const CHiveConfig* vRootConfig, const std::string& vSubconfigTypeName);
```

**作用：**获取指定类型的第一个子配置。

```c++
CHiveConfig* fetchNextOffspringConfig();
```

**作用：**获取下一个子配置。

**注意：**在第一次使用`fetchFirstOffspringConfig`方法后，子配置的列表就已经固定了，此时如果再次加入新的子配置，则不能被`fetchNextOffspringConfig`获取。（如果需要使用新的子配置的话则需要重新使用`fetchFirstOffspringConfig`）。

### 私有成员变量

1. `std::vector<CHiveConfig*> m_OffspringConfigSet;`

 子配置的集合。

2. `unsigned int m_Counter = 0;`

 计数器，用于跟踪当前访问的子配置索引。
## CConfigParser
### 公有成员函数

```c++
EParseResult readConfig(const std::string& vConfigFileOrContent, EConfigType vConfigType, CHiveConfig *voConfig);
```

**作用：**读取配置文件或配置内容，将其解析为`CHiveConfig`对象。
**参数：**

- `vConfigFileOrContent`：配置文件的路径或配置内容的字符串，如果是配置内容，则字符串应该以`"$CONFIGURATION_STRING"`开头。
- `vConfigType`：配置文件的类型（例如 XML、JSON、INI 等），由枚举 `EConfigType` 定义。
- `voConfig`：输出参数，指向要填充的 CHiveConfig 对象。

**返回值：**

- `EParseResult` 枚举值，表示解析结果。

```c++
EParseResult writeConfig(const std::string& vOutputFileName, const CHiveConfig *vConfig);
```

**作用：**将 `CHiveConfig` 对象的内容写入配置文件。
**参数：**

- `vOutputFileName`：输出配置文件的路径。
- `vConfig`：指向要写入的 CHiveConfig 对象。

**返回值：**

- `EParseResult` 枚举值，表示写入结果。  
### 私有成员函数

```c++
EParseResult __loadConfig(const std::string& vConfigFile, EConfigType vConfigType, CHiveConfig *voConfig);
```

**作用：**用于加载配置文件并解析为 `CHiveConfig` 对象。

```c++
bool __createTempConfigurationFileFromString(const std::string& vConfigString, const std::string& vTempConfigFileName) const;
```

**作用：**将配置内容字符串写入临时文件。

## CAttributeExtractor
### 公有成员函数

```c++
EParseResult extractAttribute2Config(const boost::property_tree::ptree& vPropertyTree, CHiveConfig& voTargetConfig);
```

**作用：**从 boost::property_tree::ptree 对象中提取属性并将其填充到 CHiveConfig 对象中。

### 私有成员函数

```c++
CHiveConfig* __createSubconfigObject(const std::string& vSubconfigName, const std::string& vSubconfigTypeName, const std::string& vSubconfigSig, CHiveConfig& voParentConfig);
```

**作用：**创建子配置对象并添加到父配置中。

```c++
void __extractSingleAttribute(EConfigDataType vDataType, const std::string& vAttributeName, const boost::property_tree::ptree& vAttributeValue, CHiveConfig& voTargetConfig);
```

**作用：**提取单个属性并将其添加到目标配置对象中。

# 四个枚举
## EParseResult

```c++
	enum class EParseResult : unsigned char
	{
		SUCCEED,
		SKIP_SOME_ITEMS,
		ONLY_LOAD_DEFAULT_CONFIG,
		FAIL,
	};
```

**作用：**表示解析结果的枚举类型，各部分含义与名称相同。

## EConfigType

```c++
enum class EConfigType : unsigned char
{
	XML = 0,
	INI,
	JSON,
	INFO,
};
```

**作用：**表示配置文件类型的枚举类型。

## EConfigDataType

```c++
	enum class EConfigDataType : unsigned char
	{
		ATTRIBUTE_INT = 0,
		ATTRIBUTE_BOOL,
		ATTRIBUTE_FLOAT,
		ATTRIBUTE_DOUBLE,
		ATTRIBUTE_STRING,
		ATTRIBUTE_VEC2I,
		ATTRIBUTE_VEC3I,
		ATTRIBUTE_VEC4I,
		ATTRIBUTE_VEC2F,
		ATTRIBUTE_VEC3F,
		ATTRIBUTE_VEC4F,
		ATTRIBUTE_VECSTRING,
		ATTRIBUTE_XMLATTR,
		ATTRIBUTE_SUBCONFIG,
		ATTRIBUTE_ANON_SUBCONFIG,  //不具名的subconfig
		ATTRIBUTE_USER_DEFINED,
		ATTRIBUTE_UNDEFINED,
	};
```

**作用：**表示配置数据类型的枚举类型。

## EConfigValidity

```c++
enum class EConfigValidity : unsigned char
{
	OK = 0,
	MINOR_ERROR,
	FATAL_ERROR,
};
```

**作用：**表示配置有效性的枚举类型。

# 一个接口

## hiveParseConfig

```c++
EParseResult hiveParseConfig(const std::string& vConfigFileName, EConfigType vConfigType, CHiveConfig* voConfig);
```

参数：

- `vConfigFileName`：配置文件的名称，例如aaa.xml，记得要使用`hiveAddFileSearchPath`手动添加路径。
- `vConfigType`： 配置类型。
- `voConfig`：输出的`CHiveConfig`对象的指针。
