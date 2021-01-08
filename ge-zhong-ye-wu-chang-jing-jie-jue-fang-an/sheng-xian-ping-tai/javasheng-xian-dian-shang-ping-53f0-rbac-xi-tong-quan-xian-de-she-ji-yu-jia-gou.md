# Java生鲜电商平台-RBAC系统权限的设计与架构

说明:根据上面的需求描述以及对需求的分析，我们得知通常的一个**中小型系统对于权限系统所需实现的功能**以及非功能性的需求，在下面我们将根据需求从技术角度上分析实现的策略以及基于目前两种比较流行的权限设计思想来讨论关于权限系统的实现。

## 1.技术策略

### 身份认证

在B/S的系统中，为识别用户身份，通常使用的技术策略为将用户的身份记录在Session中，也就是当用户登录时即获取用户的身份信息，并将其记录到Session里，当需要进行身份认证的时候通过从Session中获取用户的身份信息来实现用户的身份认证。

### 资源权限校验

资源权限校验取决于系统的授权模型，这块将在之后进行详细的阐述。

### 数据权限校验

数据权限校验取决于系统的授权模型，这块将在之后进行详细的阐述。

### 授权模型

授权模型作为权限系统的核心，从本质上决定了权限系统的易用性，这个易用性包括权限的授予和权限的校验，并同时也决定了权限的继承，权限的排斥和包含等方面的实现。

在经历了这么多年的发展，授权模型在目前中小型应用系统接受的比较多的主要有RBAC模型和ACL模型，将在之后展开专门的篇幅进行讲解。

### 权限校验的体现

权限校验的体现在中小型系统中体现出来的通常只是对于系统菜单、按钮显示的控制和对于拥有权限的数据的访问上。

它们共同依赖于资源权限校验和数据权限校验，对于系统菜单、按钮的显示上的控制在B/S中通常采用的技术策略为在生成菜单、按钮的Html时做权限级的判断，当操作主体不具备权限时则不生成该菜单、按钮的Html，从技术角度分析为方便使用者，避免使用者调用权限校验接口，通常的做法为提供菜单、按钮的标签，通过此标签生成的菜单和按钮即为经过权限过滤的。

### 高性能

为提高权限系统在授权以及校验权限时的性能，通常的做法为采用缓存技术以及加强权限系统的管理建设，加强权限系统的管理建设有助于建立一个最为适合需求的权限结构，同时做到了简化系统权限授予。

### 安全性

安全性方面来讲在B/S系统中通常有两个方面需要控制：

#### 1.通过非法途径访问系统文件

在Java的Web应用中通常采用的技术策略为将需要受保护的文件放入WEB-INF文件夹中，大家都知道在WEB-INF下的文件除了在服务器上能直接访问外，通过普通的URL是无法访问到的。

其次的做法为做Filter，即对需要受保护的资源做访问的Filter，如操作者不具备权限则直接报出错误。

#### 2.通过非法途径访问系统操作

通常采用的技术策略为对每个直接暴露对外的需要受权限保护的对象做操作级别的权限控制，简单来说在Web系统中通常采用MVC框架来实现，通常Command层是直接对外的，为防止用户通过URL或其他方式访问Command，从技术上我们需要考虑对现有系统的尽量少的侵入性，所以通常采用的做法是在Command之上做Before Interceptor或Proxy，在此Interceptor或Proxy中做权限的校验，以确认操作者具有相应的权限。

经过上面的描述，我们已经基本了解到满足权限系统需求的技术实现策略，从中我们也可以看出权限系统中最为重要的为授权模型，由于权限系统的通用性，在业界也是推出了不少的授权模型，在这里我们已目前比较通用的两种授权模型来具体讲解权限系统的完整实现。

## 2.基于RBAC的实现

### 2.1.RBAC介绍

RBAC模型作为目前最为广泛接受的权限模型，在此也将对其模型进行简要的介绍，RBAC模型成功的经典应用案例当属Unix系统了。

