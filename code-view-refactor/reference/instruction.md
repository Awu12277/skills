# 前端代码优化与重构详尽指南

**参考：《重构 改善既有代码的设计》第二版（Martin Fowler 著）**

---

## 目录

1. 重构基础概念
2. 代码的坏味道
3. 重构前置条件与步骤
4. HTML 重构技术
5. CSS 重构技术
6. JavaScript 重构技术
7. 函数级别重构
8. 对象与类级别重构
9. 异步代码重构
10. 测试与验证

---

## 第一部分：重构基础概念

### 1.1 什么是重构

重构（Refactoring）是在不改变代码**外部行为**的前提下，对代码内部结构进行整理，使其更加**清晰、可读、易于维护**。

**重构的两顶帽子：**
```
┌─────────────────────────────────────────┐
│              行为不变的前提下              │
│         改善既有代码的内部结构              │
└─────────────────────────────────────────┘
         重构 ≠ 添加新功能
         重构 ≠ 修复bug
         重构 ≠ 重写
```

### 1.2 重构的核心原则

根据《重构》第二版，重构的核心原则包括：

| 原则 | 说明 |
|------|------|
| **小步前进** | 每次只做一个修改，立即测试 |
| **童子军规则** | 离开时比来时代码更干净 |
| **三次法则** | 第一次做某事，情有可原；第二次做类似的事，皱眉；第三次做同样的事，必须重构 |
| **不要在重构时添加功能** | 重构和添加功能不要同时进行 |
| **让编译器和测试来发现问题** | 确保每次修改后代码仍能正常运行 |

### 1.3 为何重构

```
重构能够：
├── 改进代码结构
│     ├── 消除重复代码
│     ├── 简化复杂逻辑
│     └── 改善命名
│
├── 提升可读性
│     ├── 使代码自解释
│     ├── 减少注释需求
│     └── 便于团队协作
│
├── 便于维护
│     ├── 降低修改成本
│     ├── 减少引入bug的风险
│     └── 加速新功能开发
│
└── 改善性能（副作用）
      ├── 减少不必要的计算
      ├── 优化数据结构
      └── 提升运行效率
```

---

## 第二部分：代码的坏味道（Bad Smells）

《重构》书中指出，当代码出现以下"坏味道"时，应当考虑重构。

### 2.1 坏味道速查表

```
┌────────────────────────────────────────────────────────────────────┐
│                          代码坏味道清单                              │
├──────────────┬─────────────────────────────────────────────────────┤
│ 坏味道名称    │                    具体表现                          │
├──────────────┼─────────────────────────────────────────────────────┤
│ 重复代码      │ 同样的逻辑出现多次                                   │
│ 过长函数      │ 函数超过30行，难以理解                               │
│ 过大类        │ 类承担过多职责                                       │
│ 过长参数列表  │ 函数参数超过3个                                       │
│ 发散式变化    │ 一个类常因不同原因在不同方向变化                       │
│ 霰弹式修改    │ 某类变化需要修改多个类                                 │
│ 依恋情节      │ 函数对其他类的兴趣超过所在类                           │
│ 数据泥团      │ 总是同时出现的一组数据                                 │
│ 基本类型偏执  │ 滥用基本类型而非创建专用类型                           │
│ switch语句    │ 过多switch/if-else链                                  │
│ 平行继承体系  │ 继承树平行增长                                         │
│ 冗余类        │ 没有存在价值的类                                       │
│ 夸夸其谈的未来│ 为了未来可能的需求添加代码                             │
│ 令人迷惑的暂时│ 代码只在某个特定情况下有用                             │
│ 过度耦合的消息链│ 过度链式调用 obj.getA().getB().getC()                 │
│ 中间人        │ 类大部分工作委托给其他类                               │
│ 过度使用链接  │ 过度使用链式调用                                       │
│ 内幕交易      │ 类之间交换太多私有信息                                 │
│ 异曲同工的类  │ 两个类接口相同但命名不同                               │
│ 不完整的类库  │ 类库功能不足，需要自己添加                             │
│ 数据类        │ 只有getter/setter的类                                  │
│ 被拒绝的遗赠  │ 子类不想继承父类所有方法                               │
│ 注释          │ 注释过多说明代码本身难以理解                           │
└──────────────┴─────────────────────────────────────────────────────┘
```

### 2.2 前端特有坏味道

#### HTML/CSS 坏味道

| 坏味道 | 描述 |
|--------|------|
| 冗余标签 | 过多的嵌套div，缺乏语义化 |
| 行内样式 | style属性散落在HTML中 |
| 选择器过于复杂 | 嵌套层数过深，如 `.a .b .c .d .e` |
| 魔法数字 | 硬编码的尺寸、颜色等值 |
| 重复样式规则 | 相同样式出现多次 |
| 魔法选择器 | `.container .box ul li a span:hover` |
| 未使用的CSS | 定义但未引用的样式 |

#### JavaScript 坏味道

| 坏味道 | 描述 |
|--------|------|
| 全局变量污染 | 过多var声明的全局变量 |
| 回调地狱 | 深层嵌套的回调函数 |
| 字符串拼接 | 使用+拼接而非模板字符串 |
| 魔法字符串 | 硬编码的条件判断字符串值 |
| 事件绑定分散 | 每个元素单独绑定事件 |
| DOM查询重复 | 相同的DOM查询多次执行 |
| 类型转换混乱 | 隐式类型转换缺乏一致性 |

