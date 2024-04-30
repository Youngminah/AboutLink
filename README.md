# AboutLink
- 링커에 대한 발표자료는 Keynote에.
- Keynote에 없는 Selectview Loading에 관한 시뮬레이션은 아래 정리하여 올립니다
- 시뮬레이션은 키노트를 학습 후 보는 걸 추천드립니다 ✨

</br>
</br>



## Selective Loading 시뮬레이션
- main.c, muldiv.c, addsub.c 3개의 소스코드가 있다.
  
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
- `main.c` 안에서 쓰는 코드는 add함수과 sub함수 두가지로 `muldiv.c` 파일을 사실상 쓰이지 않는 코드.
- `muldiv.c`, `addsub.c`를 합쳐 `libmint.a`라는 `Static Library`로 만들어 main 실행파일을 만든다.
- `Selective Loading`이 사실이라면 main mach-O파일에 쓰이지 않는 `muldiv.o` 목적파일은 로드되지 않을 것.

### 실행결과
- 우선은 `gcc main.c -lmint -o main -L.`로 일반적인 Static Library로 만들어본다.
- 그리고 main의 mach-O 파일의 _TEXT를 출력결과
```
main:	file format mach-o arm64

Disassembly of section __TEXT,__text:

0000000100003ecc <_main>:
100003ecc: d100c3ff    	sub	sp, sp, #48
100003ed0: a9027bfd    	stp	x29, x30, [sp, #32]
100003ed4: 910083fd    	add	x29, sp, #32
100003ed8: 52800008    	mov	w8, #0
100003edc: b81f83a8    	stur	w8, [x29, #-8]
100003ee0: b81fc3bf    	stur	wzr, [x29, #-4]
100003ee4: 52800061    	mov	w1, #3
100003ee8: b81f43a1    	stur	w1, [x29, #-12]
100003eec: aa0103e0    	mov	x0, x1
100003ef0: 94000014    	bl	0x100003f40 <_add>
100003ef4: 910003e9    	mov	x9, sp
100003ef8: aa0003e8    	mov	x8, x0
100003efc: f9000128    	str	x8, [x9]
100003f00: 90000000    	adrp	x0, 0x100003000 <_main+0x34>
100003f04: 913e3000    	add	x0, x0, #3980
100003f08: 9400001e    	bl	0x100003f80 <_printf+0x100003f80>
100003f0c: b85f43a1    	ldur	w1, [x29, #-12]
100003f10: aa0103e0    	mov	x0, x1
100003f14: 94000013    	bl	0x100003f60 <_sub>
100003f18: 910003e9    	mov	x9, sp
100003f1c: aa0003e8    	mov	x8, x0
100003f20: f9000128    	str	x8, [x9]
100003f24: 90000000    	adrp	x0, 0x100003000 <_main+0x58>
100003f28: 913e5400    	add	x0, x0, #3989
100003f2c: 94000015    	bl	0x100003f80 <_printf+0x100003f80>
100003f30: b85f83a0    	ldur	w0, [x29, #-8]
100003f34: a9427bfd    	ldp	x29, x30, [sp, #32]
100003f38: 9100c3ff    	add	sp, sp, #48
100003f3c: d65f03c0    	ret

0000000100003f40 <_add>:
100003f40: d10043ff    	sub	sp, sp, #16
100003f44: b9000fe0    	str	w0, [sp, #12]
100003f48: b9000be1    	str	w1, [sp, #8]
100003f4c: b9400fe8    	ldr	w8, [sp, #12]
100003f50: b9400be9    	ldr	w9, [sp, #8]
100003f54: 0b090100    	add	w0, w8, w9
100003f58: 910043ff    	add	sp, sp, #16
100003f5c: d65f03c0    	ret

0000000100003f60 <_sub>:
100003f60: d10043ff    	sub	sp, sp, #16
100003f64: b9000fe0    	str	w0, [sp, #12]
100003f68: b9000be1    	str	w1, [sp, #8]
100003f6c: b9400fe8    	ldr	w8, [sp, #12]
100003f70: b9400be9    	ldr	w9, [sp, #8]
100003f74: 6b090100    	subs	w0, w8, w9
100003f78: 910043ff    	add	sp, sp, #16
100003f7c: d65f03c0    	ret

Disassembly of section __TEXT,__stubs:

0000000100003f80 <__stubs>:
100003f80: b0000010    	adrp	x16, 0x100004000 <__stubs+0x4>
100003f84: f9400210    	ldr	x16, [x16]
100003f88: d61f0200    	br	x16
```
- libmint mach-O 파일의 _TEXT를 출력해본다.
```

libmint.a(muldiv.o):	file format mach-o arm64

Disassembly of section __TEXT,__text:

0000000000000000 <ltmp0>:
       0: d10043ff     	sub	sp, sp, #16
       4: b9000fe0     	str	w0, [sp, #12]
       8: b9000be1     	str	w1, [sp, #8]
       c: b9400fe8     	ldr	w8, [sp, #12]
      10: b9400be9     	ldr	w9, [sp, #8]
      14: 1b097d00     	mul	w0, w8, w9
      18: 910043ff     	add	sp, sp, #16
      1c: d65f03c0     	ret

0000000000000020 <_div>:
      20: d10043ff     	sub	sp, sp, #16
      24: b9000fe0     	str	w0, [sp, #12]
      28: b9000be1     	str	w1, [sp, #8]
      2c: b9400fe8     	ldr	w8, [sp, #12]
      30: b9400be9     	ldr	w9, [sp, #8]
      34: 1ac90d00     	sdiv	w0, w8, w9
      38: 910043ff     	add	sp, sp, #16
      3c: d65f03c0     	ret

libmint.a(addsub.o):	file format mach-o arm64

Disassembly of section __TEXT,__text:

0000000000000000 <ltmp0>:
       0: d10043ff     	sub	sp, sp, #16
       4: b9000fe0     	str	w0, [sp, #12]
       8: b9000be1     	str	w1, [sp, #8]
       c: b9400fe8     	ldr	w8, [sp, #12]
      10: b9400be9     	ldr	w9, [sp, #8]
      14: 0b090100     	add	w0, w8, w9
      18: 910043ff     	add	sp, sp, #16
      1c: d65f03c0     	ret

0000000000000020 <_sub>:
      20: d10043ff     	sub	sp, sp, #16
      24: b9000fe0     	str	w0, [sp, #12]
      28: b9000be1     	str	w1, [sp, #8]
      2c: b9400fe8     	ldr	w8, [sp, #12]
      30: b9400be9     	ldr	w9, [sp, #8]
      34: 6b090100     	subs	w0, w8, w9
      38: 910043ff     	add	sp, sp, #16
      3c: d65f03c0     	ret
```
- Selective Loading되어 main 실행파일에는 mulsub.o에 대한 소스코드가 포함되지 않음을 확인하였다 💡
</br>
</br>

