---
layout: post
title: python click 让命令行传参666
date: 2017-07-14 07:30:00 +0800
categories: python
tag: python
---
好久没写，开个小头 :smile:
# Python click第三方模块

## 官方的小实例
```python
import click

@click.command()
@click.option("--count",default=1,help="Number of greetings.")
@click.option("--name",prompt="Your name:",help="Man you are greeting.")
def hello(count,name):
    '''Simple test on click'''
    for _ in range(count):
        click.echo("Hello {0}".format(name))
if __name__ == "__main__":
    hello()
```
看下结果如何：
```bash
gwoose@localhost ~]$python click_test.py  --name Jimy
Hello Jimy

gwoose@localhost ~]$python click_test.py  --count 3 --name Jimy
Hello Jimy
Hello Jimy
Hello Jimy

gwoose@localhost ~]$python click_test.py  --help
Usage: click_test.py [OPTIONS]

  Simple test on click

Options:
  --count INTEGER  Number of greetings.
  --name TEXT      Man you are greeting.
  --help           Show this message and exit.

```
以下是关键点：
* @click.command() 使函数 hello 成为命令行接口；
* @click.option 的第一个参数指定了命令行选项的名称，可以看到，count 的默认值是 1；
* 使用 click.echo 进行输出是为了获得更好的兼容性，因为 print 在 Python2 和 Python3 的用法有些差别
* 比较关注的是：参数解析的位置，在main函数中任意位置都是可以解析命令行参数的。
## click.option
option 最基本的用法就是通过指定命令行选项的名称，从命令行读取参数值，再将其传递给函数。在上面的例子，我们看到，除了设置命令行选项的名称，我们还会指定默认值，help 说明等，option 常用的设置参数如下：
* help: 参数说明
* default: 设置命令行参数的默认值
* type: 参数类型，可以是 string, int, float 等
* prompt: 当在命令行中没有输入相应的参数时，会根据 prompt 提示用户输入
* isflag: isflag=True，此时，该选项只是标志，不加为False，加上为True。

### 具体相关实例
1. type参数
type参数可以传入：int/string/float类型较为常见
#### 可选值
```bash
import click

@click.command()
@click.option("--count",default="1",type=click.Choice(['0','1','2','3']),help="Number of greetings.")
@click.option("--name",prompt="Your name:",help="Man you are greeting.")
def hello(count,name):
    '''Simple test on click'''
    count = int(count)
    if count < 4 and count >0:
        for _ in range(count):
            click.echo("Hello {0}".format(name))
    elif count == 0 :
        click.echo("NO greeting")
if __name__ == "__main__":
    hello()

```
运行结果：
```
gwoose@localhost ~]$python click_test.py  --count 1
Your name:: jim
Hello jim

gwoose@localhost ~]$python click_test.py  --count 4
Usage: click_test.py [OPTIONS]

Error: Invalid value for "--count": invalid choice: 4. (choose from 0, 1, 2, 3)

gwoose@localhost ~]$python click_test.py  --help
Usage: click_test.py [OPTIONS]

  Simple test on click

Options:
  --count [0|1|2|3]  Number of greetings.
  --name TEXT        Man you are greeting.
  --help             Show this message and exit.

```
 * 如官方所说：The choice type allows a value to be checked against a fixed set of supported values.  All of these values have to be strings.
 #### 多参数nargs 
 ```python
import click

@click.command()
@click.option("--centre",nargs=2 ,help="centre of the circle.")
@click.option("--radius",help="radius of the circle")
def cover(centre,radius):
    '''Get cover of circle'''
    click.echo('center: %s, radius: %s' % (centre, radius))
if __name__ == "__main__":
    cover()

 ```
 测试：
 ```bash
 gwoose@localhost ~]$python click_test_2.py  --help
Usage: click_test_2.py [OPTIONS]

  Get cover of circle

Options:
  --centre TEXT...  centre of the circle.
  --radius TEXT     radius of the circle
  --help            Show this message and exit.

gwoose@localhost ~]$python click_test_2.py  --centre 3.4 5
center: (u'3.4', u'5'), radius: None

gwoose@localhost ~]$python click_test_2.py  --centre 3 4 5
Usage: click_test_2.py [OPTIONS]

Error: Got unexpected extra argument (5)

gwoose@localhost ~]$python click_test_2.py  --centre 3 4 --radius 5
center: (u'3', u'4'), radius: 5

 ```

 #### 密码不显示
 ```python
 import click

@click.command()
@click.option("--name" ,prompt="user",help="The user name")
@click.option("--passwd",prompt="New pass" ,help="new password",hide_input= True,confirmation_prompt=True)
def chgpasswd(name,passwd):
    '''Change user's password.'''
    click.echo("'{0}'s password is changed ".format(name))
if __name__ == "__main__":
    chgpasswd()

 ```
 测试结果：
 ```bash
 gwoose@localhost ~]$python click_test_3.py  --help
Usage: click_test_3.py [OPTIONS]

  Change user's password.

Options:
  --name TEXT    The user name
  --passwd TEXT  new password
  --help         Show this message and exit.

gwoose@localhost ~]$python click_test_3.py
user:: Jimy
New pass::
Repeat for confirmation:
'Jimy's password is changed

gwoose@localhost ~]$python click_test_3.py
user: Jimy
New pass:
Repeat for confirmation:
Error: the two entered values do not match
New pass:
Repeat for confirmation:
'Jimy's password is changed

 ``` 
