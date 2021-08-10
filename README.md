# 同济大学19级信管Web实践——学习助手

[TOC]

该项目的GitHub地址为：[https://github.com/Feng-Yz/Study-Helper](https://github.com/Feng-Yz/Study-Helper)或：[https://github.com/Weber-LiWb/AI-Study-Helper](https://github.com/Weber-LiWb/AI-Study-Helper)

通过IP地址可以访问该项目网站：[http://47.103.73.100](http://47.103.73.100)

## 1 功能设计

### 1.1 项目背景

如今，随着高等教育改革和“双一流”学校建设持续推进，大学生面对更高的课程考核要求，更多的实践学习项目和更频繁的团队合作。这样的现状要求大学生具有更强的时间管理能力、规划能力和团队协调能力。与此同时，大学学习有很大一部分需要学生发挥主观能动性自行探索。然而，在现实中普遍存在同学们有问题但却羞于问出口的情况；而同学们收集的学习资料、整理的学习心得也没有机会在同学间交流，形成了一个原子化的隔离状态，失去了共同学习的机会。基于此，我们目标开发一套具有日程管理、小组分工、匿名博客功能在内的校园学习社交平台——”济忆校园“。

在济忆校园网站，同学们可以通过添加DDL（Deadline）到日程规划中对未来时间的分配有清晰的规划。可以在每日日程中标记今天的各项学习生活日程完成情况；可以创建小组，建立小组任务，并将小组任务分解为子任务进行分配，分配到组员中的任务将自动进入组员的日程规划；可以发表博客，发表评论，选择向自己的朋友公开自己在博客中的身份。这样，同学们就可以借助日程、小组等功能合理规划学习与日常生活，可以通过匿名博客提出学习疑问、分享学习心得、或者简单分享今天的心情。在济忆校园，同学们可以收获明晰的时间管理，也可以收获一个共同学习的秘密花园。济忆校园为同学们提供了科学规划时间，互助共同学习的功能，让学习在”内卷时代“回归学习本质。

### 1.2 网站功能

经过讨论，网站的功能分为基本模块、博客模块、日程模块与小组模块四个部分，每个初步设计的功能如下：

基本模块：注册、登录、修改密码、申请添加好友、通过好友申请、删除好友；

博客模块：发表博客、更改博客、删除博客、按页查看平台所有博客、按好友查看好友博客、发表评论、删除评论、收藏博客；

日程模块：查询日程、添加日程、完成确认；

小组模块：新建小组、查询小组、加入小组、添加小组任务、分配小组任务、分配任务添加至个人日程。

## 2 数据库设计

根据功能需求，抽象出以下实体以及实体之间的联系，绘制出ER图：

![数据库ER图](images/数据库ER图.png)

在本项目中，利用Django的对象关系映射（ORM）模块，使用类和对象对数据库进行操作。具体来说，也就是Django的MVT模式中的模型（Model），模型准确且唯一的描述了数据。它包含储存的数据的重要字段和行为。一般来说，每一个模型都映射一张数据库表。每个模型都是一个Python的类，这些类继承`django.db.models.Model`，模型类的每个属性都相当于一个数据库的字段。

对于该网站的实体、联系以及实体中各字段的说明如下：

### 2.1 用户（User，Profile）

学/工号（username）：由于系统服务于同济师生，该字段为最长为7个字符的字符串；

邮箱地址（email）：用户的邮箱地址，用于登录；

密码（password）：用户的密码，Django实现密码的哈希加密；

姓名/昵称（name）：用户自定义的昵称，最大长度为10个字符；

性别（gender）：用户的性别，包括M（男性）与F（女性）；

用户类型（type）：包括S（学生）与T（教师）；

班级（class_name）：用户所在的班级名称或院系，允许空。

值得注意的是，Django中有现成User类，只需要`from django.contrib.auth.models import User`。该类中，有属性`id`作为主键，且每个用户具有唯一的属性`username`，故将该属性作为学/工号。另外，对于Django自带的User类中未提供的字段，我们采取新建Profile类、将其和User建立一对一关系的方法实现。

### 2.2 小组（Group）

小组序号（id）：小组的序号，是Django自建的主键；

小组名称（group_name）：小组的名称，最大长度为20个字符；

小组种类（type）：小组的种类，最大长度为20个字符；

组长（leader）：小组的组长，以用户（User）为外键。

小组与用户是多对多的关系，即一个用户可以在多个小组，一个小组可以有多个用户。Django在根据定义的类进行建表时，会建立一张存储小组与成员关系的表。

### 2.3 用户日程（Schedule）

日程序号（id）：日程的序号；

日程描述（description）：对于日程的描述，最大长度为50个字符；

日程类型（type）：日程类型，例如学习类、运动类等，最大长度为5个字符；

是否重复（is_repeated）：日程是否需要按周期重复，例如每天、每周、每月；

重复周期（repeat_cycle）：包括D（每天）、W（每周）与M（每月）， 允许空；

开始时间（start_time）、截止日期（deadline）：日程的开始时间和截止日期；

权重（weight）：日程的重要程度；

预计所需时间（expected_minutes_consumed）：预计完成该日程需要花费的时间，以分钟为单位，允许空；

是否完成（is_done）：布尔类型，该日程是否完成。

用户与用户日程是一对多的关系，因此添加属性用户（user）。

### 2.4 小组任务（GroupAssignment）

任务序号（id）：任务的序号；

任务描述（description）：对于任务的描述；

截止日期（deadline）：任务的截止日期。

小组与小组任务是一对多的关系，因此添加属性小组（group）。

### 2.5 子任务（SubAssignment）

子任务是对小组任务的分解，将小组任务分解给每位小组成员。

子任务序号（id）：作业的序号；

前置子任务（pre_sub_assignment）：为了完成该任务所需要的前置子任务，字符串类型；

用户（user）：负责完成该任务的小组成员；

任务描述（description）：对于子任务的描述；

截止日期（deadline）：子任务的截止日期；

权重（weight）：子任务的重要程度；

预计所需时间（expected_minutes_consumed）：预计完成该子任务需要花费的时间，以分钟为单位，允许空。

任务与子任务是一对多的关系，因此添加属性任务（assignment）。

### 2.6 博客（Blog）

博客序号（id）：博客的序号，是Django自建的主键；

用户（user）：博客的作者；

标题（title）：博客的标题，最大长度为50个字符；

内容（content）、浏览量（pageview）、收藏量（collect_amount）、创建时间（created_time）、修改时间（modified_time）：博客的内容、浏览量、收藏量、创建时间与修改时间。

### 2.7 评论（Comment）

评论序号（id）：评论的序号；

用户（user）：评论博客的用户；

内容（content）：评论的内容；

创建时间（created_time）：评论的创建时间。

博客与评论是一对多的关系，因此添加属性博客（blog）。

### 2.8 好友（Friend）

用户（user）、好友（friend）：用户以及其好友；

权限（authority）：以整数存储，表示用户开放给好友的查看权限。

用户与好友作为共同主键。

### 2.9 收藏（Collection）

用户（user）、博客（blog）：用户以及其收藏的博客；

种类（type）：收藏的种类。

用户与博客作为共同主键。

### 2.10 完成日程（FinishedSchedule）

序号（id）：该完成日程的序号；

完成时间（finish_time）：完成日程的时间；

花费时间（minutes_consumed）：完成日程花费的时间，以分钟为单位。

日程与完成日程是一对多的关系，因此添加属性日程（schedule）。



综上，建立的Django中的[models.py](helper/models.py)文件。

## 3 主要界面

### 3.0 UX设计
网站整体视觉体系选用同济大学特色的蓝白色，整体设计风格接近以Mac OS为代表的扁平拟物化风格。通过色块简明地划分功能区域，通过卡片式布局实现功能模块的实现。同时以适当的图片展示凸显同济特色。

#### 3.0.1 UI体系
为了更好地实现功能，我们在UI设计时遵循了Apple生态UI设计规范。

![设计规范](images/设计规范.png)

在Apple设计规范中，有四大原则，分别是：Flexible(灵活性)、Expansive(可拓展性)、Capable(功能满足性)、Focused(聚焦性)。即，在UI设计中需要保证整体设计的灵活性，不能死板而丧失拓展空间；同时要保证UI设计满足功能需要，能够很好地承载目标功能；同时，设计需要突出聚焦的功能，让使用者能过专注于正在使用的功能和界面。

我们在整个UI设计中大体遵循了这样的设计理念。在具体的UI设计上，参考了Apple在IOS14和Mac OS Big Sour中广泛使用的卡片式布局，在内容呈现方面则参考了Fackbook、Twitter、知乎等内容社交平台的界面。UI设计的基础有赖于SmartBootstrap提供的样式，并在此基础上结合项目特色和设计原则进行修改优化。

借助Figma这样的主流UI设计软件，我们确定了以下两个基本设计模板：

![功能模板](images/功能模板.png)

功能性UI模板

![内容模板](images/内容模板.png)

内容性UI模板

我们使用侧边栏与顶栏双导航栏的设计。侧边栏主要承载功能模块的导航，顶栏负责提供系统功能和用户信息的展现。不同的模块通过不同的卡片搭配实现功能。这样的模块化设计将简化前端的开发流程，同时实现丰富的功能并预留了后期拓展的余地。

在主视觉系统上，我们选用了同济大学特色的蓝白色系进行搭配。

<img src="images/tj1.png" alt="tj1" style="zoom:67%;" />

![tj2](images/tj2.png)

![cl1](images/cl1.png)

主视觉蓝色（Peimary Color）的十六进制色号为“#0D6EFD”，与之搭配的色卡为：

![cl2](images/cl2.png)

基于此，同时依据SmartBootstrap提供的模板样式，我们在项目中采用了以下色彩搭配。

![cl3](images/cl3.png)

#### 3.0.2 响应式布局

借助Bootstrap本省的响应式布局系统，即页面根据不同的宽度等级（xs:<576px, sm>576px, md>768px, lg>992px, xl>1200px）的基础上对显示样式进行不同的调整，然后通过在CSS中使用@media方法，实现了在不同尺寸设备下不同的显示效果。具体效果如下：

1. 桌面设备

   ![d1](images/d1.png)

2. 移动设备

   ![d2](images/d2.png)

#### 3.0.3 用户友好

根据我们遵循和确定的UI设计规范，我们在UX(User Experience 用户体验)设计中要体现用户友好的原则，让操作直接明了，降低用户的学习成本。基于此，我们设计了以下细节：

1. 响应提示

   在用户进行注册等表单填写操作时，在输入框中通过弹出提示指导用户输入正确格式的内容。

   ![t1](images/t1.png)

2. 弹窗提示

   在用户错误操作导致不能指向目标页面或实现目标操作时，我们设计和弹窗提示页面。

   ![t2](images/t2.png)

3. 便捷操作

   为了让用户在使用过程中更加专注与内容展示，我们提供了折叠侧边栏的功能，

   before：

   ![t3](images/t3.png)

   after：

   ![t4](images/t4.png)

   为了让用户在长页面环境中的使用体验更佳，我们设计了回到顶部按钮，用户可以一键回到顶部。

   ![t5](images/t5.png)

### 3.1 用户相关界面

在使用该网站时，用户首先访问URL`/`。如果用户已经登录，则重定向至用户的个人主页；否则重定向至登录界面。

#### 3.1.1 注册界面

该界面用于用户的注册，URL为`register/`。

若请求为GET方法，无请求参数，返回参数为表单`form`，`form`包括的字段有：

|    字段    |     说明     |  类型  |             备注             | 是否必填 |
| :--------: | :----------: | :----: | :--------------------------: | :------: |
|  user_id   |   学/工号    |  Char  |   max_length=7，有效、唯一   |    是    |
|   email    |     邮箱     | Email  |          有效、唯一          |    是    |
| user_name  |     昵称     |  Char  |        max_length=10         |    是    |
|   gender   |     性别     | Choice |   ('M', '男'), ('F', '女')   |    是    |
| user_type  |   用户类型   | Choice | ('S', '学生'), ('T', '教师') |    是    |
| class_name |     班级     |  Char  |        max_length=20         |    否    |
| password1  |     密码     |  Char  |          不少于6位           |    是    |
| password2  | 再次输入密码 |  Char  |             一致             |    是    |

若请求为POST方法，请求参数为表单`form`，验证表单有效后创建用户，重定向至登录界面；若无效，返回错误提示信息`message`。

#### 3.1.2 登录界面

该界面用于用户的登录，URL为`login/`。

若请求为GET方法，无请求参数，返回参数为表单`form`，`form`包括的字段有：

|    字段    |     说明     |  类型  |             备注             | 是否必填 |
| :--------: | :----------: | :----: | :--------------------------: | :------: |
|   email    |     邮箱     | Email  |          有效、存在          |    是    |
| password  |     密码     |  Char  |                     |    是    |

若请求为POST方法，请求参数为表单`form`，验证密码正确后重定向至个人主页，否则返回表单`form`与错误提示信息`message`，提示“密码错误，请重新输入！”。

#### 3.1.3 修改密码界面

该界面用于用户修改密码，URL为`user/pwd_change/`。

若请求为GET方法，无请求参数，返回表单`form`与用户`user`，`form`包括的字段有：

|     字段     |     说明     | 类型 |   备注    | 是否必填 |
| :----------: | :----------: | :--: | :-------: | :------: |
| old_password |   旧的密码   | Char |           |    是    |
|  password1   |     密码     | Char | 不少于6位 |    是    |
|  password2   | 再次输入密码 | Char |   一致    |    是    |

若请求为POST方法，请求参数为表单`form`，验证旧密码正确后重定向至登录界面，否则返回表单`form`与错误提示信息`message`，提示“旧密码错误！”。

#### 3.1.4 个人主页

该界面的URL为`user/homepage/`。

请求为GET方法，无请求参数，返回参数如下：

用户（user）：当前登录用户（经过验证后也就是id为GET请求中的id的用户）；

个人日程（schedules）：按开始时间从近到远排序的最近n（可以设置）天的所有日程（包括按周期重复的所有日程）；

小组子任务（group_sub_assignments）：按截止日期从近到远排序的最近n天的所有日程；

好友（friends）：该用户添加的好友；

博客（blogs）：目前网站上最近发表的或比较热门的n（可以设置）条博客。

#### 3.1.5 好友管理界面

该界面的URL为`user/friends_admin/`。

若请求为GET方法，无请求参数，返回参数有：

好友（friends）：用户添加的好友，且对方已经通过申请；

未授权好友（friends_not_authorised）：向用户发起的好友申请的用户。

若请求为POST方法，则有下列三种情况：

1. 用户删除好友，POST请求中包含被删除的好友的id（delete_id）；
2. 用户同意好友申请，POST请求中包含同意的好友的id（agree_id）；
3. 用户发起好友申请，POST请求中包含好友的学号（apply_id）。

若传回的数据无效，则返回错误信息`message`。

#### 3.1.6 登出

用于用户退出登录，URL为`logout/`。

请求为GET方法，无请求参数，退出登录后重定向至登录界面。

### 3.2 博客相关界面

#### 3.2.1 博客主页

该界面的URL为`blog/home/`。请求方法为GET方法，无请求参数，返回参数有：

博客（blogs）：包括该用户发表的博客；

收藏（collections）：该用户收藏的博客。

#### 3.2.2 新建博客

该界面的URL为`blog/add/`。

若请求为GET方法，无请求参数，返回参数为一张表单（form），表单中包含的字段有：

|  字段   |   说明   | 类型 | 备注 | 是否必填 |
| :-----: | :------: | :--: | :--: | :------: |
|  title  | 博客标题 | Char |  无  |    是    |
| content | 博客内容 | Char |  无  |    是    |

若请求为POST方法，请求参数为表单（form），在新建博客后重定向至该篇博客的界面。

#### 3.2.3 更改博客

该界面的URL为`blog/<int:pk>/modify/`。

若请求为GET方法，请求参数为博客的id，返回参数为一张表单（form），表单中包含的字段有：

|  字段   |   说明   | 类型 |     备注     | 是否必填 |
| :-----: | :------: | :--: | :----------: | :------: |
|  title  | 博客标题 | Char | 初始为原标题 |    是    |
| content | 博客内容 | Char | 初始为原内容 |    是    |

若请求为POST方法，请求参数为表单（form），在更改博客后重定向至该篇博客的界面。

#### 3.2.4 删除博客

该界面的URL为`blog/<int:pk>/delete/`。请求为GET方法，请求参数为博客的id，在删除博客后重定向至博客的个人主页。

#### 3.2.5 博客界面

该界面的URL为`blog/<int:pk>/`。

若请求为GET方法，请求参数为博客的id，返回参数有：

用户（user）：目前登录的用户；

博客（blog）：请求的博客；

评论（comments）：该博客下的所有评论。

若请求为POST方法，则有下列三种情况：

1. 用户删除评论，POST请求中包含被删除的评论的内容（delete_comment）；
2. 用户收藏博客，POST请求中包含参数（collect_blog）即可；
3. 用户新建评论，POST请求中包含评论的内容（create_comment）。

#### 3.2.6 热门博客

该界面的URL为`blog/hot/<int:pg>/`。

请求为GET方法，考虑到热门博客有很多，将其分为几页，请求参数为页码。若请求参数非法，则返回404NotFound；若请求参数合法，则返回参数包括：该页的博客（blogs）、当前页码（current_page）和总页数（page_num）。

#### 3.2.7 个人公开界面

该界面的URL为`blog/friend/<int:friend_id>/`。

请求为GET方法，请求参数为好友的用户id，若好友没有开放权限给用户，则返回信息（`message`）“没有权限访问！”；否则返回好友的博客（blogs）与好友的信息（User类，friend）。

### 3.3 小组相关界面

#### 3.3.1 小组主页

该界面的URL为`group/<int:pk>/`。

请求为GET方法，请求参数为小组的主键id，若当前用户不是小组的成员，返回**403Forbidden**；否则返回小组（group）、小组的成员（partcipants）、小组的任务（assignments）、小组分配的子任务情况（sub_assignments）。

#### 3.3.2 添加任务

该界面的URL为`group/<int:pk>/add_assign/`。若当前用户不是小组组长，则返回**403Forbidden**。

若请求为GET方法，请求参数为小组的id，返回参数为一张表单（form），表单中包含的字段有：

|    字段     |   说明   |   类型   | 备注 | 是否必填 |
| :---------: | :------: | :------: | :--: | :------: |
| description | 任务描述 |   Char   |      |    是    |
|  deadline   | 截止日期 | Datetime |      |    是    |

若请求为POST方法，请求参数为表单（form），若表单有效则在新建小组任务后重定向至小组主页。

#### 3.3.3 分配子任务

该界面的URL为`group/<int:pk>/add_sub_assign/`。若当前用户不是小组组长，则返回**403Forbidden**。

若请求为GET方法，请求参数为小组的id，返回参数为一张表单（form），表单中包含的字段有：

|           字段            |     说明     |   类型   |            备注            | 是否必填 |
| :-----------------------: | :----------: | :------: | :------------------------: | :------: |
|        description        |   任务描述   |   Char   |                            |    是    |
|         deadline          |   截止日期   | Datetime |                            |    是    |
|    pre_sub_assignment     |   前置任务   |   Char   |                            |    否    |
|        start_time         |   开始时间   | Datetime |                            |    是    |
|           user            |     用户     |  Choice  | 选择范围为该小组的所有成员 |    是    |
|        assignment         |    父任务    |  Choice  | 选择范围为该小组的所有任务 |    是    |
|          weight           |   紧急程度   |   Int    |                            |    是    |
| expected_minutes_consumed | 预期花费时间 |   Int    |                            |    是    |

若请求为POST方法，请求参数为表单（form），若表单有效则新建小组子任务、同时为用户新建相关日程，然后重定向至小组主页；否则返回报错`message`：添加失败。

#### 3.3.4 小组管理

该界面的URL为`group/`。该界面的功能有显示小组、新建小组、根据组长学号搜索小组并加入的功能。

若请求为GET方法，无请求参数，返回参数有：

表单（add_form）：用于新建小组的表单；

小组（groups）：用户加入的所有小组。

若请求为POST方法，则有下列三种情况：

1. 用户新建小组，POST请求中包含新建小组的名称（group_name）、小组种类（type）；
2. 用户根据组长学号搜索小组，POST请求中包含组长的学号（leader_id），此时返回该组长的所有组（leader_groups）；
3. 用户加入小组，POST请求中包含搜索到的小组id（group_id）。

若出现组长学号不存在或小组序号不存在的情况，则返回错误信息`message`。

### 3.4 日程相关界面

#### 3.4.1 日程主页

该界面的URL为`schedule/`。

若请求为GET方法，无请求参数，返回参数有：

天数（day_num）：查询day_num天数之内的日程；

日程（schedules）：查询的结果；

今日日程（daily_schedules）：今日需要完成的日程。

若请求为POST方法，则有下列三种情况：

1. 用户查询一定天数之内的日程，POST请求中包含天数（search_day_num）；
2. 用户完成日程，POST请求中包含完成的日程序号（finish_id）；
3. 用户完成时间，POST请求中包含完成日程花费的时间（time_consumed）。

#### 3.4.2 添加日程

该界面的URL为`schedule/add/`。

若请求为GET方法，请求参数为小组的id，返回参数为一张表单（form），表单中包含的字段有：

|           字段            |     说明     |   类型   |            备注            | 是否必填 |
| :-----------------------: | :----------: | :------: | :------------------------: | :------: |
|        description        |   日程描述   |   Char   |                            |    是    |
|         deadline          |   截止日期   | Datetime |                            |    是    |
|        is_repeated        |   是否重复   | Boolean  |                            |    是    |
|           repeat_cycle            |     重复周期     |  Choice  | 每日、每周、每月 |    否    |
|        start_time         |   开始时间   | Datetime |                            |    是    |
|        type         |    日程种类    |  Char  |  |    是    |
|          weight           |   紧急程度   |   Int    |                            |    是    |
| expected_minutes_consumed | 预期花费时间 |   Int    |                            |    是    |

若请求为POST方法，请求参数为表单（form），若表单有效则新建日程，然后重定向至日程主页；否则返回报错`message`：添加失败。

## 4 主要技术

### 4.1 前端技术

本组的前端技术主要使用Figma进行基础设计，通过Figma生成的CSS样式对Bootstrap的CSS样式进行调整与个性化设计，并同步更新JS文件。

![l1](images/l1.png)

![l2](images/l2.png)

本组前端技术主要基于BootStrap4.0实现。

![f1](images/f1.png)

BootStrap是由Twitter的工程师们为了规范Twitter的网页开发而设计的前端框架，并在GitHub进行了开源。Bootstrap代码规范良好，提供了丰富的组件和方法，让前端开发工作变得更加简洁。Bootstrap是目前最受欢迎的HTML、CSS和JS框架。

Bootstrap的响应式格栅系统可以非常方便的让网页实现跨平台运用。同时支持了less动态样式和内置丰富的jQuery插件，可以让Web开发更加便捷美观。

一个典型的Bootstrap目录结构如下：

![Bootstrap结构](images/Bootstrap结构.png)

- CSS提供了样式，包含min字样的文件是压缩版本；
- JS提供了脚本。

在模板确定与CSS样式确定阶段，我们使用了Figma工具。

Figma是一个向量图形编辑器和原型设计工具。可在Web完成跨平台即时协作的UI设计并自动生成CSS文件。它基于网页、Mac OS/Windows通用、着重于用户界面设计并强调即时协作，是目前除Sketch外最流行的UI设计工具，但是其跨平台的优势是Sketch所不具备的。

![f2](images/f2.png)

### 4.2 后端技术

本小组后端主要使用Django（版本为2.0）搭建框架。

Django是一个开放源代码的Web应用框架，由Python写成。采用了MVT的软件设计模式，即模型（Model），视图（View）和模板（Template）：

- 模型（Model）：与经典MVC模式下的模型Model类似；

- 视图（View）：与MVC下的控制器（Controller）更类似。视图不仅负责根据用户请求从数据库读取数据、指定向用户展示数据的方式（网页或json数据）， 还可以指定渲染模板并处理用户提交的数据。

- 模板（Template）：与MVC模式下的视图（View）一致。模板用来呈现Django View传来的数据，也决定了用户界面的外观。模版里面也包含了表单，可以用来搜集用户的输入内容。

Django框架的核心包括：一个对象关系映射器，用作数据模型（以Python类的形式定义）和关系型数据库间的介质；一个基于正则表达式的URL分发器；一个视图系统，用于处理请求；以及一个模板系统。如下图所示：

![Django框架](images/Django框架.png)

## 5 人员分工

### 5.1 前端实现

| 工作               |         人员         |
| :-----------------: | :------------------: |
| 概念、功能设计     |        李炜博        |
| 页面规划/UI设计    |        李炜博        |
| 用户友好设计与实现  |        李炜博、谭世杰        |
| HTML实现 | 李炜博 |
| 页面测试、调试     |    李炜博、谭世杰    |

### 5.2 后端实现

| 工作               |         人员         |
| :-----------------: | :------------------: |
| 服务器配置         |        封钰震        |
| 数据库设计         |     封钰震、缪洲     |
| 用户模块、博客模块 |        封钰震        |
| 小组模块、日程模块 |     缪洲、封钰震     |
| 后端整合           |        封钰震        |

### 5.3 整体呈现

| 工作               |         人员         |
| :-----------------: | :------------------: |
| 前后端对接         |    李炜博、封钰震    |
|    测试    | 李炜博、谭世杰、封钰震 |
| 报告编写 | 封钰震、李炜博、缪洲、谭世杰 |

## 6 实习心得

### 6.1 1850811 李炜博

在这一次的项目开发中我主要负责了功能设计、页面规划和大部分的前端工作以及前后端对接、综合调试的工作。

项目分配任务之处，其实我的压力是比较大的。因为我之前并没有接触过Web项目的开发，对于前端技术更是一无所知。唯一可以称为“优势”的地方可能就是自己做过一些设计作品，个人也对设计比较感兴趣。不过，即使这样的“优势”在后期开发过程中也被证明并不存在。

万事开头难，项目刚开始的时候我甚至用PPT完成了第一版的UI设计，现在回想起来真是十分滑稽。一头雾水之际，我一方面在网络上多方收集信息，另一方面向有经验的学长学姐、计算机/软件的同学取经，这才在心中有了一点底，在脑袋里对要做的事情能有一个模糊的轮廓。在跟组内有过Web开发经验的缪洲同学沟通之后，我确定了使用Bootstrap作为前端的框架。这是因为Bootstrap一方面流行范围广，有较多的案例可以参考，另一方面则是因为Bootstrap的组件设计与我个人构思的UI设计较为接近，且Bootstrap提供了规范详细的官方文档，非常便于学习，同时，Bootstrap对我们项目使用的Django框架支持性也更好。

之后，我很想当然的就下载了原生的Bootstrap包，并直接进行界面设计。很快我就发现这样的办法并行不通，可能坐在电脑前调试一下午也不能将脑海中构想的页面实现。于是我只好在Bootstrap提供的案例网站中一个一个研究源代码，每看到一个优秀的设计就仔细学习并模仿他的HTML实现方式。唯一的问题就是这些优秀案例基本上都不是使用标准Bootstrap库实现的，而在网页上我无法获取其CSS文件。我只好一方面回归本源学习Twitter网站的网页源代码，另一方面则寻找一些优秀的UI模板如ElementUI等。最后，在SmartBootstrap项目中我找到了一个比较合适的基础样式模板同时也学习了在Figma平台进行UI设计的方法，而此时我对Bootstrap框架也相对熟悉了，之后的页面设计和代码实现虽然也耗费了时间精力，但是心理上的压力就不像之前一样沉重。这给我一个很深的印象——办法总比困难多。不要因为困难或者恐惧就止步不前，如果没有人攀登，华山永远是一处绝境，但只有迈出了攀登险峰的第一步之后，我们才能看到第二步前进的方向。

得益于后端同学提供的规范文档和模板，我与后端的封钰震同学在前后端对接这一部分的工作进行得较为顺利，特别是使用Git托管代码的方法极大地降低了代码合并的工作量，无论前端、后端哪个地方出现了需要修改的部分或者bug，都可以通过本地修改之后commit、pull request等方法及时地与封钰震同学更新，让后期测试、调试的工作处理起来十分流畅，体验极好。

在这一次的项目开发中，我对一个主流的前端框架有了一定程度的熟悉，了解了Web开发涉及的方方面面和具体流程，实践了上学期学习的软件工程开发方式、熟练使用了Git、Pycharm等优秀、便捷、易用的程序开发工具、增强了英文文档的阅读能力，也对Django框架有了初步的了解。这些收获是我在项目之处完全没有想到的，十分感谢老师给我们这样一次暑期实践的机会，感谢Bootstrap、SmarBootstrap的开源精神、感谢Bootstrap优秀的官网文档、感谢Chrome浏览器的网页源代码检查功能、感谢Miler撰写、李伟翻译的《Django项目实例精解》、感谢小组成员的通力合作，这都是这次暑期实践顺利结束不可缺少的因素。

### 6.2 1951362 封钰震

在这次项目中，我主要负责的有云服务器的配置、数据库的设计、大部分的后端工作以及与前端同学的对接。

在项目开始时，首先和缪洲同学确定了以Django作为后端框架。Django是目前比较流行的一款开源Web应用框架，其遵循MVT的软件设计模式；同时通过Python语言就可以搭建我们的网站，且可以在Python语言中加载很多开源的库。

接着我在阿里云购买了云服务器，我之前有LAMP搭建个人博客的经验，因此在云服务器上进行了环境的搭建。接着根据功能进行了部分数据库的设计，然后将我们网站的所有功能分成了四个模块，其中两个模块交给了缪洲同学。缪洲同学反映数据库设计不足以支撑其功能，便让他自行修改他那部分数据库。在制作模块后端的时候，我首先在GitHub上面查找了一些Django项目的源码进行阅读，之后根据Django的MVT模式，依次制作模型、视图和模板，在模板中提供给前端大致的样式，并在README文档中告诉前端后端的接口。

在后端合并时，我发现缪洲同学修改的数据库有很多冗余、不符合数据库设计的范式。于是对冗余的部分进行了删改，删改后视图部分也需要进行较大的修改。因此，在后端合作的时候，出现了一点问题。这给我在以后的小组合作中有所启示，应该先把顶层设计敲定好，才能给底层的实现减少麻烦。

由于在后端制作的过程中，我一边开发一边维护提供给前端同学的接口文档，我也修改了缪洲同学的代码，并和他一起写了文档，在写文档的同时提供了大致的模板文件（Template）。因此，我与前端李炜博同学工作的对接相对比较顺利。另外，我们采用Git托管代码，写完的部分Commit到GitHub云端仓库，需要合并的地方由李炜博同学Pull Request，代码合并异常顺利。

项目中碰到的问题主要还是上面所说的后端代码整合，另外还有代码规范问题。除此之外，还有零零碎碎的一些小问题，例如Django的版本问题、调试过程中遇到的各种各样的小Bug、将循环日程按天呈现给用户的方法等。

通过这次暑期实践，我学习了Django这样一个主流框架、知道了一个网站从设计到最终实现需要完成的事、更加熟悉了Linux服务器命令行的一些指令。最后，要感谢Pycharm提供便利的IDE、感谢GitHub提供有效的代码托管服务、感谢Django提供翔实的官方文档、感谢Stack Overflow等论坛的网友提供十分有用的交流、感谢Alibaba提供廉价的学生服务器、感谢李健工程师出版的Django书籍、感谢组员的配合！

### 6.3 1951369 缪洲

这一次参与暑期实习我和我的同学一起开发了一个关于学生个人管理的网站平台，参与设计了网站的后端，参与了网站的日程设计，小组管理设计，数据库设计以及网站文档的编写。

学习了django的基本框架，ORM的数据查询语法。之前我经常使用python的pymysql库进行数据库的查询，这样的查询是比较便于理解的，因为只需要进行固定的连接和查询，更新操作，但是也会有一定的麻烦，首先是需要封装连接和查询的函数，另外就是数据库需要自己另外建立。然而这些颇多不便都可以用ORM语句解决。我们可以在models文件中建立数据库的文件，然后通过引用models中的实体，进行数据库的相关操作，这是django便于用户进行数据库操作的一个帮助。

另外认识了Form组件。Form组件的数据查询语法，form组件能够从前端获取数据，并且返回数据处理之后的结果。之前我认为判断数据的工作是由前端完成的工作，但是form组件为用户提供了较为完善的数据的判断结果。Form组件并且像前端提供了自动生成组件的方式以及返回错误信息的方式，并且也可以帮助用户保存上一次输入的结果。用户可以设置多种输入的限制，比前端设置限制要更加灵活多样，因此选择使用form组件是不错的选择。

另外学会了使用django的auth组件，这是django为了用户设置的用户类，比自己设置的用户类要更加的科学安全，因为这是经过了密码加密的设置，因此选择使用auth组件也是一个不错的选择。

在设计此次小学期作业的时候，我也犯了数据库设计冗杂的错误。由于事先并没有经过仔细的推敲，而只是注重代码的实现，因此设计出了结构冗杂的数据库，给其他同学带来了困扰，这给我的教训是进行网页设计时首先需要进行数据库的设计，之后再进行后端代码的书写。只有优先考虑数据库的设计，才能让后端的代码更加科学，也更加的便于其他人维护改进。

以及代码的规范性书写方面也是进一步需要去改进的方向，这关乎其他人能否快速的读懂自己的代码，是否能够维护自己的代码。首先代码需要加入注释，其次需要去除用于调试的多余的代码，最后需要注意变量的命名规则以及书写的规范。

### 6.4 1951785 谭世杰

先说说知识上的收获吧。原本我是一个对于Web开发完全没有知识储备的人，现在对Web开发有了个基本的了解，学习了HTML, CSS, JavaScript，也学习了一些bootstrap组件的使用。学习了关于URL的一些知识，对HTTP协议有了更深的了解。

然后是自学经验上的收获。我以前认为自己是一个自学能力很差的人。然而这门课逼迫着我去自学。这段时间的自学我总结出两个经验，一是不要怕困难，要多投入时间。小学期期间，我和另一组的海同学交流过小学期的进度。我发现他比我学的快很多，一问才知道，他在选题的时候就开始每天花时间学前端的知识了。我反省自己，虽然我那时也尝试去了解了一些Web知识，但是看到很多陌生的名词和单词缩写，以及晦涩的内容后，我产生了畏难的心理，之后的学习也不积极。其实自学就应该像海同学那样积极投入时间，因为开头是最难的部分。越是难，越应该去面对，只有积极面对，它才会变得简单。二是我认识到了不止是方向指引行动，行动同时也有助于找到方向。一开始接触Web时，我不知道应该先学些什么，一开始我在网上胡乱地搜索，也没弄明白个所以然。后来我决定先学习bootstrap，才发现需要HTML, CSS, JavaScript的基础。当我学习完这些之后再学习bootstrap，才终于理解“前端框架”是个什么含义，bootstrap就像是python里导入库后可以方便地调用函数一样，只要设置一个class就能轻松解决一个小部件的css样式，之后我了解到Vue等其他框架也是值得学习的，了解到我还应该深入学习get/post相关的知识……越是去学习，越是能接触更多的知识，才越能了解到自己需要学什么，了解到自己现在学的知识有什么用处。

然后是遗憾，小学期时间很紧迫，有段时间因为个人身体状况不佳，耽误学习进度，导致参与度不够高，回想起来觉得可惜，如果时间没有这么紧就好了。但转念一想，也没必要在意一时的得失，学习的路还很长，至少我已经入了门，有了基础的前端知识，知道了前后端开发会有哪些常用的框架和工具，因此能有很明确的学习方向。同时我还有了有后端开发经验的同学，我学习后端知识也能更轻松些。

## 7 项目总结

本项目整体使用Django框架。具体实用技术如下：

### 7.1前端技术

前端整体使用Bootstrap框架，使用JavaScript、Jquery实现响应式布局和动态效果。

UI设计使用Figma平台，完成基础CSS样式以及页面模板制作。

整体结合SmartBootstrap完成模块化设计。建立了UI设计标准和模板，实现模块化开发，在保证界面一致性的前提下提高了开发效率与代码复用率。实现了响应式设计与跨平台运用，实现了用户友好型设计。

### 7.2后端技术

采用MySql关系型数据库管理系统，在阿里云搭建基于CentOS的Linux服务器，可通过远程连接直接访问。

使用了Django框架中的ORM组件去调用数据库中的数据并返回给前端，前端可以直接通过POST或GET请求来查询数据并接收返回值。

在数据交互方面，使用GET和POST进行前后端的数据交互，有的以表单的形式，有的以Json格式传递。

### 7.3开发技术

开发过程统一使用Pycharm IDE，Pycharm提供了丰富的功能和插件，满足开发过程全栈技术的需要。

代码使用Git方法进行托管，实现了便捷的代码合并和版本管理，为协作开发提供了有力支持。

### 7.4项目总结

本实践项目基本实现了前期构想的功能，在使用体验上达到了较为直观、简介的目的，页面设计较为美观、突出了同济大学特色。在技术层面采用了主流的Web开发技术，通过Django搭配Bootstrap的组合，结合阿里云服务器、Mysql等技术平台进行了综合运用。最终呈现出一个功能完备、实用性强的网站。但是，在开发过程中也有一些遗憾。因为早前规划功能过多，所以在时间限制下一部分功能保留了数据库设计和后端接口，但只能预备以后进行拓展，同时，为了在时间限制条件下实现复杂的功能，在网站开发过程中技术倾向相对保守。而这些遗憾之处相反也让实践学习的体会更加立体。