---

## 第三部分：重构前置条件与步骤

### 3.1 重构前置条件

根据《重构》第二版，进行重构前必须确保：

```
前置条件检查清单：
├── 版本控制就绪
│     ├── 代码已提交到版本控制系统
│     └── 可以随时回滚
│
├── 测试覆盖
│     ├── 有现有的测试用例
│     └── 能够快速验证功能正确性
│
├── 环境稳定
│     ├── 了解代码的运行机制
│     └── 明确哪些是外部行为不能改变
│
└── 目标明确
      ├── 知道为什么要重构
      └── 明确重构的范围
```

### 3.2 重构的标准步骤

```
重构执行流程：

  ┌─────────────────────────────────────┐
  │ Step 1: 确认重构范围                 │
  │  - 确定要重构的文件/模块             │
  │  - 明确重构目标                      │
  └──────────────┬──────────────────────┘
                 ▼
  ┌─────────────────────────────────────┐
  │ Step 2: 编写/运行测试                │
  │  - 确保现有功能有测试覆盖             │
  │  - 记录当前行为                      │
  └──────────────┬──────────────────────┘
                 ▼
  ┌─────────────────────────────────────┐
  │ Step 3: 小步重构                     │
  │  - 每次只做一个修改                  │
  │  - 编译/运行测试验证                 │
  └──────────────┬──────────────────────┘
                 ▼
  ┌─────────────────────────────────────┐
  │ Step 4: 提交代码                    │
  │  - 使用原子提交                      │
  │  - 描述本次重构内容                  │
  └──────────────┬──────────────────────┘
                 ▼
  ┌─────────────────────────────────────┐
  │ Step 5: 重复 Step 3-4               │
  │  - 直到完成所有重构目标              │
  └─────────────────────────────────────┘
```

### 3.3 重构的心理模型

《重构》书中强调的重构心态：

```
┌────────────────────────────────────────────────────────┐
│                     重构心态                             │
│                                                        │
│   "在代码运行正常的前提下，持续改善代码结构"              │
│                                                        │
│   ✗ 不要：                                             │
│     - 想着"反正要重写，先凑合用"                         │
│     - 一次性做太多改动                                  │
│     - 重构和功能开发同时进行                            │
│                                                        │
│   ✓ 应该：                                             │
│     - 每次只做一个小改动                                │
│     - 每次改动后验证代码仍能工作                        │
│     - 保持稳定的节奏，逐步改善                          │
└────────────────────────────────────────────────────────┘
```

---

## 第四部分：HTML 重构技术

### 4.1 语义化重构

**问题代码：**
```html
<div class="header">
  <div class="logo"></div>
  <div class="nav">
    <div class="nav-item"><a href="#">首页</a></div>
    <div class="nav-item"><a href="#">关于</a></div>
  </div>
</div>
```

**重构后：**
```html
<header>
  <div class="logo"></div>
  <nav>
    <a href="#">首页</a>
    <a href="#">关于</a>
  </nav>
</header>
```

**语义化标签对应表：**

| 场景 | 推荐标签 |
|------|----------|
| 页面头部 | `<header>` |
| 导航链接 | `<nav>` |
| 主要内容 | `<main>` |
| 文章/博客 | `<article>` |
| 区块/章节 | `<section>` |
| 侧边栏 | `<aside>` |
| 页面底部 | `<footer>` |
| 标题组 | `<hgroup>` |
| 独立内容 | `<figure>` + `<figcaption>` |
| 时间显示 | `<time datetime="2024-01-01">` |

### 4.2 减少不必要的标签嵌套

**问题代码：**
```html
<div class="container">
  <div class="title">
    <span class="text">标题文本</span>
  </div>
</div>
```

**重构后：**
```html
<div class="container">
  <h1 class="title">标题文本</h1>
</div>
```

### 4.3 合并分散的属性

**问题代码：**
```html
<div class="user-card" data-id="123" data-role="admin" data-active="true">
</div>
```

**重构后：**
```html
<div class="user-card" data-user='{"id":123,"role":"admin","active":true}'>
</div>
```

### 4.4 表单结构优化

**问题代码：**
```html
<div class="form-group">
  <div class="label">用户名</div>
  <input type="text" class="input" />
</div>
```

**重构后：**
```html
<div class="form-group">
  <label for="username">用户名</label>
  <input type="text" id="username" name="username" />
</div>
```

### 4.5 图片使用优化

**问题代码：**
```html
<img src="loading.gif" width="100" height="100" />
```

**重构后：**
```html
<img src="avatar.webp" alt="用户头像" width="100" height="100" loading="lazy" decoding="async" />
```

---

## 第五部分：CSS 重构技术

### 5.1 提取重复样式——合并相似规则

**问题代码：**
```css
.header {
  background-color: #ffffff;
  padding: 20px;
  margin-bottom: 20px;
  border-radius: 8px;
}

.footer {
  background-color: #ffffff;
  padding: 20px;
  margin-top: 20px;
  border-radius: 8px;
}

.sidebar {
  background-color: #ffffff;
  padding: 20px;
  border-radius: 8px;
}
```

