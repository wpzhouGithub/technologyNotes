# python+selenium+Firefox爬虫开发

[TOC]

系统环境： Ubuntu 18

## Selenium

Selenium是一个web自动化测试框架；它通过driver与浏览器交互发命令，控制浏览器上的交互；

Selenium  有两张运行方式：

1. 有界面的，即浏览器直接在桌面ui打开
2. Headless ，无头模式，不打开浏览器，在后台运行；

- 无头模式可以直接在无显示器的Ubuntu系统显示；

```python
from selenium import webdriver

options = webdriver.FirefoxOptions()
options.add_argument('--headless')
driver = webdriver.Firefox(options=options)

driver = webdriver.Firefox()
driver.get('https://www.baidu.com')
print(driver.title)
driver.quit()
```

- 想要在无显示器的系统显示非Headless 模式，就需要虚拟环境；xvfb 可以实现此功能，pyvirtualdisplay是基于xvfb 的python包，它会自动调起xvfb ；

```python
from pyvirtualdisplay import Display
from selenium import webdriver

display = Display(visible=0)
display.start()

driver = webdriver.Firefox()
driver.get('https://www.baidu.com')
print(driver.title)
driver.quit()
display.stop()
```

## 由chrome看driver的原理

首先我们来写一个示例代码：

```python
from selenium import webdriver

driver = webdriver.Chrome()
driver.get("https://www.baidu.com/")
```

执行这段代码，会看到系统启动了chrome浏览器，并跳转到了百度首页。

看driver = webdriver.Chrome()这一句，以下是这段代码的流程图解。

![](I:\笔记\pictures\chrome_driver.png)

可以看到在这个流程中，chromedriver起到的是桥梁的作用，他接受客户端的请求，然后转化为浏览器的标准指令操作浏览器

## 环境安装

### 1. python环境安装

```shell
#conda安装
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
bash Miniconda3-latest-Linux-x86_64.sh

#安装python爬虫虚拟环境
conda env create -n crawler_selenium  python==3.6
conda activate crawler_selenium
pip install selenium
pip install pyvirtualdisplay
```

### 2. 浏览器环境安装

```shell
#Firefox浏览器安装
apt-get install firefox

#下载对应的driver，不同的浏览器driver不一样； 要注意driver和浏览器的兼容，版本不兼容容易报错
wget https://github.com/mozilla/geckodriver/releases/download/v0.26.0/geckodriver-v0.26.0-linux64.tar.gz
#解压到PATH目录，这里是放到了python虚拟环境的bin目录下了


```

## 如何调测爬虫

