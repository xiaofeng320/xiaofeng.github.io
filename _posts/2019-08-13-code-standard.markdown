---
layout:     post
title:      "代码规范"
subtitle:   " \"php、mysql规范\""
date:       2019-08-13 11:00:00
author:     "xiaofeng"
header-img: "img/WechatIMG10.jpeg"
tags:
    - 技术
    - 规范
---


### php编程规约

#### 命名规约

**【强制】代码中命名只能以_和字母开始**
  
```
class A {

    private $_name; // 私有变量以下划线开始，首字母小写

    protected $_sex; // 保护变量以下划线开始，首字母小写

    public $cellphone; // 公有变量，首字母小写，不允许加下划线

    function func($name) {

        // good

        $temp = $name; //局变量，首字母小写

        // bad

        $Temp = $name;

    }

}
```

**【强制】代码中严禁使用拼音和英文组合**

**【强制】类名强制使用驼峰形式，首字母大写**

```
class AccountPayServer{

}
```

**【强制】函数参数名强制使用驼峰形式**

```
class A {

    function setAmount($accountId, $amount) {

    }

}
```

**【强制】常量命名全部大写，单词间用下划线隔开，力求表达清楚**

```
class A {

    const PAY_START = 10;

    const PAY_END = 20;

}
```

**【强制】包名统一首字母大写，其他小写**

```
use \Service\Policy;

namespace Service;
```

**【强制】全局变量必须已GLOBAL做前缀，全部大写**

```
$GLOBAL_LOG_LEVEL
```

**【推荐】避免复制已有变量**

```
反例:

$desc = strip_tags($_POST['PHP description']);

echo $desc

正例:

echo strip_tags($_POST['PHP description']);
```

#### 常量定义

**【推荐】常量分为：跨应用共享常量、应用内共享常量,跨应用内共享常量:应放到ops-package项目对应目录下**

```
示例：src/User/Pay.php
```

**【强制】常量使用：不允许直接使用常量值**

```
class A {

    const PAY_START = 10;

    const PAY_END = 20;
}

// 正例

if($status == A::PAY_START){ }

// 反例

if($status == 10){ }

```

**应用内共享常量：避免重复定义**

```
// 反例
class Pay {

    const PAY_START = 10;
}
class Order {

    const PAY_START = 10;
}
```

#### 格式规约

**【强制】缩进采用4个空格，禁止使用tab，**

**【强制】大括号使用。如果是大括号内为空，则简洁地写成{}即可，不需要换行；如果是非空代码块则：**

1) 左大括号前不换行。

2) 左大括号后换行。

3) 右大括号前换行。

4) 右大括号后还有 else 等代码则不换行; 表示终止右大括号后必须换行。

示例：

```
if($status == A::PAY_START) { }

if($status == A::PAY_START) {

    print($_POST['PHP description']);

}

function abc() {

    if(1 == 1) {

        ...

    } else {

        ...

    }
}
```

**【强制】左括号和后一个字符之间不出现空格；同样，右括号和前一个字符之间也不出现空格**

**【强制】任何运算符左右必须加一个空格**

说明:运算符包括赋值运算符=、逻辑运算符&&、加减乘除符号、三目运算符、条件比较等。

**【强制】if/for/while/switch/do 等保留字与左右括号之间都必须加空格**
```
if ($a == $b) {

    ...

}
```

**【强制】单行字符数限制不超过 80 个，超出需要换行，换行时遵循如下原则:**

1) 第二行相对第一行缩进 4 个空格，从第三行开始，不再继续缩进，参考示例。 如果是函数参数换行，第二行相对第一行缩进8个空格。
2) 运算符与下文一起换行。
3) 方法调用的点符号与下文一起换行。 
4) 在多个参数超长，逗号后进行换行。
5) 在括号前不要换行，见反例。

```
正例：
$a = new User(); 
$a->setName('tao')
    ->setNickname('dada')
    ... 
    ->setBirthdate('19990909')
    ... 
    ->setSex(1);

反例：
$a = new User(); 
$a->setName('tao')->setNickname('dada')->setBirthdate('19990909')...->setSex(1);
```