NIST（The National Institute of Standards and Technology，美国国家标准与技术研究院）标准RBAC模型由4个部件模型组成，这4个部件模型分别是基本模型RBAC0（Core RBAC）、角色分级模型RBAC1（Hierarchal RBAC）、角色限制模型RBAC2（Constraint RBAC）和统一模型RBAC3（Combines RBAC）\[1\]。RBAC0模型如图1所示。

#### 图表 1 RBAC 0模型

1.RBAC0定义了能构成一个RBAC控制系统的最小的元素集合  
在RBAC之中,包含用户users\(USERS\)、角色roles\(ROLES\)、目标objects\(OBS\)、操作operations\(OPS\)、许可权permissions\(PRMS\)五个基本数据元素，权限被赋予角色,而不是用户，当一个角色被指定给一个用户时，此用户就拥有了该角色所包含的权限。会话sessions是用户与激活的角色集合之间的映射。RBAC0与传统访问控制的差别在于增加一层间接性带来了灵活性，RBAC1、RBAC2、RBAC3都是先后在RBAC0上的扩展。

2.RBAC1引入角色间的继承关系  
角色间的继承关系可分为一般继承关系和受限继承关系。一般继承关系仅要求角色继承关系是一个绝对偏序关系，允许角色间的多继承。而受限继承关系则进一步要求角色继承关系是一个树结构。

3.RBAC2模型中添加了责任分离关系  
RBAC2的约束规定了权限被赋予角色时,或角色被赋予用户时,以及当用户在某一时刻激活一个角色时所应遵循的强制性规则。责任分离包括静态责任分离和动态责任分离。约束与用户-角色-权限关系一起决定了RBAC2模型中用户的访问许可。

4.RBAC3包含了RBAC1和RBAC2  
既提供了角色间的继承关系，又提供了责任分离关系。

### 2.2.实现方案

通过上面章节对RBAC的介绍，从RBAC模型中我们可以看出它已经实现了一个使用起来很方便的授权模型，并同时也就权限的继承，权限的排斥和包含提出了解决的模型。

那么现在的关键是我们需要来看看基于RBAC到底是怎么实现权限系统的需求的呢？在这里我们针对在技术策略中未描述的授权模型和权限校验部分做实现方案的讲解。

1.授权模型  
授权模型遵循RBAC进行搭建，即建立如上图表一的模型。

针对授权模型中的几个关键部分我们进行描述：

2.授权  
按照RBAC的模型，在授权时分为配置资源以及资源的操作、授予角色对资源的操作权限、分配角色给用户这几个步骤来完成。

**从这几个步骤我们进行分析：**

1.配置资源以及资源的操作

实现这步非常的简单，直接维护资源以及资源操作两个对象的持久即可实现。

2.授予角色对资源的操作权限

实现这步同样非常的简单，维护角色与资源的关联模型即可。

3.分配角色给用户

实现这步同样非常的简单，维护角色与用户的关联模型即可。

4.权限的继承

权限的继承在RBAC的模型中通过增加角色的自关联来实现，即角色可拥有子角色，子角色继承父角色的权限。

按照此模型可以看出在授权时维护权限的继承也是非常的简单，维护角色的自关联模型即可。

5.权限的排斥和包含

权限的排斥和包含这块我没有具体看RBAC的规范，通常的做法是通过在资源的操作权限模型中增加自关联模型以定义哪些资源的操作权限是排斥和包含的，在授权时可以看到同样需要维护的只是资源权限的自关联模型。

6.资源权限校验

根据上面的授权模型，在做资源权限校验的时候需要经过以下步骤：

7.判断用户所在的角色是否拥有对资源进行操作的权限

获取用户所拥有的角色，遍历其角色，以各角色建立Session，并通过类似的role.doPrivilege\(Resource,Operation\)的方式来判断该角色是否具备权限，如具备则直接返回，如不具备则直到遍历结束。

8.递规用户所在角色的父角色判断是否拥有对资源进行操作的权限

当遍历完用户本身的角色得到用户不具备对资源进行该操作的权限时，则开始递规其所在角色的父角色来判断是否拥有对资源进行操作的权限，过程同上，如确定某角色具备，则返回，如不具备直到递规结束。

