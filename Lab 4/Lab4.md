# Ex 1

Creati un proces nou folosind fork(2) si afisati fisierele din directorul
curent cu ajutorul execve(2). Din procesul initial afisati pid-ul propriu
si pid-ul copilului. 

```ssh-session
gcc ex1.c -o ex1
```

```ssh-session
./ex1
```


ex1.c

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/wait.h>
#include <errno.h>

int main(){

pid_t pid = fork();

if(pid < 0) {
	return errno;
	}
else if(pid == 0){
	/* child instructions */
	char *argv2[] = {"/bin/ls", "-l", NULL};
	execve("/bin/ls", argv2, NULL);
	}
else {
	/* parent instructions */
	printf("My PID = %d, Child PID = %d\n", getpid(), pid);
	wait(NULL);
	printf("Child %d finished\n", pid);
	}
	
	return 0;
}
```


# Ex 2

Ipoteza Collatz spune ca plecand de la orice numar natural daca aplicam
repetat urmatoarea operatie


f(n) = n / 2 daca n este par
f(n) = n * 3 + 1 daca n este impar


sirul ce rezulta va atinge valoarea 1. Implementati un program care
foloseste fork(2) si testeaza ipoteza generand sirul asociat unui numar
dat in procesul copil.

```ssh-session
gcc ex2.c -o ex2
```

```ssh-session
./ex2 24
```

ex2.c

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/wait.h>
#include <errno.h>

void collatz(int n){
	printf("%d: ", n);
	while(n != 1){
		printf("%d ", n);
		if(n % 2 == 0){
			n /= 2;
		}
		else{
			n = 3 * n + 1;
		}
	}
	printf("1.\n");
}


int main(int argc, char *argv[]){
	int n = atoi(argv[1]);
	
	pid_t pid = fork();
	
	if(pid < 0)
		return errno;
	else if(pid == 0)
		{
		collatz(n);
		printf("Child %d finished\n", getpid());
		}
	else{
	wait(NULL);
	}
	return 0;
}
```


# Ex 3

Implementati un program care sa testeze ipoteza Collatz pentru mai multe
numere date. Pornind de la un singur proces parinte, este creat cate un
copil care se ocupa de un singur numar. Parintele va astepta sa termine
executia fiecare copil. Programul va demonstra acest comportament folosind functiile getpid(2) si getppid(2).


```ssh-session
gcc ex3.c -o ex3
```

```ssh-session
./ex3 24 12 3 4 7 8
```

ex3.c

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/wait.h>
#include <errno.h>

void collatz(int n){
	printf("%d: ", n);
	while(n != 1){
		printf("%d ", n);
		if(n % 2 == 0){
			n /= 2;
		}
		else{
			n = 3 * n + 1;
		}
	}
	printf("1.\n");
}

int main(int argc, char *argv[]){
	printf("Starting parent %d\n", getpid());
	for(int i=1; i<argc; i++){
		
	int n = atoi(argv[i]);
		
	pid_t pid = fork();
	
	if(pid < 0)
		return errno;
	else if(pid == 0)
		{
		collatz(n);
		printf("Done Parent %d Me %d\n", getppid(), getpid());
		exit(0);
		}
	}
	
	while(wait(NULL) > 0); //astept sa se termine toate procesele copil de executat
	printf("All children have finished.\n");
	
	return 0;
}

```