**重构后（提取公共样式）：**
```css
/* 公共样式 */
.card {
  background-color: #ffffff;
  padding: 20px;
  border-radius: 8px;
}

.header {
  margin-bottom: 20px;
}

.footer {
  margin-top: 20px;
}
```

### 5.2 使用 CSS 变量——消除魔法数字

**问题代码：**
```css
.button-primary {
  padding: 15px 30px;
  font-size: 16px;
  color: #1890ff;
  border-color: #1890ff;
}

.button-secondary {
  padding: 15px 30px;
  font-size: 16px;
  color: #52c41a;
  border-color: #52c41a;
}
```

**重构后：**
```css
:root {
  --btn-padding-x: 30px;
  --btn-padding-y: 15px;
  --btn-font-size: 16px;
  --color-primary: #1890ff;
  --color-success: #52c41a;
}

.button {
  padding: var(--btn-padding-y) var(--btn-padding-x);
  font-size: var(--btn-font-size);
}

.button-primary {
  color: var(--color-primary);
  border-color: var(--color-primary);
}

.button-secondary {
  color: var(--color-success);
  border-color: var(--color-success);
}
```

**⚠️ 注意：在 Vue scoped 样式中使用 `:root` 的问题**

在 Vue 单文件组件的 `<style scoped>` 中使用 `:root` 定义 CSS 变量可能不会生效，因为 scoped 样式会给所有选择器添加属性选择器限定，导致 `:root` 的全局作用域被限制。

**正确做法：** 将 CSS 变量定义在父级容器选择器上（如 `.wrap`、`.container`），而非 `:root`。

```scss
/* ❌ 错误：在 scoped 中 :root 可能不生效 */
<style scoped>
:root {
  --color-primary: #0066cc;
}
</style>

/* ✅ 正确：定义在父容器上 */
<style scoped>
.wrap {
  --color-primary: #0066cc;
  background-color: var(--color-bg-light);
}
</style>
```

### 5.3 简化选择器——消除过深嵌套

**问题代码：**
```css
.container .content .sidebar .nav-list .nav-item .nav-link:hover {
  color: #1890ff;
}
```

**重构方法一（使用class直接选择）：**
```css
.nav-link:hover {
  color: #1890ff;
}
```

**重构方法二（使用BEM命名规范）：**
```css
.nav__link:hover {
  color: #1890ff;
}
```

### 5.4 提取组件样式

**问题代码：**
```css
.product-card-1 {
  border: 1px solid #e8e8e8;
  border-radius: 8px;
  overflow: hidden;
  background: #fff;
}

.product-card-2 {
  border: 1px solid #e8e8e8;
  border-radius: 8px;
  overflow: hidden;
  background: #fff;
}
```

**重构后（定义可复用组件）：**
```css
/* 组件定义 */
.card {
  border: 1px solid #e8e8e8;
  border-radius: 8px;
  overflow: hidden;
  background: #fff;
}

/* 组件变体 */
.product-card {
  /* 产品卡片特有样式 */
}

.user-card {
  /* 用户卡片特有样式 */
}
```

### 5.5 简化条件样式

**问题代码：**
```css
.sidebar {
  width: 300px;
}

.container-fluid .sidebar {
  width: 350px;
}

.dark-theme .sidebar {
  background: #333;
}

.container-fluid .dark-theme .sidebar {
  background: #222;
}
```

**重构后：**
```css
.sidebar {
  width: var(--sidebar-width, 300px);
  background: var(--sidebar-bg, #fff);
}

/* 变体通过单独的选择器处理 */
.sidebar--wide {
  width: 350px;
}

.sidebar--dark {
  background: #333;
}
```

### 5.6 动画性能优化

**问题代码：**
```css
.animated-element {
  animation: move 1s ease-in-out;
}

@keyframes move {
  from {
    left: 0;
    opacity: 0;
  }
  to {
    left: 100px;
    opacity: 1;
  }
}
```

**重构后（使用GPU加速属性）：**
```css
.animated-element {
  animation: move 1s ease-in-out;
  will-change: transform, opacity;
}

@keyframes move {
  from {
    transform: translateX(0);
    opacity: 0;
  }
  to {
    transform: translateX(100px);
    opacity: 1;
  }
}
```

### 5.7 响应式断点统一

**问题代码：**
```css
@media (min-width: 768px) { /* ... */ }
@media screen and (max-width: 767px) { /* ... */ }
@media all and (min-width: 1200px) { /* ... */ }
```

**重构后：**
```css
:root {
  --breakpoint-sm: 576px;
  --breakpoint-md: 768px;
  --breakpoint-lg: 992px;
  --breakpoint-xl: 1200px;
}

@media (min-width: var(--breakpoint-md)) { /* 平板 */ }
@media (min-width: var(--breakpoint-lg)) { /* 桌面 */ }
@media (min-width: var(--breakpoint-xl)) { /* 大屏 */ }
```

---

## 第六部分：JavaScript 重构技术

### 6.1 变量声明重构

#### 6.1.1 用 const/let 替代 var

**问题代码：**
```javascript
var name = '张三';
var isActive = true;
var count = 0;
```

**重构后：**
```javascript
const name = '张三';
const isActive = true;
let count = 0; // 因为 count 会变化，所以用 let
```