9.数据权限校验

在RBAC模型中没有明确定义数据权限的实现策略，鉴于此首先要讲解下基于RBAC模型的数据授权模型的建立，基于RBAC模型，将数据映射为RBAC中的资源，对数据的操作则映射为资源的操作，同样的是将此资源以及资源的操作构成的权限授予给角色，将用户分配给角色完成数据权限的授权过程。

但根据数据权限校验的需求，数据的权限也是需要继承的，而且数据权限的授予对象需要是多种，这样的话就对上面根据RBAC映射形成的数据权限的授权模型造成了冲击，需要重构上面的授权模型来满足需求。

为实现数据权限的继承，需要将RBAC模型中的资源重构为允许自关联的模型，为实现数据权限能够授予给多种对象，需要将本来资源操作权限授予给角色的模型演变为数据操作权限授予给角色、组织机构或具体人员，根据RBAC模型，同样的建立一个中间对象，此对象和数据操作权限所授予的对象做1对多的关联，在经过这样的重构之后数据权限的授权模型就形成了，也满足了数据权限的继承和授予给多种对象的需求。

**图表 2 基于RBAC演变的数据权限模型**

上面的图中少画了数据的自关联。  
根据上面的数据权限模型，来看看数据权限的校验是怎么样去实现呢？

在做数据权限校验的时候我们需要实现的为两种方式，一种是获取操作主体具有数据操作权限的全部数据，另外一种为分页获取操作主体具有数据操作权限的数据。

就这两种方式分别来进行阐述：

1.获取操作主体具有数据操作权限的全部数据

从数据库中获取所有数据，遍历取出的数据从数据权限模型中获取相应的拥有数据操作权限的权限拥有者，如果该数据未配置数据操作权限的控制，那么就无需对该数据进行权限级的判断，如配置了，则需判断当前用户是否在该数据操作权限所对应的拥有者中，如用户不在，则需递规获取该数据的父数据的操作权限的拥有者，到用户拥有权限或递规结束时终止。

2.分页获取操作主体具有数据操作权限的数据

分页的做法和上面差不多，只是在获取了所有的数据后在内存中做分页返回。

### 2.3.优缺点分析

从上面的基于RBAC的实现方案中可以看出基于RBAC模型的优点在于：

1.易用和高效的授权方式

用户在进行授权时只需对角色进行授权，之后将相应的角色分配给用户即可。

2.简便和高效的授权模型维护

在技术角度来讲，进行授权模型的维护上因为基本只需要维护关联模型而显得简单而高效。

缺点在于：

1.复杂的权限校验

在进行权限校验时需要不断的遍历和递规，造成了性能的影响。

2.对于数据权限的不够支持

没有明确的数据权限模型，可以看到在经过重构的数据权限模型其实已经和RBAC模型有一定的  
出入，而且在数据权限的校验上实现起来是非常的低效。

最终根据说明，设计如下系统架构图：  
![](/static/image/641237-20180601092237191-1563045597.png)

## 3.设计如下

### 1.设计规则定义

数据库表明定义规则：t_ 模块名_表名

### 2.权限管理系统系统设计