- 다음은 `-all_load`옵션을 주고 Static Library로 만들어본다.
- main의 mach-O 파일의 _TEXT를 출력결과
```
main:	file format mach-o arm64

Disassembly of section __TEXT,__text:

0000000100003e8c <_main>:
100003e8c: d100c3ff    	sub	sp, sp, #48
100003e90: a9027bfd    	stp	x29, x30, [sp, #32]
100003e94: 910083fd    	add	x29, sp, #32
100003e98: 52800008    	mov	w8, #0
100003e9c: b81f83a8    	stur	w8, [x29, #-8]
100003ea0: b81fc3bf    	stur	wzr, [x29, #-4]
100003ea4: 52800061    	mov	w1, #3
100003ea8: b81f43a1    	stur	w1, [x29, #-12]
100003eac: aa0103e0    	mov	x0, x1
100003eb0: 94000024    	bl	0x100003f40 <_add>
100003eb4: 910003e9    	mov	x9, sp
100003eb8: aa0003e8    	mov	x8, x0
100003ebc: f9000128    	str	x8, [x9]
100003ec0: 90000000    	adrp	x0, 0x100003000 <_main+0x34>
100003ec4: 913e3000    	add	x0, x0, #3980
100003ec8: 9400002e    	bl	0x100003f80 <_printf+0x100003f80>
100003ecc: b85f43a1    	ldur	w1, [x29, #-12]
100003ed0: aa0103e0    	mov	x0, x1
100003ed4: 94000023    	bl	0x100003f60 <_sub>
100003ed8: 910003e9    	mov	x9, sp
100003edc: aa0003e8    	mov	x8, x0
100003ee0: f9000128    	str	x8, [x9]
100003ee4: 90000000    	adrp	x0, 0x100003000 <_main+0x58>
100003ee8: 913e5400    	add	x0, x0, #3989
100003eec: 94000025    	bl	0x100003f80 <_printf+0x100003f80>
100003ef0: b85f83a0    	ldur	w0, [x29, #-8]
100003ef4: a9427bfd    	ldp	x29, x30, [sp, #32]
100003ef8: 9100c3ff    	add	sp, sp, #48
100003efc: d65f03c0    	ret

0000000100003f00 <_mul>:
100003f00: d10043ff    	sub	sp, sp, #16
100003f04: b9000fe0    	str	w0, [sp, #12]
100003f08: b9000be1    	str	w1, [sp, #8]
100003f0c: b9400fe8    	ldr	w8, [sp, #12]
100003f10: b9400be9    	ldr	w9, [sp, #8]
100003f14: 1b097d00    	mul	w0, w8, w9
100003f18: 910043ff    	add	sp, sp, #16
100003f1c: d65f03c0    	ret

0000000100003f20 <_div>:
100003f20: d10043ff    	sub	sp, sp, #16
100003f24: b9000fe0    	str	w0, [sp, #12]
100003f28: b9000be1    	str	w1, [sp, #8]
100003f2c: b9400fe8    	ldr	w8, [sp, #12]
100003f30: b9400be9    	ldr	w9, [sp, #8]
100003f34: 1ac90d00    	sdiv	w0, w8, w9
100003f38: 910043ff    	add	sp, sp, #16
100003f3c: d65f03c0    	ret

0000000100003f40 <_add>:
100003f40: d10043ff    	sub	sp, sp, #16
100003f44: b9000fe0    	str	w0, [sp, #12]
100003f48: b9000be1    	str	w1, [sp, #8]
100003f4c: b9400fe8    	ldr	w8, [sp, #12]
100003f50: b9400be9    	ldr	w9, [sp, #8]
100003f54: 0b090100    	add	w0, w8, w9
100003f58: 910043ff    	add	sp, sp, #16
100003f5c: d65f03c0    	ret

0000000100003f60 <_sub>:
100003f60: d10043ff    	sub	sp, sp, #16
100003f64: b9000fe0    	str	w0, [sp, #12]
100003f68: b9000be1    	str	w1, [sp, #8]
100003f6c: b9400fe8    	ldr	w8, [sp, #12]
100003f70: b9400be9    	ldr	w9, [sp, #8]
100003f74: 6b090100    	subs	w0, w8, w9
100003f78: 910043ff    	add	sp, sp, #16
100003f7c: d65f03c0    	ret

Disassembly of section __TEXT,__stubs:

0000000100003f80 <__stubs>:
100003f80: b0000010    	adrp	x16, 0x100004000 <__stubs+0x4>
100003f84: f9400210    	ldr	x16, [x16]
100003f88: d61f0200    	br	x16
```
- libmint mach-O 파일의 _TEXT를 출력해본다.
```

libmint.a(muldiv.o):	file format mach-o arm64

Disassembly of section __TEXT,__text:

0000000000000000 <ltmp0>:
       0: d10043ff     	sub	sp, sp, #16
       4: b9000fe0     	str	w0, [sp, #12]
       8: b9000be1     	str	w1, [sp, #8]
       c: b9400fe8     	ldr	w8, [sp, #12]
      10: b9400be9     	ldr	w9, [sp, #8]
      14: 1b097d00     	mul	w0, w8, w9
      18: 910043ff     	add	sp, sp, #16
      1c: d65f03c0     	ret

0000000000000020 <_div>:
      20: d10043ff     	sub	sp, sp, #16
      24: b9000fe0     	str	w0, [sp, #12]
      28: b9000be1     	str	w1, [sp, #8]
      2c: b9400fe8     	ldr	w8, [sp, #12]
      30: b9400be9     	ldr	w9, [sp, #8]
      34: 1ac90d00     	sdiv	w0, w8, w9
      38: 910043ff     	add	sp, sp, #16
      3c: d65f03c0     	ret

libmint.a(addsub.o):	file format mach-o arm64

Disassembly of section __TEXT,__text:

0000000000000000 <ltmp0>:
       0: d10043ff     	sub	sp, sp, #16
       4: b9000fe0     	str	w0, [sp, #12]
       8: b9000be1     	str	w1, [sp, #8]
       c: b9400fe8     	ldr	w8, [sp, #12]
      10: b9400be9     	ldr	w9, [sp, #8]
      14: 0b090100     	add	w0, w8, w9
      18: 910043ff     	add	sp, sp, #16
      1c: d65f03c0     	ret

0000000000000020 <_sub>:
      20: d10043ff     	sub	sp, sp, #16
      24: b9000fe0     	str	w0, [sp, #12]
      28: b9000be1     	str	w1, [sp, #8]
      2c: b9400fe8     	ldr	w8, [sp, #12]
      30: b9400be9     	ldr	w9, [sp, #8]
      34: 6b090100     	subs	w0, w8, w9
      38: 910043ff     	add	sp, sp, #16
      3c: d65f03c0     	ret
```
- `-all_load`시 `addsub.o`, `muldiv.o` 둘다 잘 아카이브되었음을 확인.