#### 6.1.2 合并相关变量声明

**问题代码：**
```javascript
var width = window.innerWidth;
var height = window.innerHeight;
var devicePixelRatio = window.devicePixelRatio;
```

**重构后：**
```javascript
const { innerWidth: width, innerHeight: height, devicePixelRatio } = window;
```

### 6.2 函数重构

#### 6.2.1 拆分过长函数

根据《重构》，超过30行的函数就应该考虑拆分。

**问题代码：**
```javascript
function processUser(userData) {
  // 验证（30行）
  if (!userData.name) {
    throw new Error('用户名不能为空');
  }
  if (userData.name.length < 2) {
    throw new Error('用户名至少2个字符');
  }
  if (userData.name.length > 50) {
    throw new Error('用户名最多50个字符');
  }
  if (!userData.email) {
    throw new Error('邮箱不能为空');
  }
  // 正则验证邮箱...
  // 更多验证逻辑...

  // 处理（20行）
  const processedUser = {
    id: generateId(),
    name: userData.name.trim(),
    email: userData.email.toLowerCase(),
    createdAt: new Date()
  };

  // 保存（15行）
  saveToDatabase(processedUser);
  sendWelcomeEmail(processedUser);
  updateStatistics(processedUser);

  return processedUser;
}
```

**重构后：**
```javascript
// 拆分后的结构
function processUser(userData) {
  const validatedData = validateUserData(userData);
  const processedUser = transformUserData(validatedData);
  saveUser(processedUser);
  return processedUser;
}

function validateUserData(userData) {
  validateName(userData.name);
  validateEmail(userData.email);
  return userData;
}

function validateName(name) {
  if (!name) {
    throw new Error('用户名不能为空');
  }
  if (name.length < 2 || name.length > 50) {
    throw new Error('用户名长度需在2-50个字符之间');
  }
}

function validateEmail(email) {
  if (!email) {
    throw new Error('邮箱不能为空');
  }
  // 邮箱格式验证...
}

function transformUserData(userData) {
  return {
    id: generateId(),
    name: userData.name.trim(),
    email: userData.email.toLowerCase(),
    createdAt: new Date()
  };
}

function saveUser(user) {
  saveToDatabase(user);
  sendWelcomeEmail(user);
  updateStatistics(user);
}
```

#### 6.2.2 用函数替代复杂表达式

**问题代码：**
```javascript
function getDiscountPrice(price, vipLevel, orderCount, isHoliday) {
  // 复杂的折扣计算逻辑
  return price * (vipLevel === 'gold' ? 0.7 : vipLevel === 'silver' ? 0.85 : 1)
         * (orderCount > 100 ? 0.9 : orderCount > 50 ? 0.95 : 1)
         * (isHoliday ? 0.8 : 1);
}
```

**重构后：**
```javascript
function getDiscountPrice(price, vipLevel, orderCount, isHoliday) {
  const vipDiscount = getVipDiscount(vipLevel);
  const loyaltyDiscount = getLoyaltyDiscount(orderCount);
  const holidayDiscount = getHolidayDiscount(isHoliday);

  return price * vipDiscount * loyaltyDiscount * holidayDiscount;
}

function getVipDiscount(level) {
  const discounts = {
    gold: 0.7,
    silver: 0.85,
    bronze: 1
  };
  return discounts[level] || 1;
}

function getLoyaltyDiscount(count) {
  if (count > 100) return 0.9;
  if (count > 50) return 0.95;
  return 1;
}

function getHolidayDiscount(isHoliday) {
  return isHoliday ? 0.8 : 1;
}
```

#### 6.2.3 移除参数默认值的地方性解释

**问题代码：**
```javascript
function createUser(name, age, role = 'user', active = true) {
  // role默认为user, active默认为true
  return { name, age, role, active };
}
```

**重构后：**
```javascript
function createUser(name, age, role = 'user', active = true) {
  return { name, age, role, active };
}

// 或者如果默认值很多，使用配置对象
function createUser(name, age, options = {}) {
  const {
    role = 'user',
    active = true,
    email = null,
    avatar = null
  } = options;

  return { name, age, role, active, email, avatar };
}
```

### 6.3 条件表达式重构

#### 6.3.1 分解条件表达式

**问题代码：**
```javascript
if (date.getMonth() >= 6 && date.getMonth() <= 8 && temperature > 30 && !isRaining) {
  // 暑假高温晴天
}
```

**重构后：**
```javascript
function isSummerVacation(date) {
  const month = date.getMonth();
  return month >= 6 && month <= 8;
}

function isHotDay(temperature) {
  return temperature > 30;
}

function isSunnyWeather(isRaining) {
  return !isRaining;
}

if (isSummerVacation(date) && isHotDay(temperature) && isSunnyWeather(isRaining)) {
  // 暑假高温晴天
}
```

#### 6.3.2 合并重复的条件片段

**问题代码：**
```javascript
function getTitle(user) {
  if (user.isAdmin) {
    document.title = '管理员面板';
    return renderAdminPanel(user);
  }
  if (user.isEditor) {
    document.title = '编辑面板';
    return renderEditorPanel(user);
  }
  if (user.isAdmin) {
    document.title = '管理员面板';
    return renderAdminPanel(user);
  }
  return renderUserPanel(user);
}
```

