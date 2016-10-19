title: python全角半角转换
date: 2015-09-21 14:34:54
tags: python
---
python全角半角转换
===========
```python
class full2half:

    # 构造函数
    def __ini__(self):
        pass

    # 将半角字符转换成全角字符
    def charB2Q(self, uchar):

        if uchar == '':
            return ''

        inside_code = ord(uchar)

        if not inside_code in range(32, 127):
            return uchar

        if inside_code == 32:
            inside_code = 12288
        else:
            inside_code += 65248

        if inside_code in range(32, 127):
            return uchar

        return unichr(inside_code).encode('utf-8', 'ignore')

    # 将全角字符转换成半角字符
    def charQ2B(self, uchar):

        if uchar == '':
            return ''

        inside_code = ord(uchar)

        if inside_code in range(32, 127):
            return uchar

        if inside_code == 12288:
            inside_code = 32
        else:
            inside_code -= 65248

        if not inside_code in range(32, 127):
            return uchar

        return unichr(inside_code).encode('utf-8', 'ignore')

    # 将字符串中的半角转换成全角
    def StingB2Q(self, ustring):

        try:
            ustring = ustring.decode('utf-8', 'ignore')
        except:
            pass

        result = ''
        for uchar in ustring:

            try:
                uchar = uchar.encode('utf-8', 'ignore')
            except:
                pass

            try:
                inside_code = ord(uchar)
            except:
                inside_code = ''

            if inside_code:
                if inside_code in range(32, 127):
                    result += self.charB2Q(uchar)
                else:
                    result += uchar
            else:
                result += uchar

        return result

    # 将字符串中的全角转换成半角
    def StingQ2B(self, ustring):

        try:
            ustring = ustring.decode('utf-8', 'ignore')
        except:
            pass

        result = ''
        for uchar in ustring:

            try:
                inside_code = ord(uchar)
            except:
                inside_code = ''

            if inside_code:
                if inside_code in range(32, 127):
                    result += uchar
                else:
                    result += self.charQ2B(uchar)
            else:
                result += uchar

        return result.encode('utf-8', 'ignore')

    # 析构函数
    def __del__(self):
        pass
```
