

# Pytest 的使用

## 一、运行测试用例的多种方式

### 1. 执行单个模块中的全部用例

   ```shell
   pytest test_mod.py
   ```

### 2. 执行指定路径下的全部用例

   ```shell
   pytest testing/
   ```

### 3. 执行字符串表达式中的用例

    ```shell
    pytest -k "MyClass and not method"
    ```
    
    - 此例将选择 TestMyClass.test_something，排除了TestMyClass.test_method_simple
    - 表达式将匹配文件名、类名、方法名

### 4. 运行某个模块、某个类、某个方法

   ```shell
   #运行指定模块中的某个用例，如运行 test_mod.py 模块中的 test_func 测试函数
   pytest test_mod.py::test_func
   #运行某个类下的某个用例，如运行 TestClass 类下的 test_method 测试方法
   pytest test_mod.py::TestClass::test_method
   ```

### 5. 执行某个包中的用例

   ```shell
   pytest --pyargs pkg.testing
   ```

   - 执行 pkg.testing包目录下的所有用例,使用其文件系统位置来查找和执行用例。

### 6. 通过pytest.mark对test方法分类执行

    ```shell
    @pytest.mark.website
    def test_1(setup_function):
        print('Test_1 called.')
        
    #运行使用pytest.mark.website 装饰的用例 
    pytest -m website 
    ```
### 7. marker 的常用用法

   ```python
   # 跳过测试
   @pytest.mark.skip(reason=None)
   
   # 满足某个条件时跳过该测试
   @pytest.mark.skipif(condition)
   
   # 预期该测试是失败的
   @pytest.mark.xfail(condition, reason=None, run=True, raises=None, strict=False)
   
   # 参数化测试函数。给测试用例添加参数，供运行时填充到测试中
   # 如果 parametrize 的参数名称与 fixture 名冲突，则会覆盖掉 fixture
   @pytest.mark.parametrize(argnames, argvalues)
   
   # 对给定测试执行给定的 fixtures
   # 这种用法与直接用 fixture 效果相同
   # 只不过不需要把 fixture 名称作为参数放在方法声明当中
   @pytest.mark.usefixtures(fixturename1, fixturename2, ...)
   
   # 让测试尽早地被执行
   @pytest.mark.tryfirst
   
   # 让测试尽量晚执行
   @pytest.mark.trylast
   ```

   例如一个使用参数化测试的例子：

   ```python
   @pytest.mark.parametrize(("n", "expected"), [
       (1, 2),
       (2, 3),
   ])
   def test_increment(n, expected):
        assert n + 1 == expected
   ```

   除了内建的 markers 外，pytest 还支持没有实现定义的 markers，如：

   ```python
   @pytest.mark.old_test
   def test_one():
       assert False
   
   @pytest.mark.new_test
   def test_two():
       assert False
   
   @pytest.mark.windows_only
   def test_three():
       assert False
   ```

   通过使用 `-m` 参数可以让 pytest 选择性的执行部分测试：

   ```shell
   $ pytest test.py -m 'not windows_only'
   ...
   collected 3 items / 1 deselected                   
   test_marker.py::test_one FAILED
   ```

   更详细的关于 marker 的说明可以参考官方文档：

   - [https://docs.pytest.org/en/latest/mark.html](https://docs.pytest.org/en/latest/mark.html)
   - [https://docs.pytest.org/en/latest/example/markers.html](https://docs.pytest.org/en/latest/example/markers.html)
	



## 二、Fixtures 的使用

### 1. 作为参数

`fixture` 可以作为其他测试函数的参数被使用，前提是其必须返回一个值：

```python
@pytest.fixture()
def hello():
    return "hello"

def test_string(hello):
    assert hello == "hello", "fixture should return hello"
```

一个更加实用的例子：

```python
@pytest.fixture
def smtp():
    import smtplib
    return smtplib.SMTP("smtp.gmail.com")

def test_ehlo(smtp):
    response, msg = smtp.ehlo()
    assert response == 250
    assert 0 # for demo purposes
```

## 三、Console参数介绍

- -v 用于显示每个测试函数的执行结果
- -q 只显示整体测试结果
- -s 用于显示测试函数中print()函数输出
- -x, --exitfirst, exit instantly on first error or failed test
- -h 帮助

