# Python外部参数传入

## 1. argparse

```python
import argparse

def test():
    parser = argparse.ArgumentParser(description="test")
    
    parser.add_argument('-n','--name',default="alex")   
    parser.add_argument('-y', '--year',default="18")
    parser.add_argument('-c','--city',default='beijing')  
    

    args = parser.parse_args()
    

    name = args.name
    year = args.year
    city = args.city
    

    print(f'my name is {name}, {year} years old. my city is {city}')
    
if __name__ == '__main__':
    test()
```

执行命令时:

```bash
python "C:\Users\Lian\Desktop\test.py" -y 23 -c shanghai -n tony 
```

## 2. fire

```python
import fire

def hello(name):
  return 'Hello {name}!'.format(name=name)

if __name__ == '__main__':
  fire.Fire(hello)
```

调用

```bash
python "C:\Users\Lian\Desktop\test.py" test
```

## 3. sys

在右键菜单中调用python脚本.

```python
import sys

script,first,second,third = sys.argv

print ("The script is called:{%s}"% script)
print ("The first variable is:{%s}"% first)
print ("The second variable is:{%s}"% second)
print ("The third variable is:{%s}"% third)
```

右键菜单的生效位置和注册表对应的路径.

![未命名图片.png](https://img1.imgtp.com/2023/01/02/U4XDv9vO.png)

注册表

![reg](https://img1.imgtp.com/2023/01/02/r79nymIu.png)

注意参数的设置

![未命名图片.png](https://img1.imgtp.com/2023/01/03/d238ULQj.png)

![未命名图片_a.png](https://img1.imgtp.com/2023/01/02/THFoQ4ow.png)
