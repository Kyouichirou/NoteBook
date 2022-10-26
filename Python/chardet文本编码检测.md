# chardet文本编码检测

The easiest way to use the Universal Encoding Detector library is with the `detect` function.

## Example: Using the `detect` function

基本使用

The `detect` function takes one argument, a non-Unicode string. It returns a dictionary containing the auto-detected character encoding and a confidence level from `0` to `1`.

```python
>>> import urllib.request
>>> rawdata = urllib.request.urlopen('http://yahoo.co.jp/').read()
>>> import chardet
>>> chardet.detect(rawdata)
{'encoding': 'EUC-JP', 'confidence': 0.99}
```

## Advanced usage

进阶

If you’re dealing with a large amount of text, you can call the Universal Encoding Detector library incrementally, and it will stop as soon as it is confident enough to report its results.

Create a `UniversalDetector` object, then call its `feed` method repeatedly with each block of text. If the detector reaches a minimum threshold of confidence, it will set `detector.done` to `True`.

分块读取, 通过预设的阈值, 当数据的可信度达到阈值就退出检测.

Once you’ve exhausted the source text, call `detector.close()`, which will do some final calculations in case the detector didn’t hit its minimum confidence threshold earlier. Then `detector.result` will be a dictionary containing the auto-detected character encoding and confidence level (the same as the `chardet.detect` function [returns](https://chardet.readthedocs.io/en/latest/usage.html#example-using-the-detect-function)).

## Example: Detecting encoding incrementally

对大文件的检测, 当文件过大, 显然不能直接检查全部的内容, 分块读取判断

```python
import urllib.request
from chardet.universaldetector import UniversalDetector

usock = urllib.request.urlopen('http://yahoo.co.jp/')
detector = UniversalDetector()
for line in usock.readlines():
    detector.feed(line)
    if detector.done: break
detector.close()
usock.close()
print(detector.result)
{'encoding': 'EUC-JP', 'confidence': 0.99}
```

If you want to detect the encoding of multiple texts (such as separate files), you can re-use a single `UniversalDetector` object. Just call `detector.reset()` at the start of each file, call `detector.feed` as many times as you like, and then call `detector.close()` and check the `detector.result` dictionary for the file’s results.

## Example: Detecting encodings of multiple files

检查多个文件

```python
import glob
from chardet.universaldetector import UniversalDetector

detector = UniversalDetector()
for filename in glob.glob('*.xml'):
    print(filename.ljust(60), end='')
    detector.reset()
    for line in open(filename, 'rb'):
        detector.feed(line)
        if detector.done: break
    detector.close()
    print(detector.result)
```

## 对pandas的read_csv封装

```python
__all__ = ['read_csv']

import chardet
import pandas as pd
from .log_module import Logs

_logger = Logs()


# @description: 读取csv表格, 部分文件的编码非标准, 需要侦测其中的编码格式

def _detect_code(filepath: str):
    with open(filepath, 'rb+') as ff:
        content = ff.read()
        # 不能只读取部分, 判断不准确, 需要完整的读取, 才能准确判断出文本的编码格式
        code = chardet.detect(content)['encoding'].lower()
        if code.startswith('gb'):
            # 183030涵盖, gbk, gb2312
            _logger.debug(f'{filepath}, 侦测到文件的编码为{code}, 自动变更为gb18030')
            code = 'gb18030'
        return code


@_logger.decorator('处理csv文件错误')
def read_csv(configs: dict, mode=False):
    try:
        df = pd.read_csv(**configs)
        return df
    except UnicodeError:
        if not mode:
            pre = configs.get('encoding') or 'utf-8'
            code = _detect_code(configs['filepath_or_buffer'])
            if code and pre != code:
                _logger.debug(f'文件编码异常: {pre}, 自动调整编码格式为: {code}')
                configs['encoding'] = code
                return read_csv(configs, mode=True)
        else:
            _logger.warning(f'无法正确打开文件: {configs["filepath_or_buffer"]}')
```

