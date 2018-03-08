```flow
st=>start: start:>http://www.baidu.com
op1=>operation: 操作1
cond1=>condition: YES or NO?
sub=>subroutine: 子程序
e=>end

st->op1->cond1
cond1(yes)->e
cond1(no)->sub(right)->op1  
```