**【强制】方法参数在定义和传入时，多个参数逗号后边必须加空格。**

如： updateUser($userId, $name, $sex);

**【推荐】没有必要增加若干空格来使某一行的字符与上一行的相应字符对齐。**

```
const PAY_WAIT = 10;

const PAY_SUCCESS = 20;

const PAY_FAILURE = 30;
```

**【推荐】在执行多条语句组、变量时。不同的业务逻辑之前插入一个空行。方便阅读和理解。**


**【强制】在一个 switch 块内，明确每个case在哪儿结束**

1）每个 case 要么通过 break/return 等来终止,要么注释说明程 序将继续执行到哪一个 case 为止;

2）在一个 switch 块内，都必须包含一个 default 语句并且放在最后，即使它什么代码也没有。

**【强制】在 if/else/for/while/do 语句中必须使用大括号，即使只有一行代码。**
```
正例：

if ($a == 1) {

    print('error.');

}

反例：

if ($a == 1) print('error.');

3.3【推荐】尽量少用 else, if-else 的方式可以改写成:
if (contition) {

    statements;

    return false;

}

// TODO: 接着写 else 的业务逻辑代码;
如果必须使用if-else，请勿超过4层。超过可以用switch-case代替或者状态机来实现
```

**【推荐】循环体中的语句要考虑性能问题。如变量定义、相同的操作、数据库连接、及不必要的try-catch操作，请移至循环体外进行处理。**

**【参考】方法中不需要参数校验。**

####  注释规约

**【强制】单行注释，在被注释语句上方另起一行，使用//注释。多行注释，同样在被注释语句上方，使用/ /。需要与被注释代码对齐。**

**【强制】在改动代码时，如果上方有注释，必须对注释进行同步更新**

**【参考】针对永久不会的代码，建议直接删掉，不要使用注释。我们可以从代码仓库里找到历史代码。**

#### 日志规约

**【强制】数组或对象输出必须是以json的格式输出，不能使用var_export|var_dump|print_r等方式输出**

**【强制】exception不能直接输出对象**

```
try {

    ...

} catch (\Exception $e) {

    // good

    error_log(json_encode($e->getTrace()));

    error_log($e->getMessage());  



    // bad

    error_log($e); 

}
```


### mysql规范

**【强制】库名全部小写，采用下划线命名法，并统一以abc_作为前缀**

**【强制】表名全部小写，采用下划线命名法**

**【强制】禁止使用SQL关键字作为字段/表名 http://dev.mysql.com/doc/mysqld-version-reference/en/mysqld-version-reference-reservedwords-5-5.html**

**【强制】索引名称必须以 idx为前缀，后接直观反应索引的字段；唯一索引必须以 udx 为前缀**

**【强制】禁止定义enum/set，请使用int/tinyint/smallint代替**

**【强制】禁止使用外键，由应用中保证数据一致性**

**【强制】业务开发中禁止使用触发器/存储过程(脚本中使用存储过程必需备案)**

**【强制】业务开发中禁止左值使用函数(左值使用的函数必需备案)**

**【强制】索引列应该定义为 not null，并且设定非null的默认值**

**【强制】必须用innoDB存储**

**【强制】字符集统一用UTF8**

**【强制】禁止使用小数存储货币，统一使用毫作为货币单位**

**【强制】自增主键使用BIGINT(20)**

**【建议】尽量避免在字段说明里加常量定义。在常量值有更新的情况下，可能会导致两个问题：1、说明更新不及时，引起使用者误会；2、如果表数据过大，更新字段说明文案的成本会变的很高。**

**【建议】禁止使用blob/mediumblob/longblob/mediumtext/longtext**

**【建议】尽量避免使用select * 进行查询。**

**【建议】尽量避免连表查询，可以考虑两次查询的方式来实现**

**【建议】尽量避免 !=, NOT IN , OR**

