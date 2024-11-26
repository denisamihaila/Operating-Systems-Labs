# Ex 1 + Ex 2

Ipoteza Collatz spune ca plecand de la orice numar natural daca aplicam
repetat urmatoarea operatie


f(n) = n / 2 , daca n este par

f(n) = n * 3 + 1, daca n este impar


sirul ce rezulta va atinge valoarea 1. Implementati un program care sa
testeze ipoteza Collatz pentru mai multe numere date folosind memorie
partajata.

Indicatii: Pornind de la un singur proces parinte, este creat cate un
copil care se ocupa de un singur numar si scrie sirul rezultant undeva
in memoria partajata. Parintele va crea obiectul de memorie partajata
folosind shm_open(3) si ftruncate(2) si pe urma va incarca in memorie
intreg spatiul pentru citirea rezultatelor cu mmap(2).
O conventie trebuie stabilita intre parinte si copii, astfel incat fiecare copil
sa aiba acces exclusiv la o parte din memoria partajata unde isi va scrie
datele (ex. impartim memoria in mod egal pentru fiecare copil). Astfel, fiecare copil va incarca doar zona dedicata lui pentru scriere folosind
dimensiunea cuvenita si un deplasament nenul in mmap(2). Parintele va
astepta ca fiecare copil sa termine executia, dupa care va scrie pe ecran
rezultatele obtinute de copiii sai.
Programul va demonstra acest comportament folosind functiile getpid(2)
si getppid(2).


In programul anterior folositi shm_unlink(3) si munmap(2) pentru a elibera resursele folosite.


```ssh-session
gcc ex1.c -o ex1
```


```ssh-session
./ex1 9 16 25 36
```


ex1.c

```c
#include <stdio.h> // pt input ouput (ex. printf)
#include <stdlib.h> // pt exit, atoi
#include <unistd.h> // pt fork, getpid
#include <sys/mman.h> // pt mmap, munmap 
#include <sys/stat.h> // pt S_IRUSR, S_IWUSR
#include <fcntl.h> // pt shm_open, O_CREAT
#include <sys/wait.h> // pt wait
#include <string.h> // pt strcat
#include <errno.h> // pt perror, errno

void collatz(int n, char* result){
	char buffer[256];
	
	snprintf(buffer, sizeof(buffer), "%d: ", n);
	strcat(result, buffer);
	
	while(n != 1){
		snprintf(buffer, sizeof(buffer), "%d ", n);
		strcat(result, buffer);
		if(n % 2 == 0){
			n /= 2;
		}
		else{
			n = 3 * n + 1;
		}
	}
	strcat(result, "1.\n");
}

int main(int argc, char *argv[]) {
	if(argc < 2){
		return errno;
	}
	char shm_name[] = "myshm"; // numele memoriei partajate
	int shm_fd; // descriptorul memoriei partajate
	int page_size = getpagesize(); // dimensiunea unei pagini de memorie, in bytes
	int num_numbers = argc - 1; // nr de argumente (numere)
	size_t shm_size = num_numbers * page_size; //dimensiunea memoriei partajate (o pagina per numar)
	pid_t pid;
	
	// creare memorie partajata
	// S_IRUSR = permisiune de citire pt utilizatorul care a creat obiectul
	// S_IWUSR = permisiune de citire pt utilizatorul care a creat obiectul
	shm_fd = shm_open(shm_name, O_CREAT | O_RDWR, S_IRUSR | S_IWUSR);
	
	if(shm_fd < 0){
		perror("Error at shm_open");
		return errno;
	}
	
	// modificam dimensiunea memoriei partajate la dimensiunea shm_size
	if(ftruncate(shm_fd, shm_size) == -1){
		perror("Error at ftruncate");
		shm_unlink(shm_name); // stergem memoria partajata daca avem eroare
		return errno;
	}
	// mapeaza obiectul de memorie in spatiul procesuui 
	void *shm_ptr = mmap(NULL, shm_size, PROT_READ | PROT_WRITE, MAP_SHARED, shm_fd, 0);
	
	if(shm_ptr == MAP_FAILED){
		perror("Error at mmap");
		shm_unlink(shm_name);
		return errno;
	}
	
	//creem un proces copil pt fiecare numar
	for(int i = 1; i <= num_numbers; i++) {
		// procesul copil
		if((pid = fork()) == 0){
			// adresa din memoria partajata pt copilul respectiv
			char *child_ptr = (char*)shm_ptr + (i - 1) * page_size;
			int number = atoi(argv[i]);
			collatz(number, child_ptr);
			printf("Done Parent %d Me %d\n", getppid(), getpid()); //pid-ul parintelui si al copilului
			exit(0); // termina procesul copil
		} else if (pid < 0) {
            perror("fork");
            munmap(shm_ptr, shm_size);
            shm_unlink(shm_name);
            return errno;
        }
	}
	
	//procesul parinte asteapta sa se termine toate procesele copil
	for(int i = 0; i < num_numbers; i++){
		wait(NULL);
	}
	
	for(int i = 0; i < num_numbers; i++){
	printf("%s", (char*)shm_ptr + i * page_size);
	}
	
	munmap(shm_ptr, shm_size); // elibereaza zona de memorie partajata
	shm_unlink(shm_name); // sterge obiectul de memorie partajata din sistem
	
	return 0;
}
```
