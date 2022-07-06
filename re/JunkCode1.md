```c
#include <stdio.h>
int main(){
	int a;
	printf("plz ");
	__asm{
		_emit 0EBh
		_emit 0FFh
		_emit 0C0h
        //EB FF 跳转到前一条，此时会执行FF C0，即inc eax
	}
	printf("flag ");
	__asm{
		_emit 0EBh
		_emit 0FFh
		_emit 0C0h
	}
	printf("is: ");
		__asm{
		_emit 0EBh
		_emit 0FFh
		_emit 0C0h
		_emit 0EBh
		_emit 1h
		_emit 66h

		_emit 0EBh
		_emit 0FFh
		_emit 0C0h
		_emit 0EBh
		_emit 1h
		_emit 6ch

		_emit 0EBh
		_emit 0FFh
		_emit 0C0h
		_emit 0EBh
		_emit 1h
		_emit 61h

		_emit 0EBh
		_emit 0FFh
		_emit 0C0h
		_emit 0EBh
		_emit 1h
		_emit 67h

		_emit 0EBh
		_emit 0FFh
		_emit 0C0h
		_emit 0EBh
		_emit 1h
		_emit 7bh

		_emit 0EBh
		_emit 0FFh
		_emit 0C0h
		_emit 0EBh
		_emit 1h
		_emit 65h

		_emit 0EBh
		_emit 0FFh
		_emit 0C0h
		_emit 0EBh
		_emit 1h
		_emit 7ah
		

		_emit 0EBh
		_emit 0FFh
		_emit 0C0h
		_emit 0EBh
		_emit 1h
		_emit 7dh
	}
	return 0;
}
```

