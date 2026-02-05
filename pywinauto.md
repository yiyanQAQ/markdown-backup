## 一、pywinauto 是什么

**pywinauto** 是一个用于 **Windows GUI 自动化** 的 Python 库，适合自动化：

- 传统 Win32 程序（如：记事本、老式工具）
- 基于 **UI Automation (UIA)** 的程序（Win10/Win11、WPF、UWP）
- 部分 Electron / Qt 程序（取决于控件暴露情况）

常用于：

- 自动化测试
- RPA
- 批量操作工具软件
- 无接口程序的“补救方案”

---

## 二、安装

```bash
pip install pywinauto
```

推荐同时安装：

```bash
pip install pywin32 comtypes
```

---

## 三、核心概念（非常重要）

### 1️⃣ backend（后端）

pywinauto 有两个主要 backend：

| backend | 适用场景                      |
| ------- | ----------------------------- |
| `win32` | 老程序、经典 Win32            |
| `uia`   | Win10/Win11、现代软件（推荐） |

**原则：**

> **优先用 `uia`，不行再试 `win32`**

---

### 2️⃣ Application（应用对象）

所有操作从 `Application` 开始

```python
from pywinauto import Application
```

---

## 四、启动 / 连接程序

### 1️⃣ 启动程序

```python
app = Application(backend="uia").start(r"C:\Windows\System32\notepad.exe")
```

---

### 2️⃣ 连接已打开的程序（常用）

#### 按进程名

```python
app = Application(backend="uia").connect(path="notepad.exe")
```

#### 按 PID

```python
app = Application(backend="uia").connect(process=1234)
```

#### 按窗口标题（模糊）

```python
app = Application(backend="uia").connect(title_re=".*记事本.*")
```

---

## 五、窗口对象（Window）

### 1️⃣ 获取主窗口

```python
dlg = app.window(title_re=".*记事本.*")
```

或：

```python
dlg = app.top_window()
```

---

### 2️⃣ 等待窗口出现（非常重要）

```python
dlg.wait("visible", timeout=10)
```

常见等待条件：

- `visible`
- `exists`
- `enabled`
- `ready`

---

## 六、控件定位（核心中的核心）

### 1️⃣ 查看控件结构（必会）

```python
dlg.print_control_identifiers()
```

> **新手 80% 的问题靠这个解决**

---

### 2️⃣ 常用定位方式

#### 按 title / name

```python
dlg.child_window(title="确定", control_type="Button")
```

#### 按 automation_id（最稳定）

```python
dlg.child_window(auto_id="1", control_type="Edit")
```

#### 简写（推荐）

```python
dlg["确定"]
dlg.Edit
dlg.Button
```

---

### 3️⃣ 常见 control_type

| 类型                | 说明   |
| ------------------- | ------ |
| `Window`            | 窗口   |
| `Button`            | 按钮   |
| `Edit`              | 输入框 |
| `Text`              | 文本   |
| `ComboBox`          | 下拉框 |
| `List` / `ListItem` | 列表   |
| `CheckBox`          | 复选框 |
| `RadioButton`       | 单选框 |
| `Pane`              | 容器   |

---

## 七、常见操作

### 1️⃣ 点击

```python
dlg["确定"].click()
dlg["确定"].click_input()   # 更接近真实点击（推荐）
```

---

### 2️⃣ 输入文本

```python
dlg.Edit.set_text("hello world")
```

或模拟键盘：

```python
dlg.Edit.type_keys("hello{ENTER}", with_spaces=True)
```

---

### 3️⃣ 读取文本

```python
dlg.Text.window_text()
```

---

### 4️⃣ 下拉框选择

```python
dlg.ComboBox.select("选项1")
```

---

### 5️⃣ 勾选 / 取消勾选

```python
dlg.CheckBox.check()
dlg.CheckBox.uncheck()
```

---

## 八、键盘 & 鼠标（全局）

```python
from pywinauto.keyboard import send_keys
from pywinauto.mouse import click
```

### 键盘

```python
send_keys("^a{DEL}")   # Ctrl+A + Delete
```

### 鼠标

```python
click(button='left', coords=(100, 200))
```

---

## 九、等待 & 稳定性（RPA 必看）

### 1️⃣ 等控件就绪

```python
btn.wait("enabled", timeout=10)
```

### 2️⃣ 判断是否存在

```python
if dlg.child_window(title="确定").exists():
    print("存在")
```

---

## 十、异常处理（非常实用）

```python
from pywinauto.findwindows import ElementNotFoundError

try:
    dlg["确定"].click()
except ElementNotFoundError:
    print("按钮不存在")
```

---

## 十一、与 RPA / Selenium / 影刀的关系

| 场景             | 推荐                  |
| ---------------- | --------------------- |
| 浏览器网页       | Selenium / Playwright |
| Windows 桌面程序 | pywinauto             |
| 复杂业务流程     | 影刀                  |
| pywinauto 搞不定 | pyautogui（兜底）     |

你现在做 **影刀 RPA + Python 拓展**，pywinauto 非常适合用来：

- 操作影刀无法识别的桌面程序
- 写成 Python 子流程给影刀调用

---

## 十二、常见坑（经验总结）

1. **权限问题**
   - Python 必须和目标程序 **同权限**
   - 程序是管理员启动 → Python 也要管理员

2. **Electron 程序**
   - UIA 不一定暴露完整控件
   - 有时只能退回 `pyautogui`

3. **窗口最小化**
   - 最小化状态无法操作

   ```python
   dlg.restore()
   ```

---

## 十三、一个完整示例

```python
from pywinauto import Application

app = Application(backend="uia").start("notepad.exe")
dlg = app.window(title_re=".*记事本.*")
dlg.wait("ready", timeout=10)

dlg.Edit.set_text("pywinauto 测试")
dlg.menu_select("文件->保存")
```

---