**重构后：**
```javascript
function getTitle(user) {
  if (user.isAdmin) {
    document.title = '管理员面板';
    return renderAdminPanel(user);
  }
  if (user.isEditor) {
    document.title = '编辑面板';
    return renderEditorPanel(user);
  }
  return renderUserPanel(user);
}
```

#### 6.3.3 用卫语句替代嵌套条件

**问题代码：**
```javascript
function getPayAmount(employee) {
  let result;
  if (employee.isSeparated) {
    result = 0;
  } else {
    if (employee.isRetired) {
      result = 0;
    } else {
      result = salary * 0.8; // 正常逻辑
    }
  }
  return result;
}
```

**重构后（使用卫语句）：**
```javascript
function getPayAmount(employee) {
  if (employee.isSeparated) return 0;
  if (employee.isRetired) return 0;

  return salary * 0.8;
}
```

#### 6.3.4 用多态替代条件（Switch/If-Else链）

**问题代码：**
```javascript
function calculateArea(shape) {
  if (shape.type === 'circle') {
    return Math.PI * shape.radius * shape.radius;
  } else if (shape.type === 'rectangle') {
    return shape.width * shape.height;
  } else if (shape.type === 'triangle') {
    return 0.5 * shape.base * shape.height;
  } else if (shape.type === 'square') {
    return shape.side * shape.side;
  }
  throw new Error('Unknown shape type');
}
```

**重构后（使用多态/策略模式）：**
```javascript
class Shape {
  getArea() {
    throw new Error('Method getArea() must be implemented');
  }
}

class Circle extends Shape {
  constructor(radius) {
    super();
    this.radius = radius;
  }

  getArea() {
    return Math.PI * this.radius ** 2;
  }
}

class Rectangle extends Shape {
  constructor(width, height) {
    super();
    this.width = width;
    this.height = height;
  }

  getArea() {
    return this.width * this.height;
  }
}

// 使用
const shapes = [new Circle(5), new Rectangle(4, 6)];
const totalArea = shapes.reduce((sum, shape) => sum + shape.getArea(), 0);
```

#### 6.3.5 用查找表替代条件

**问题代码：**
```javascript
function getWeekdayName(dayNumber) {
  switch (dayNumber) {
    case 0: return '星期日';
    case 1: return '星期一';
    case 2: return '星期二';
    case 3: return '星期三';
    case 4: return '星期四';
    case 5: return '星期五';
    case 6: return '星期六';
    default: return '无效';
  }
}
```

**重构后：**
```javascript
const WEEKDAY_NAMES = [
  '星期日', '星期一', '星期二', '星期三', '星期四', '星期五', '星期六'
];

function getWeekdayName(dayNumber) {
  return WEEKDAY_NAMES[dayNumber] ?? '无效';
}
```

### 6.4 循环重构

#### 6.4.1 用数组方法替代命令式循环

**问题代码：**
```javascript
const numbers = [1, 2, 3, 4, 5];
const doubled = [];

for (let i = 0; i < numbers.length; i++) {
  doubled.push(numbers[i] * 2);
}
```

**重构后：**
```javascript
const doubled = numbers.map(n => n * 2);
```

#### 6.4.2 用 filter + map + reduce 组合替代多次循环

**问题代码：**
```javascript
// 找出活跃用户，过滤出VIP，计算VIP余额总和
const users = getUsers();

const activeUsers = [];
for (let i = 0; i < users.length; i++) {
  if (users[i].isActive) {
    activeUsers.push(users[i]);
  }
}

const vipUsers = [];
for (let i = 0; i < activeUsers.length; i++) {
  if (activeUsers[i].isVIP) {
    vipUsers.push(activeUsers[i]);
  }
}

let totalBalance = 0;
for (let i = 0; i < vipUsers.length; i++) {
  totalBalance += vipUsers[i].balance;
}
```

**重构后：**
```javascript
const totalBalance = users
  .filter(user => user.isActive && user.isVIP)
  .reduce((sum, user) => sum + user.balance, 0);
```

### 6.5 字符串处理重构

#### 6.5.1 用模板字符串替代拼接

**问题代码：**
```javascript
const message = '亲爱的 ' + userName + '，您好！\n' +
  '您的订单编号为 ' + orderId + '，\n' +
  '将于 ' + deliveryDate + ' 送达。';
```

**重构后：**
```javascript
const message = `亲爱的 ${userName}，您好！

您的订单编号为 ${orderId}，
将于 ${deliveryDate} 送达。`;
```

#### 6.5.2 提取字符串构建逻辑

**问题代码：**
```javascript
function buildQueryString(params) {
  let result = '';
  if (params.page) {
    result += 'page=' + params.page + '&';
  }
  if (params.limit) {
    result += 'limit=' + params.limit + '&';
  }
  if (params.sort) {
    result += 'sort=' + params.sort + '&';
  }
  return result.slice(0, -1);
}
```

**重构后：**
```javascript
function buildQueryString(params) {
  return Object.entries(params)
    .map(([key, value]) => `${encodeURIComponent(key)}=${encodeURIComponent(value)}`)
    .join('&');
}

// 或使用 URLSearchParams
function buildQueryString(params) {
  return new URLSearchParams(params).toString();
}
```

