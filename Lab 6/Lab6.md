# Ex 1

Scrieti un program care primeste un sir de caractere la intrare, ale carui
caractere le copiaza in ordine inversa si le salveaza intr-un sir separat.
Operatia de inversare va avea loc intr-un thread separat. Rezultatul va fi
obtinut cu ajutorul functiei pthread_join.

```ssh_session
gcc ex1.c -o ex1
```

```ssh-session
./ex1 hello
```

ex1.c

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/wait.h>
#include <errno.h>
#include <fcntl.h>
#include <sys/mman.h>
#include <pthread.h>
#include <string.h>

char* reverseString(char str[])
{
	int length = strlen(str);
	int start = 0;
	int end = length - 1;
	
	while (start < end){
		char temp = str[start];
		str[start] = str[end];
		str[end] = temp;
		start++;
		end--;
	}
	
	return str;
}

void* thread_reverse(void* arg){
	char* str = (char*)arg;
	reverseString(str);
	return (void*)str;
}

int main(int argc, char* argv[])
{
// sa verific ca sunt param
	pthread_t thr;
	void* result;
	
	if(pthread_create(&thr, NULL, thread_reverse, argv[1])){
		perror("Nu s-a putut crea thread-ul");
		return 1;
	}
	
	// asteptam finalizarea threadului si preluam rezultatul
	if(pthread_join(thr, &result)){
		perror("Eroare la join thread");
		return 1;
	}
	
	printf("Sirul inversat este: %s\n", (char*)result);
	
	return 0;
}
```

# Ex 2 

Scrieti un program care sa calculeze produsul a doua matrice date (de
dimensiuni compatibile) unde fiecare element al matricei rezultate este
calculat de catre un thread distinct.

```ssh-session
gcc ex2.c -o ex2
```

```ssh-session
./ex2
```

ex2.c

```c
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <errno.h>

int firstMatrix[2][3] = {{1, 4, 3}, {3, 2, 1}};
int secondMatrix[3][3] = {{1, 2, 3}, {1, 2, 3}, {1, 2, 3}};
int resultMatrix[2][3];

// calcularea fiecarui element din rezultat
void* calculate_element(void* v) {
    int *args = (int *)v;
    int i = args[0]; 
    int j = args[1];
    int colsFirst = args[2];
    int *s = malloc(sizeof(int));
    *s = 0;
    
    for (int k = 0; k < colsFirst; ++k) {
        *s += firstMatrix[i][k] * secondMatrix[k][j];
    }
    return s;
}

void displayMatrix(int rows, int cols) {
    for (int i = 0; i < rows; ++i) {
        for (int j = 0; j < cols; ++j) {
            printf("%d ", resultMatrix[i][j]);
        }
        printf("\n");
    }
}

int main(int argc, char *argv[]) {
    int rowsFirst = 2;
    int colsFirst = 3;
    int rowsSecond = 3;
    int colsSecond = 3;

    // ne asiguram ca sunt compatibile pentru inmultire
    if (colsFirst != rowsSecond) {
        fprintf(stderr, "Matricele sunt incompatibile pentru inmultire.\n");
        return -1;
    }

    int argumente[rowsFirst * colsSecond][3]; //array de vectori cu cate 3 valori
    pthread_t thr[rowsFirst][colsSecond]; // vector de thread-uri pt fiecare element din rezultat

    // cream cate un thread pentru fiecare element rezultat
    for (int i = 0; i < rowsFirst; ++i) {
        for (int j = 0; j < colsSecond; ++j) {
            argumente[i * colsSecond + j][0] = i;
            argumente[i * colsSecond + j][1] = j;
            argumente[i * colsSecond + j][2] = colsFirst;

			// cream un thread pt a calcula elem [i][j] din rezultat
            if (pthread_create(&thr[i][j], NULL, calculate_element, argumente[i * colsSecond + j])) {
                perror("Eroare la crearea thread-ului");
                return errno;
            }
        }
    }

    for (int i = 0; i < rowsFirst; ++i) {
        for (int j = 0; j < colsSecond; ++j) {
            int *result;
            // asteptam finalizarea thread-ului si preluam rezultatul
            if (pthread_join(thr[i][j], (void**)&result)) {
                perror("Eroare la join thread");
                return errno;
            }
            resultMatrix[i][j] = *result;
            free(result); //eliberam memoria alocata pt result
        }
    }

    printf("Rezultatul înmultirii matricelor:\n");
    displayMatrix(rowsFirst, colsSecond);

    return 0;
}
```