注意：
    * 两次输入密码不一致可以自动处理，666
    * 这里的prompt自动有格式，不用画蛇添足
#### 截断程序运行
```python

import click

def print_version(ctx, param, value):
    if not value or ctx.resilient_parsing:
        return
    click.echo('Version 1.0')
    ctx.exit()

@click.command()
@click.option('--version', is_flag=True, callback=print_version,expose_value=False, is_eager=True,help="show software version and exit.")
@click.option('--name', default='Ethan', help='name')
def hello(name):
    click.echo('Hello %s!' % name)

if __name__ == '__main__':
    hello()
```
执行结果：
```bash
gwoose@localhost ~]$python click_test_4.py   --help
Usage: click_test_4.py [OPTIONS]

Options:
  --version    show software version and exit.
  --name TEXT  name
  --help       Show this message and exit.

gwoose@localhost ~]$python click_test_4.py   --version --name Jimy
Version 1.0

gwoose@localhost ~]$python click_test_4.py   --version --name jim
Version 1.0

gwoose@localhost ~]$python click_test_4.py
Hello Ethan!
```
说明：
* is_eager=True 表明该命令行选项优先级高于其他选项；
* expose_value=False 表示如果没有输入该命令行选项，会执行既定的命令行流程；
* callback 指定了输入该命令行选项时，要跳转执行的函数；
## click.argument
我们除了使用 @click.option 来添加可选参数，还会经常使用 @click.argument 来添加固定参数。它的使用和 option 类似，但支持的功能比 option 少。
入门示例
```python
import click

@click.command()
@click.argument('coordinates')
def show(coordinates):
    click.echo('coordinates: %s' % coordinates)

if __name__ == '__main__':
    show()
```
执行结果：
```bash
gwoose@localhost ~]$python click_test_5.py   --help
Traceback (most recent call last):
  File "click_test_5.py", line 4, in <module>
    @click.argument('coordinates',help="COORDINATES")
  File "C:\Python27\lib\site-packages\click\decorators.py", line 151, in decorator
    _param_memo(f, ArgumentClass(param_decls, **attrs))
  File "C:\Python27\lib\site-packages\click\core.py", line 1699, in __init__
    Parameter.__init__(self, param_decls, required=required, **attrs)
TypeError: __init__() got an unexpected keyword argument 'help'

gwoose@localhost ~]$python click_test_5.py   --help
Usage: click_test_5.py [OPTIONS] COORDINATES

Options:
  --help  Show this message and exit.

gwoose@localhost ~]$python click_test_5.py   234
coordinates: 234

```
### 传递多个参数
```python
import click

@click.command()
@click.argument('x')
@click.argument('y')
@click.argument('z')
def show(x, y, z):
    click.echo('x: %s, y: %s, z:%s' % (x, y, z))

if __name__ == '__main__':
    show()
```
执行结果
```bash
gwoose@localhost ~]$python click_test_6.py   --help
Usage: click_test_6.py [OPTIONS] X Y Z

Options:
  --help  Show this message and exit.

gwoose@localhost ~]$python click_test_6.py   2 3 4
x: 2, y: 3, z:4
```
### 不定参数
```python
import click

@click.command()
@click.argument('src', nargs=-1)
@click.argument('dst', nargs=1)
def move(src, dst):
    click.echo('move %s to %s' % (src, dst))

if __name__ == '__main__':
    move()
```
执行结果：
```bash
gwoose@localhost ~]$python ./click_test_7.py /a /b/v /d /dest
move (u'/a', u'/b/v', u'/d') to /dest

gwoose@localhost ~]$python ./click_test_7.py --help
Usage: click_test_7.py [OPTIONS] [SRC]... DST

Options:
  --help  Show this message and exit.

```
### 彩色输出
在前面的例子中，我们使用 click.echo 进行输出，如果配合 colorama 这个模块，我们可以使用 click.secho 进行彩色输出，在使用之前，使用 pip 安装 colorama。
```python
import click

@click.command()
@click.option('--name', help='The person to greet.')
def hello(name):
    click.secho('Hello %s!' % name, fg='red', underline=True)
    click.secho('Hello %s!' % name, fg='yellow', bg='black')

if __name__ == '__main__':
    hello()
```
执行结果：
```bash
gwoose@localhost ~]$python ./click_test_color.py --name jimy
Hello jimy!
Hello jimy!
```