### 6.6 对象与数组重构

#### 6.6.1 用解构替代属性访问

**问题代码：**
```javascript
function displayUserInfo(user) {
  const name = user.name;
  const email = user.email;
  const avatar = user.avatar;
  const isAdmin = user.isAdmin;

  console.log(`用户: ${name}, 邮箱: ${email}`);
}
```

**重构后：**
```javascript
function displayUserInfo({ name, email, avatar, isAdmin }) {
  console.log(`用户: ${name}, 邮箱: ${email}`);
}
```

#### 6.6.2 用对象替代冗长参数列表

**问题代码：**
```javascript
function createButton(label, color, width, height, onClick, disabled, icon) {
  // ...
}

createButton('提交', '#1890ff', 120, 40, handleClick, false, 'check');
```

**重构后：**
```javascript
function createButton({ 
  label, 
  color = '#1890ff', 
  width = 100, 
  height = 36, 
  onClick, 
  disabled = false, 
  icon = null 
}) {
  // ...
}

createButton({
  label: '提交',
  onClick: handleClick,
  icon: 'check'
});
```

#### 6.6.3 用展开运算符合并对象

**问题代码：**
```javascript
const defaultConfig = { theme: 'light', language: 'zh', debug: false };
const userConfig = { theme: 'dark' };

const finalConfig = Object.assign({}, defaultConfig, userConfig);
```

**重构后：**
```javascript
const finalConfig = { ...defaultConfig, ...userConfig };
```

#### 6.6.4 用记录类型替代对象散列

**问题代码：**
```javascript
// 用对象存储相关数据，但容易混淆
const data = { n: '张三', a: 25, e: 'zhang@example.com' };
```

**重构后：**
```javascript
const data = {
  name: '张三',
  age: 25,
  email: 'zhang@example.com'
};
```

### 6.7 异常处理重构

#### 6.7.1 用异常替代错误码返回

**问题代码：**
```javascript
function divide(a, b) {
  if (b === 0) {
    return { success: false, error: '除数不能为零' };
  }
  return { success: true, result: a / b };
}

const result = divide(10, 0);
if (!result.success) {
  console.error(result.error);
}
```

**重构后：**
```javascript
function divide(a, b) {
  if (b === 0) {
    throw new Error('除数不能为零');
  }
  return a / b;
}

try {
  const result = divide(10, 0);
} catch (error) {
  console.error(error.message);
}
```

#### 6.7.2 提取异常处理逻辑

**问题代码：**
```javascript
try {
  saveUser(user);
} catch (error) {
  showError('保存用户失败');
  logError(error);
  reportToAnalytics(error);
}

try {
  updateSettings(settings);
} catch (error) {
  showError('更新设置失败');
  logError(error);
  reportToAnalytics(error);
}
```

**重构后：**
```javascript
function withErrorHandling(action, errorMessage) {
  try {
    return action();
  } catch (error) {
    showError(errorMessage);
    logError(error);
    reportToAnalytics(error);
    throw error;
  }
}

withErrorHandling(() => saveUser(user), '保存用户失败');
withErrorHandling(() => updateSettings(settings), '更新设置失败');
```

### 6.8 简化布尔表达式

#### 6.8.1 消除双重否定

**问题代码：**
```javascript
if (!isNotDisabled) { /* ... */ }
if (!isNotLoggedOut) { /* ... */ }
```

**重构后：**
```javascript
if (isEnabled) { /* ... */ }
if (isLoggedIn) { /* ... */ }
```

#### 6.8.2 用短路求值替代三元表达式

**问题代码：**
```javascript
const greeting = user ? user.name : 'Guest';
```

**重构后：**
```javascript
const greeting = user?.name ?? 'Guest';
```

---

## 第七部分：函数级别重构

### 7.1 函数命名重构

**问题代码：**
```javascript
function a() { /* 处理数据 */ }
function calc() { /* 计算价格 */ }
function proc() { /* 处理订单 */ }
```

**重构后：**
```javascript
function processUserData() { /* 处理数据 */ }
function calculatePrice() { /* 计算价格 */ }
function processOrder() { /* 处理订单 */ }
```

### 7.2 函数重新组织

#### 7.2.1 移动语句——让相关代码靠近

**问题代码：**
```javascript
function renderPage() {
  const data = fetchData();  // 数据获取

  setupEventListeners();    // 事件绑定

  renderHeader();           // 渲染
  renderBody(data);         // 渲染
  renderFooter();           // 渲染

  fetchData();              // 又获取了一次
}
```

**重构后：**
```javascript
function renderPage() {
  const data = fetchData(); // 只获取一次

  renderHeader();
  renderBody(data);
  renderFooter();

  setupEventListeners();
}
```

#### 7.2.2 提取函数——将意图与实现分离

**问题代码：**
```javascript
function printOwing(invoice) {
  // 打印标题
  console.log('===================');
  console.log('===== 客户账单 =====');
  console.log('===================');

  // 打印客户名
  console.log(`客户名: ${invoice.customer}`);

  // 打印金额
  let outstanding = 0;
  for (const o of invoice.orders) {
    outstanding += o.amount;
  }
  console.log(`金额: ${outstanding}`);
}
```