| **数据项**项 | **字段名称** | **数据定义** | **必输项** | **检查规则** |
| :--- | :--- | :--- | :--- | :--- |
| userId | 用户ID | Varchar\(20\) | 系统产生\(UUID\) | 单表唯一 |
| userName | 用户名称 | Varchar\(8\) | 手工输入 | 不可为空 |
| userAccount | 用户帐户 | Varchar\(20\) | 手工输入 | 全系统唯一 |
| userMobile | 移动电话 | Varchar\(11\) | 可以为空 |  |
| isAdmin | 管理员标志 | **  DECIMAL**\(1,0\) | 系统产生 | 1、超级管理员2、企业管理员3用户 |
| lockFlag | 帐号锁定标志 | **  DECIMAL**\(1,0\) | 系统产生 | 是否锁定 1、锁住　0、未锁住锁定后的用户不能登录系统 |
| departId | 所属部门ID | Varchar\(20\) | 手动选择 | 来源t\_sys\_org表\(orgId\) |
| departName | 部门名称 | Varchar\(30\) | 系统产生 |  |
| orgId | 企业ID | Varchar\(20\) | 系统产生 | 来源t\_sys\_org表\(orgId\) |
| orgName | 企业名称 | Varchar\(30\) | 系统产生 | 来源t\_sys\_org表\(orgName\) |
| createBy | 创建人ID | Varchar\(20\) | 系统产生 |  |
| createName | 创建人名称 | Varchar\(30\) | 系统产生 |  |
| createTime | 创建时间 | **DATETIME** | 系统产生 |  |
| updateBy | 更新人ID | Varchar\(20\) | 系统产生 |  |
| updateName | 更新人名称 | Varchar\(30\) | 系统产生 |  |
| lastUpdateTime | 更新时间 | **DATETIME** | 系统产生 |  |

    CREATE TABLE `t_sys_userInfo` (

    `userId` VARCHAR(20) NOT NULL COMMENT '用户主键',

    `userName` VARCHAR(20) NOT NULL,

    `orgId` VARCHAR(20) NOT NULL COMMENT '所属机构ID',

    `orgName` VARCHAR(50) NOT NULL COMMENT '所属机构名称',

    `orgPath` VARCHAR(120) NOT NULL COMMENT '所属机构路径',

    `userAccount` VARCHAR(16) NOT NULL COMMENT '登录帐号',

    `userPwd` VARCHAR(16) NOT NULL COMMENT '登录密码',

    `userMobile` VARCHAR(11) NULL DEFAULT NULL COMMENT '移动电话',

    `isAdmin` DECIMAL(1,0) NOT NULL DEFAULT '3' COMMENT '1、超级管理员2、企业管理员3用户',

    `lockFlag` DECIMAL(1,0) NOT NULL COMMENT '是否锁住 1、锁住　0、未锁住',

    `departId` VARCHAR(20) NOT NULL COMMENT '所属部门',

    `createBy` VARCHAR(20) NOT NULL,

    `createName` VARCHAR(50) NOT NULL COMMENT '创建人',

    `createTime` DATETIME NOT NULL COMMENT '创建时间',

    `lastUpdateBy` VARCHAR(32) NOT NULL COMMENT '最后修改人ID',

    `updateName` VARCHAR(50) NOT NULL COMMENT '最后修改人',

    `lastUpdateTime` DATETIME NOT NULL COMMENT '最后修改时间',

    PRIMARY KEY (`userId`)

    )COMMENT='用户信息表'COLLATE='utf8_general_ci'ENGINE=InnoDB;

#### 2.1.用户管理\(t\_sys\_userInfo\)

