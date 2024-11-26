# Ex 1

Rescrieti programul HelloWorld de data trecuta folosind numai functii sistem.

```sh-session
gcc ex1.c  -o ex1
./ex1
```

ex1.c 

```c
#include <unistd.h>

int main()
{
	char message[] = "Hello, World!\n";
	write(1, message, sizeof(message) - 1);
	
	// 1 inseamna output in consola
	
	return 0;
}
```

# Ex 2

Scrieti un program care sa primeasca la intrare in primul argument
un fisier sursa pe care sa-l copieze intr-un alt fisier cu numele primit in al
doilea argument.

```sh-session
gcc ex2.c  -o ex2
./ex2 fisier1.txt fisier2.txt
```


```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <fcntl.h>
#include <errno.h>

int main(int argc, char *argv[]) {
	
	char buffer[4096];
	int source, destination;
	ssize_t bytes;

	source = open(argv[1], O_RDONLY);
	if(source < 0)
	{
	perror("Error opening source");
	return errno;
	}
	
	destination = open(argv[2], O_WRONLY | O_CREAT | O_TRUNC, 0666);
	if(destination < 0)
	{
	perror("Error opening destination");
	return errno;
	}
	printf("%lu\n",sizeof(buffer));
	while((bytes = read(source, buffer, sizeof(buffer)))>0)
	{
	write(destination, buffer, bytes);
	}
	
	close(source);
	close(destination);

}
```
