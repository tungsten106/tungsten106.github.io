
# pandas 缺失值处理

- 向上/向下填充
```python
# 向下填充（用na的上一个值来填充na位置）
df["colA"].fillna(method="ffill", inplace=True)
# 向上填充（用na的下一个值来填充na位置）
df["colA"].fillna(method="bfill", inplace=True) 
```
- 用平均值来填充: 
```python
df["colA"].fillna(df["colA"].mean(), inplace = True)
```
- 
