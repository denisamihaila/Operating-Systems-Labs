### Pornim OpenBSD și folosim urmatoarele date de logare:

 	login: root
  	password: student

# Ex 1 

Compilati un kernel nou.


Pentru a se compila un kernel nou se executa urmatoarele comenzi:

```ssh-session
cd /sys/arch/$(machine)/compile/GENERIC.MP				- navigarea la sursa kernelului

make obj								

make config

make

make install					

reboot									- repornirea sistemului
```

# Ex 2

Adaugati o functie de sistem noua simpla care sa afiseze ceva pe ecran si
demonstrati ca merge apeland-o dintr-un program. Exemplu apel
syscall(id_functie, "world"). 
Atentie, trebuie sa recompilati kernelul si sa reporniti sistemul de operare cu kernelul nou!


## pas 1: Declararea

Am deschis nano /sys/kern/syscalls.master și am adăugat noua funcție la finalul acestui fișier:

```c
331   STD      { int sys_khello(const char *msg); }
```

apoi apelam comanda :

```ssh_session
cd/sys/kern && make syscalls
```
pentru a regenera fisierele aferente 

  - /sys/kern/syscalls.c
  - /sys/sys/syscallargs.h
  - /sys/sys/syscall.h

## pas 2 : Definirea 

Am adăugat definiția funcției de sistem sys_khello în fișierul nano /sys/kern/sys_generic.c

```c
int sys_khello(struct proc *p, void *v, register_t *retval) {
    struct sys_khello_args *uap = v;
    char kmsg[100];

    copyinstr(SCARG(uap, msg), kmsg, 100, NULL);
    printf("%s\n", kmsg);  // afișează mesajul primit de la user
    return 0;
}

```

## pas 3 : Recompilam kernelul 

pentru acest pas se vor executa comenzile de la ex. 1

## pas 4 : Am creat fișierul test_khello.c și am scris un scurt cod pentru a testa funcția nouă


```c
#include <stdio.h>
#include <unistd.h>
#include <sys/syscall.h>

int main() {
    const char *msg = "Hello from user space!";
    int result = syscall(331, msg);
    if (result == 0) {
        printf("syskhello a fost apelată cu succes!\n");
    } else {
        perror("Eroare la apelarea sys_khello");
    }
    return 0;
}

```

## pas 5 : Generam executabilul

```ssh-session
cc -o test_khello test_khello.c
```

## pas 6 : Apelam executabilul

```ssh-session
./test_khello
```



# Ex 3

Modificati functia de sistem de mai devreme sa copieze un numar dat de
bytes dintr-un buffer sursa intr-altul destinatie. La iesire, functia va scrie
numarul de bytes copiat efectiv. Verificati intrarile primite si semnalati
eventualele erori.




## pas 1: Declararea

modificam fisierul /sys/kern/syscalls.master adaugand :

```c
332   STD      { ssize_t sys_copy(const void *src, void *dst, const size_t n); }
```

apoi apelam comanda :

```ssh_session
cd/sys/kern && make syscalls
```
pentru a regenera fisierele aferente 

  - /sys/kern/syscalls.c
  - /sys/sys/syscallargs.h
  - /sys/sys/syscall.h

## pas 2 : Definirea 

modificam fisierul /sys/kern/sys_generic.c adaugand :

```c
int
sys_copy(struct proc *p, void *v, register_t *retval)
{
	struct sys_copy_args *uap = v;

	void *kdest = (void*) malloc(SCARG(uap, n), M_TEMP, M_WAITOK);

	if(kdest == NULL)
	{
		*retval = 0;
		return ENOMEM;
	}

	int eroare = copyin(SCARG(uap, src), kdest, SCARG(uap, n));

	if(eroare != 0)
	{
		*retval = 0;
		return eroare;
	}

	int eroare1 = copyout(kdest, SCARG(uap, dst), SCARG(uap, n));

	if(eroare1 != 0)
	{
		*retval = 0;
		return eroare;
	}
	
	free(kdest, M_TEMP, SCARG(uap, n));
	*retval = SCARG(uap, n);
	return 0;
}
```


## pas 3 : Recompilam kernelul 

pentru acest pas se vor executa comenzile de la ex. 1

## pas 4 : Scriem un program ca sa apeleze functia

de exemplu : 

ex3.c 

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <errno.h>

int main()
{
	char source[] = "Salutare!";
	char destination[20];

	int ret_value = syscall(332, sourcem destionation, sizeof(source));

	if(ret_value != sizeof(source)) {
		perror("syscall");
		return 1;
	}
	printf("%s\n", destination);
	printf("%d \n", ret_value);
	return 0;
}
```

## pas 5 : Generam executabilul

```ssh-session
cc ex3.c -o ex3
```

## pas 6 : Apelam executabilul

```ssh-session
./ex3
```