| **数据项** | **字段名称** | **数据定义** | **必输项** | **检查规则** |
| :--- | :--- | :--- | :--- | :--- |
| resId | 用户ID | Varchar\(20\) | 系统产生\(UUID\) | 单表唯一 |
| resName | 用户名称 | Varchar\(20\) | 手工输入 | 不可为空 |
| parentId | 用户帐户 | Varchar\(20\) | 手工输入 | 来源t\_sys\_resInfo表\(resId\) |
| parentName | 移动电话 | Varchar\(20\) | 可以为空 | 来源t\_sys\_resInfo表\(resName\) |
| resType | 资源类型 | **  DECIMAL**\(1,0\) | 手动选择 | 1、目录2、菜单 3.操作权限检查规则：1、目录下面，不能添加操作权限2、菜单下面不能添加目录3、以此类推 |
| resState | 资源状态 | **  DECIMAL**\(1,0\) | 手动选择 | 0，不可用1可用 |
| resSort | 资源排序字段 | **DECIMAL**\(1,0\) | 系统产生 | 来源t\_sys\_org表\(orgId\) |
| resUrl | 资源访问地址 | Varchar\(30\) | 手动输入 | 后期自动产生 |
| leafCount | 子级菜单数量 | **DECIMAL**\(1,0\) | 系统产生 |  |
| resPath | 企业名称 | Varchar\(30\) | 系统产生 | 来源t\_sys\_org表\(orgName\) |
| iconCls | 创建人ID | Varchar\(32\) | 系统产生 |  |

    CREATE TABLE `t_sys_resInfo` (

    `resId` VARCHAR(32) NOT NULL COMMENT '主键ID',

    `resName` VARCHAR(128) NOT NULL COMMENT '资源名称',

    `parentId` VARCHAR(32) NOT NULL COMMENT '资源父节点',

    `parentName` VARCHAR(128) NOT NULL COMMENT '资源父节点名称',

    `resType` DECIMAL(1,0) NULL DEFAULT '1' COMMENT '1、功能菜单　２、功能点　３、按钮',

    `resState` DECIMAL(1,0) NULL DEFAULT '1' COMMENT '记录资源的状态(可用，不可用):0表示不可用，1表示可用',

    `resSort` DECIMAL(8,0) NULL DEFAULT '0' COMMENT '资源排序 ',

    `resUrl` VARCHAR(512) NULL DEFAULT '' COMMENT '资源URL地址',

    `leafCount` DECIMAL(1,0) NULL DEFAULT '0' COMMENT '子节点数量',

    `resPath` VARCHAR(2048) NULL DEFAULT '' COMMENT '记录资源节点的路径：当前菜单的上级菜单，上级菜单的父级菜单',

    `iconCls` VARCHAR(20) NULL DEFAULT '' COMMENT '资源图标',

    PRIMARY KEY (`resId`)

    )COMMENT='系统资源'COLLATE='utf8_general_ci'ENGINE=InnoDB;

#### 2.2.组织机构表\(t\_sys\_org\)

    CREATE TABLE `t_sys_org` (

    `orgId` VARCHAR(20) NOT NULL COMMENT '组织主键',

    `orgName` VARCHAR(50) NOT NULL COMMENT '组织名称',

    `orgShortName` VARCHAR(30) NULL DEFAULT NULL COMMENT '组织简称',

    `parentId` VARCHAR(32) NOT NULL COMMENT '组织父节点ID',

    `parentName` VARCHAR(50) NOT NULL COMMENT '父节点编码',

    `orgType` DECIMAL(1,0) NOT NULL DEFAULT '1' COMMENT '组织的类型 1、企业 2、部门 3、分公司,4,支行',

    `orgStatus` DECIMAL(1,0) NOT NULL DEFAULT '1' COMMENT '记录组织的状态是否启用：0，停用/1，启用',

    `orgHisStatus` DECIMAL(1,0) NOT NULL DEFAULT '1' COMMENT '0，停用/1，启用',

    `orgLinkMan` VARCHAR(50) NULL DEFAULT NULL COMMENT '机构的法人代表名称',

    `orgTelephone` VARCHAR(20) NULL DEFAULT NULL COMMENT '机构的联系电话',

    `orgFax` VARCHAR(20) NULL DEFAULT NULL COMMENT '机构的传真',

    `orgEmail` VARCHAR(100) NULL DEFAULT NULL COMMENT '机构的电子邮箱',

    `orgAddress` VARCHAR(200) NULL DEFAULT NULL COMMENT '组织地址',

    `orgWebsite` VARCHAR(200) NULL DEFAULT NULL COMMENT '组织的网站',

    `leafCount` DECIMAL(1,0) NULL DEFAULT NULL COMMENT '子企业数量',

    `orgSort` DECIMAL(10,0) NULL DEFAULT NULL COMMENT '组织排序',

    `resIconStyle` VARCHAR(20) NULL DEFAULT '' COMMENT '资源图标',

    `orgPath` VARCHAR(2048) NULL DEFAULT NULL COMMENT '记录组织的节点路径',

    `createBy` VARCHAR(20) NULL DEFAULT NULL COMMENT '创建人ID',

    `createName` VARCHAR(50) NULL DEFAULT NULL COMMENT '创建人',

    `createTime` DATETIME NULL DEFAULT NULL COMMENT '创建时间',

    `lastUpdateBy` VARCHAR(20) NULL DEFAULT NULL COMMENT '最后修改人ID',

    `updateName` VARCHAR(50) NULL DEFAULT NULL COMMENT '最后修改人',

    `lastUpdateTime` DATETIME NULL DEFAULT NULL COMMENT '最后修改时间',

    PRIMARY KEY (`orgId`)

    )

    COMMENT='组织架构信息'

    COLLATE='utf8_general_ci'

    ENGINE=InnoDB

    ;