**重构后：**
```javascript
function printOwing(invoice) {
  printBanner();
  const outstanding = calculateOutstanding(invoice);
  printDetails(invoice.customer, outstanding);
}

function printBanner() {
  console.log('===================');
  console.log('===== 客户账单 =====');
  console.log('===================');
}

function calculateOutstanding(invoice) {
  return invoice.orders.reduce((sum, order) => sum + order.amount, 0);
}

function printDetails(customer, outstanding) {
  console.log(`客户名: ${customer}`);
  console.log(`金额: ${outstanding}`);
}
```

### 7.3 替代算法

**问题代码：**
```javascript
function foundPerson(people) {
  for (let i = 0; i < people.length; i++) {
    if (people[i] === 'Don') {
      return 'Don';
    }
    if (people[i] === 'John') {
      return 'John';
    }
    if (people[i] === 'Kent') {
      return 'Kent';
    }
  }
  return '';
}
```

**重构后：**
```javascript
function foundPerson(people) {
  const candidates = ['Don', 'John', 'Kent'];
  return people.find(person => candidates.includes(person)) ?? '';
}
```

---

## 第八部分：对象与类级别重构

### 8.1 对象封装重构

#### 8.1.1 封装变量

**问题代码：**
```javascript
let defaultOwner = { firstName: 'Martin', lastName: 'Fowler' };

// 直接访问和修改
console.log(defaultOwner.firstName);
spaceship.owner = defaultOwner;
```

**重构后：**
```javascript
// 封装为私有变量 + Getter/Setter
let defaultOwnerData = { firstName: 'Martin', lastName: 'Fowler' };

export function getDefaultOwner() {
  return { ...defaultOwnerData }; // 返回副本，防止外部修改
}

export function setDefaultOwner(firstName, lastName) {
  defaultOwnerData = { firstName, lastName };
}

// 或使用 ES6 Proxy
const defaultOwner = new Proxy({}, {
  get(target, prop) {
    return defaultOwnerData[prop];
  },
  set(target, prop, value) {
    defaultOwnerData[prop] = value;
  }
});
```

#### 8.1.2 提取类

**问题代码：**
```javascript
class Person {
  constructor(name, officeAreaCode, officeNumber) {
    this.name = name;
    this.officeAreaCode = officeAreaCode;
    this.officeNumber = officeNumber;
  }

  getName() { return this.name; }
  getPhoneNumber() { return `(${this.officeAreaCode}) ${this.officeNumber}`; }
}
```

**重构后（提取电话号码类）：**
```javascript
class PhoneNumber {
  constructor(areaCode, number) {
    this._areaCode = areaCode;
    this._number = number;
  }

  get areaCode() { return this._areaCode; }
  get number() { return this._number; }
  toString() { return `(${this._areaCode}) ${this._number}`; }
}

class Person {
  constructor(name, phone) {
    this._name = name;
    this._phone = phone;
  }

  get name() { return this._name; }
  get phone() { return this._phone; }
  get phoneNumber() { return this._phone.toString(); }
}
```

### 8.2 函数与对象间转换

#### 8.2.1 用对象替代参数

**问题代码：**
```javascript
function setTemperature(room, desiredTemp, minTemp, maxTemp) {
  room.currentTemp = Math.max(minTemp, Math.min(maxTemp, desiredTemp));
}

setTemperature(kitchen, 22, 18, 26);
setTemperature(bedroom, 20, 18, 26);
```

**重构后：**
```javascript
function setTemperature(room, desiredTemp, limits = { min: 18, max: 26 }) {
  room.currentTemp = Math.max(limits.min, Math.min(limits.max, desiredTemp));
}

setTemperature(kitchen, 22);
setTemperature(bedroom, 20, { min: 18, max: 24 });
```

### 8.3 简化条件逻辑

#### 8.3.1 引入特例（Null Object模式）

**问题代码：**
```javascript
function getCustomer(id) {
  const customer = customers.get(id);
  if (customer === null) {
    return {
      name: '占位用户',
      isGuest: true,
      billingPlan: basicBillingPlan,
      getBillableHours: () => 0
    };
  }
  return customer;
}

// 使用时需要检查
const c = getCustomer(customerId);
if (c.isGuest) {
  // 处理访客
}
```

**重构后：**
```javascript
// 定义 NullCustomer
const NullCustomer = {
  isGuest: true,
  name: '占位用户',
  billingPlan: basicBillingPlan,
  getBillableHours: () => 0
};

function getCustomer(id) {
  return customers.get(id) ?? NullCustomer;
}

// 使用时不需要检查
const c = getCustomer(customerId);
console.log(c.getBillableHours()); // 统一调用，无须判断
```

---

## 第九部分：异步代码重构

### 9.1 回调地狱扁平化

**问题代码：**
```javascript
getData(function(a) {
  getMoreData(a, function(b) {
    getMoreData(b, function(c) {
      getMoreData(c, function(d) {
        console.log('Done:', d);
      });
    });
  });
});
```

**重构后（使用 Promise 链）：**
```javascript
getData()
  .then(a => getMoreData(a))
  .then(b => getMoreData(b))
  .then(c => getMoreData(c))
  .then(d => console.log('Done:', d))
  .catch(error => console.error(error));
```

