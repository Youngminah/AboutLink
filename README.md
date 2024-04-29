# AboutLink
- 링커에 대한 발표자료는 Keynote에 있습니다.
- Keynote에 없는 Selectview Loading에 관한 시뮬레이션은 아래 정리하여 올립니다
- 시뮬레이션은 키노트를 학습 후 보는 걸 추천드립니다 ✨

</br>
</br>



## Selective Loading 시뮬레이션
```c
// main.c 파일입니다.

#include <stdio.h>
#include "addsub.h"
 
int main()
{
    printf("add: %d\n", add(3,3));
    printf("sub: %d\n", sub(3,3));
    
    return 0;
}
```

```c
// muldiv.c 파일입니다.

#include "muldiv.h"
 
int mul(int a, int b) {
    return a * b;
}
 
int div(int a, int b) {
    return a / b;
}
```


```c
// addsub.c 파일입니다.
#include "addsub.h"
 
int add(int a, int b) {
    return a + b;
}
 
int sub(int a, int b) {
    return a - b;
}
```



