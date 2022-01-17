# Django-Study

## 랜덤 발표 순서 뽑기
```python
import random

lists = [0,1,2]
names = ["손명현", "이하연", "임지현"]
idx = random.sample(lists, 2)

for i,val in enumerate(idx):
  print(f"{i+1}번 {names[val]}")
```