#### 2.3.组织角色表\(t\_sys\_role\)

    CREATE TABLE `t_sys_role` (

    `roleId` VARCHAR(20) NOT NULL COMMENT '角色表主键ID',

    `orgId` VARCHAR(20) NULL DEFAULT NULL COMMENT '所属组织',

    `orgPath` VARCHAR(20) NULL DEFAULT NULL COMMENT '组织路径',

    `orgName` VARCHAR(50) NOT NULL COMMENT '组织名称',

    `roleName` VARCHAR(50) NOT NULL COMMENT '角色名称',

    `roleDesc` VARCHAR(100) NULL DEFAULT NULL COMMENT '角色描述',

    `createBy` VARCHAR(20) NOT NULL COMMENT '创建人ID',

    `createName` VARCHAR(50) NOT NULL COMMENT '创建人',

    `createTime` DATETIME NOT NULL COMMENT '创建时间',

    `lastUpdateBy` VARCHAR(20) NULL DEFAULT NULL COMMENT '最后修改人ID',

    `updateName` VARCHAR(50) NULL DEFAULT NULL COMMENT '最后修改人',

    `lastUpdateTime` DATETIME NULL DEFAULT NULL COMMENT '最后修改时间',

    PRIMARY KEY (`roleId`)

    )

    COMMENT='角色信息表'

    COLLATE='utf8_general_ci'

    ENGINE=InnoDB

    ;

#### 2.4.用户角色表\(t\_sys\_userRole\)

    CREATE TABLE `t_sys_userRole` (

    `userRoleId` VARCHAR(20) NOT NULL COMMENT '主键ID',

    `roleId` VARCHAR(20) NULL DEFAULT NULL COMMENT '角色表主键ID',

    `userId` VARCHAR(20) NULL DEFAULT NULL COMMENT '用户主键',

    PRIMARY KEY (`userRoleId`)

    )

    COMMENT='用户角色关系表'

    COLLATE='utf8_general_ci'

    ENGINE=InnoDB;

#### 2.5.角色资源表\(t\_sys\_roleRes\)

    CREATE TABLE `t_sys_roleRes` (

    `roleResId` VARCHAR(20) NOT NULL COMMENT '角色资源主键ID',

    `roleId` VARCHAR(22) NOT NULL  COMMENT '角色表主键ID',

    `resId` VARCHAR(20) NOT NULL  COMMENT '资源主键',

    `isReadWrite` DECIMAL(1,0) NOT NULL DEFAULT 1 COMMENT '1,只读取2，可读写',

    PRIMARY KEY (`roleResId`)

    )

    COMMENT='角色资源关系表'

    COLLATE='utf8_general_ci'

    ENGINE=InnoDB

#### 2.6.企业资源表\(t\_sys\_orgRes\)

    CREATE TABLE `t_sys_orgRes` (

    `orgResId` VARCHAR(20) NOT NULL COMMENT '角色资源主键ID',

    `orgId` VARCHAR(22) NOT NULL  COMMENT '角色表主键ID',

    `resId` VARCHAR(20) NOT NULL   COMMENT '资源主键',

    `isReadWrite` DECIMAL(1,0) NOT NULL DEFAULT 1 COMMENT '1,只读取2，可读写',

    PRIMARY KEY (`orgResId`)

    )

    COMMENT='角色资源关系表'

    COLLATE='utf8_general_ci'

    ENGINE=InnoDB


## 相关运营截图：
641237-20180601092621749-1558672373.png

