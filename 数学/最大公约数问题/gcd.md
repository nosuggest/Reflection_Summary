# 辗转相除法
```python
def solve(a,b):
    return a if b==0 else solve(b,a%b)
```

# 其他方法
- 穷举法
- 辗转相减法