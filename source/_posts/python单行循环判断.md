title: python单行循环判断
date: 2015-09-24 10:21:04
tags:
- python
- spark
---
```python
quickbrownfox = 'A quick brown fox jumps over the lazy dog.'
split_regex = r'\W+'

def simpleTokenize(string):
    """ A simple implementation of input string tokenization
    Args:
        string (str): input string
    Returns:
        list: a list of tokens
    """
    return [token for token in re.split(split_regex,string.lower()) if token != ""]

print simpleTokenize(quickbrownfox) # Should give ['a', 'quick', 'brown', ... ]
```