</br>
</br>

## Q1. -all_load옵션에 dead_code_strippping 옵션 둘다 주면 어떻게될까? 
- 질문으로 들어온 내용이였다.
- 당시 발표대는 해본적은 없지만 dead_code_stripping이 더 마지막 단계에서 일어나는 연산이라
- 아마 안쓰는 심볼에 대한 내용은 삭제될거라 답은 했었는데, 발표 후에 한번 해보았다.
- `gcc -o main main.o -Wl,-all_load,libmint.a -Wl,-dead_strip` : -all_load 옵션에 dead_code_stripping 까지 적용하여 아카이브
- main 실행파일의 _TEXT출력결과
```

main:	file format mach-o arm64

Disassembly of section __TEXT,__text:

0000000100003ecc <_main>:
100003ecc: d100c3ff    	sub	sp, sp, #48
100003ed0: a9027bfd    	stp	x29, x30, [sp, #32]
100003ed4: 910083fd    	add	x29, sp, #32
100003ed8: 52800008    	mov	w8, #0
100003edc: b81f83a8    	stur	w8, [x29, #-8]
100003ee0: b81fc3bf    	stur	wzr, [x29, #-4]
100003ee4: 52800061    	mov	w1, #3
100003ee8: b81f43a1    	stur	w1, [x29, #-12]
100003eec: aa0103e0    	mov	x0, x1
100003ef0: 94000014    	bl	0x100003f40 <_add>
100003ef4: 910003e9    	mov	x9, sp
100003ef8: aa0003e8    	mov	x8, x0
100003efc: f9000128    	str	x8, [x9]
100003f00: 90000000    	adrp	x0, 0x100003000 <_main+0x34>
100003f04: 913e3000    	add	x0, x0, #3980
100003f08: 9400001e    	bl	0x100003f80 <_printf+0x100003f80>
100003f0c: b85f43a1    	ldur	w1, [x29, #-12]
100003f10: aa0103e0    	mov	x0, x1
100003f14: 94000013    	bl	0x100003f60 <_sub>
100003f18: 910003e9    	mov	x9, sp
100003f1c: aa0003e8    	mov	x8, x0
100003f20: f9000128    	str	x8, [x9]
100003f24: 90000000    	adrp	x0, 0x100003000 <_main+0x58>
100003f28: 913e5400    	add	x0, x0, #3989
100003f2c: 94000015    	bl	0x100003f80 <_printf+0x100003f80>
100003f30: b85f83a0    	ldur	w0, [x29, #-8]
100003f34: a9427bfd    	ldp	x29, x30, [sp, #32]
100003f38: 9100c3ff    	add	sp, sp, #48
100003f3c: d65f03c0    	ret

0000000100003f40 <_add>:
100003f40: d10043ff    	sub	sp, sp, #16
100003f44: b9000fe0    	str	w0, [sp, #12]
100003f48: b9000be1    	str	w1, [sp, #8]
100003f4c: b9400fe8    	ldr	w8, [sp, #12]
100003f50: b9400be9    	ldr	w9, [sp, #8]
100003f54: 0b090100    	add	w0, w8, w9
100003f58: 910043ff    	add	sp, sp, #16
100003f5c: d65f03c0    	ret

0000000100003f60 <_sub>:
100003f60: d10043ff    	sub	sp, sp, #16
100003f64: b9000fe0    	str	w0, [sp, #12]
100003f68: b9000be1    	str	w1, [sp, #8]
100003f6c: b9400fe8    	ldr	w8, [sp, #12]
100003f70: b9400be9    	ldr	w9, [sp, #8]
100003f74: 6b090100    	subs	w0, w8, w9
100003f78: 910043ff    	add	sp, sp, #16
100003f7c: d65f03c0    	ret

Disassembly of section __TEXT,__stubs:

0000000100003f80 <__stubs>:
100003f80: b0000010    	adrp	x16, 0x100004000 <__stubs+0x4>
100003f84: f9400210    	ldr	x16, [x16]
100003f88: d61f0200    	br	x16
```
- 예상대로 안쓰는 mul, sub에 대한 코드는 제거된 _TEXT의 결과임을 확인.
- 하지만, 찾아본바로는 _all_load, dead_code_stripping은 결국 상반되는 행위라서 꼭 필요한지 확인하고 해야함..
- 더 복잡한 코드에서는 예상만큼 성능이 괜찮고 안전하진 않음.
