# string.templatelib （模板字符串支持）

此模块支持 [PEP 750](https://peps.python.org/pep-0750/) 中定义的模板字符串（t-字符串）。模板字符串使用 `t` 前缀创建，并在组合之前提供对文字字符串部分和插补文字的访问。

**可用性**：模板字符串要求在编译时启用 `MICROPY_PY_TSTRINGS`。默认情况下，它们在完整功能级别启用，其中包括 alif、mimxrt 和 samd（仅限SAMD51）、unix 变体和 webassembly pyscript 变体。

## 类

- class string.templatelib.`Template`(*args)  
  表示模板字符串。模板对象通常通过 t-string 语法（t"..."）创建，但也可以直接使用构造函数构造。
  
  - `strings`  
    在插补文字之间出现的字符串字面值的元组。
  
  - `interpolations`  
    表示插值表达式的插值对象元组。
  
  - `values`  
    包含模板中每个插值属性的只读属性元组

  - `__iter__`()  
    迭代模板内容，按照出现的顺序生成字符串部分和插值对象。空字符串被省略。
  
  - `__add__`(other)  
    连接两个模板。返回一个新的 `Template`，将两个模板中的字符串和插值组合在一起。如果 `other` 不是 `Template`，将引发 `TypeError`。
  
    禁止使用str进行模板连接，以避免字符串应被视为文字还是插补文字的歧义：
    ```python
    t1 = t"Hello "
    t2 = t"World"
    result = t1 + t2  # 有效

    # TypeError: 无法将str连接到Template
    result = t1 + "World"
    ```

- class string.templatelib.`Interpolation`(value, expression='', conversion=None, format_spec='')  
  模板字符串中的插补文字表达式。所有参数都可以作为关键字参数传递。

  - `value`  
  插补文字表达式的计算值。
  
  - `expression`  
  表达式在模板字符串中的字符串表示形式。
  
  - `conversion`  
  转换说明符（'s'或'r'）（如果存在），否则为 `None`。请注意，MicroPython 不支持'a'转换。
  
  - `format_spec`  
  格式规范字符串（如果存在），否则为空字符串。


## 模板字符串语法

模板字符串使用与f字符串相同的语法，但前缀为 `t`：

```python
name = "World"
template = t"Hello {name}!"

# 访问模板组件
print(template.strings)   # ('Hello ', '!')
print(template.values)    # ('World',)
print(template.interpolations[0].expression) # 'name'
```

## 转换说明符

模板字符串将转换说明符存储为元数据。与f-string不同，转换不会自动应用：

```python
value = "test"
t = t"{value!r}"
# t.interpolations[0].value == "test" (not repr(value))
# t.interpolations[0].conversion == "r"
```

处理代码必须在需要时显式应用转换。

### 格式规范

格式规范作为元数据存储在插补文字对象中。与f字符串不同，格式不会自动应用：

```python
pi = 3.14159
t = t"{pi:.2f}"
# t.interpolations[0].value == 3.14159 (not formatted)
# t.interpolations[0].format_spec == ".2f"
```

根据PEP 750，处理代码不需要使用格式规范，但如果存在，则应遵守这些规范，并在可能的情况下匹配f-string行为。

### 调试格式

支持调试格式 `{expr=}`：

```python
x = 42
t = t"{x=}"
# t.strings == ("x=", "")
# t.interpolations[0].expression == "x"
# t.interpolations[0].conversion == "r"
```

**重要**

根据PEP 750，与f字符串不同，模板字符串不会自动应用转换或格式规范。这是为了允许处理代码控制如何处理这些。处理代码必须显式处理这些属性。

MicroPython不提供 `format()` 内置函数。请改用字符串格式化方法，如 `str.format()`。


## 示例用法

不支持格式的基本处理：

```python
def simple_process(template):
    """Simple template processing"""
    parts = []
    for item in template:
        if isinstance(item, str):
            parts.append(item)
        else:
            parts.append(str(item.value))
    return "".join(parts)
```

支持格式的处理模板：

```python
from string.templatelib import Template, Interpolation

def convert(value, conversion):
    """Apply conversion specifier to value"""
    if conversion == "r":
        return repr(value)
    elif conversion == "s":
        return str(value)
    return value

def process_template(template):
    """Process template with conversion and format support"""
    result = []
    for part in template:
        if isinstance(part, str):
            result.append(part)
        else: # Interpolation
            value = convert(part.value, part.conversion)
        if part.format_spec:
            # Apply format specification using str.format
            value = ("{:" + part.format_spec + "}").format(value)
        else:
            value = str(value)
        result.append(value)
    return "".join(result)

pi = 3.14159
name = "Alice"
t = t"{name!r}: {pi:.2f}"
print(process_template(t))
# Output: "'Alice': 3.14"

# Other format specifications work too
value = 42
print(process_template(t"{value:>10}")) # "        42"
print(process_template(t"{value:04d}")) # "0042"
```

HTML转义示例：

```python
def html_escape(value):
    """Escape HTML special characters"""
    if not isinstance(value, str):
        value = str(value)
    return value.replace("&", "&amp;").replace("<", "&lt;").replace(">", "&gt;")

def safe_html(template):
    """Convert template to HTML-safe string"""
    result = []
    for part in template:
        if isinstance(part, str):
            result.append(part)
        else:
            result.append(html_escape(part.value))
    return "".join(result)

user_input = "<script>alert('xss')</script>"
t = t"User said: {user_input}"
print(safe_html(t))
# Output: "User said: &lt;script&gt;alert('xss')&lt;/script&gt;"
```

## 参见

- [PEP 750](https://peps.python.org/pep-0750/) - 模板字符串规范
- [Format String Syntax](https://docs.python.org/3.5/library/string.html#formatstrings) - 格式化字符串语法
- [Formatted string literals](https://docs.python.org/3/reference/lexical_analysis.html#f-strings) - Python中的f字符串