**重构后（使用 Async/Await）：**
```javascript
async function fetchAll() {
  try {
    const a = await getData();
    const b = await getMoreData(a);
    const c = await getMoreData(b);
    const d = await getMoreData(c);
    console.log('Done:', d);
  } catch (error) {
    console.error(error);
  }
}
```

### 9.2 并行处理优化

**问题代码：**
```javascript
async function getUserInfo() {
  const user = await fetchUser();
  const posts = await fetchPosts();
  const comments = await fetchComments();
  return { user, posts, comments };
}
```

**重构后（并行请求）：**
```javascript
async function getUserInfo() {
  const [user, posts, comments] = await Promise.all([
    fetchUser(),
    fetchPosts(),
    fetchComments()
  ]);
  return { user, posts, comments };
}
```

### 9.3 Promise 错误处理统一

**问题代码：**
```javascript
fetchData()
  .then(success => processSuccess(success))
  .catch(error => {
    console.error('Error occurred');
    throw error;
  })
  .then(null, error => {
    console.error('Another error handler');
  });
```

**重构后：**
```javascript
async function handleData() {
  try {
    const data = await fetchData();
    return processSuccess(data);
  } catch (error) {
    console.error('Error occurred:', error);
    throw error;
  }
}
```

---

## 第十部分：测试与验证

### 10.1 重构验证检查清单

```
┌─────────────────────────────────────────────────────────────────┐
│                    重构后验证检查清单                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. 功能完整性                                                   │
│     □ 所有原有功能正常工作                                       │
│     □ 边界条件处理一致                                           │
│     □ 错误处理行为一致                                           │
│                                                                 │
│  2. 代码质量                                                     │
│     □ 无语法错误                                                 │
│     □ 无新增警告                                                 │
│     □ 代码风格一致                                               │
│                                                                 │
│  3. 性能                                                         │
│     □ 页面加载时间未明显增加                                     │
│     □ 无明显的性能回退                                           │
│                                                                 │
│  4. 兼容性                                                       │
│     □ 在目标浏览器中正常运行                                     │
│     □ 无新增的兼容性问题                                         │
│                                                                 │
│  5. 可访问性                                                     │
│     □ 页面仍可正常使用                                           │
│     □ 无新的可访问性问题                                         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 10.2 《重构》书中的测试理念

根据《重构》第二版的核心理念：

```
┌─────────────────────────────────────────────────────────────┐
│                      测试金字塔                               │
│                                                             │
│                        ▲                                    │
│                       ╱ ╲      E2E 测试                      │
│                      ╱   ╲     (少量)                        │
│                     ╱─────╲                                  │
│                    ╱       ╲   集成测试                       │
│                   ╱─────────╲  (适量)                        │
│                  ╱           ╲                                │
│                 ╱─────────────╲ 单元测试                     │
│                ╱               ╲(大量)                        │
│               ╱─────────────────╲                             │
│                                                             │
│  原则：                                                      │
│  • 单元测试：覆盖核心业务逻辑                                 │
│  • 集成测试：验证组件间协作                                   │
│  • E2E测试：覆盖关键用户路径                                  │
└─────────────────────────────────────────────────────────────┘
```

### 10.3 自验证代码

即使没有正式的测试框架，也可以让代码自我验证：

```javascript
// 重构前
function calculatePrice(quantity, itemPrice) {
  return quantity * itemPrice * 0.9;
}

// 重构后 - 包含自验证
function calculatePrice(quantity, itemPrice) {
  const price = quantity * itemPrice * 0.9;

  // 自验证断言
  if (price < 0) {
    throw new Error('价格不能为负数');
  }
  if (quantity < 0) {
    throw new Error('数量不能为负数');
  }

  return price;
}
```

---

## 附录：重构速查卡

### 常用重构手法速查

| 原代码 | 重构手法 | 目标代码 |
|--------|----------|----------|
| 重复代码 | 提取函数/类 | 单一职责 |
| 长函数 | 拆分函数 | 短小精悍 |
| 过长参数列表 | 引入参数对象 | 简洁调用 |
| 全局变量 | 封装变量 | 控制访问 |
| switch语句 | 用多态替代 | 扩展性 |
| 深层嵌套 | 卫语句 | 可读性 |
| 魔法字符串/数字 | 提取常量 | 可维护性 |
| 链式调用 | 隐藏委托 | 封装性 |
| 冗余注释 | 提取函数 | 自解释代码 |

---

## 总结

```
┌─────────────────────────────────────────────────────────────────┐
│                     重构核心要点                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  心态：                                                         │
│  • 小步前进，每次只做一个改动                                   │
│  • 编译/运行验证每次改动                                         │
│  • 保持代码行为不变                                             │
│                                                                 │
│  方法：                                                         │
│  • 先识别坏味道                                                 │
│  • 选择合适的重构手法                                           │
│  • 逐步实施                                                     │
│                                                                 │
│  目标：                                                         │
│  • 代码更清晰                                                   │
│  • 结构更合理                                                   │
│  • 易于维护和扩展                                               │
│                                                                 │
│  记住《重构》作者 Martin Fowler 的话：                          │
│  "任何一个傻瓜都能写出计算机可以理解的代码，                       │
│   只有写出人类能够理解的代码，才是真正的程序员。"                  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---