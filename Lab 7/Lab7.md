# Ex 1

Scrieti un program care gestioneaza accesul la un numar finit de resurse.
Mai multe fire de executie pot cere concomitent o parte din resurse pe care
le vor da inapoi o data ce nu le mai sunt utile. Definim numarul maxim
de resurse dat:

```c
# define MAX_RESOURCES 5
int available_resources = MAX_RESOURCES ;
```

Cand un thread doreste sa obtina un numar de resurse, acesta apeleaza
decrease_count.

```c
int decrease_count ( int count )
{
  if ( available_resources < count )
      return -1;
  else
      available_resources -= count ;
  return 0;
}
```

iar cand resursele nu-i mai sunt necesare apeleaza increase_count

```c
int increase_count ( int count )
{
    available_resources += count ;
    return 0;
}
```

Functiile de mai sus prezinta mai multe defecte intr-un mediu de executie
paralel printre care si un race condition. Modificati functiile si rezolvati
race condition-ul folosind obiecte de tip mutex. Aratati ca modificarile
dumneavoastra sunt corecte cu ajutorul unui program care porneste mai
multe thread-uri ce consuma un numar diferit de resurse fiecare.

```ssh_session
gcc ex1.c -o ex1
./ex1
```

ex1.c

```c
#include <stdio.h>
#include <pthread.h>

#define MAX_RESOURCES 5

int available_resources = MAX_RESOURCES;
pthread_mutex_t mtx;

int decrease_count(int count) {
    if (available_resources < count) {
        return -1;
    } else {
        available_resources -= count;
        return 0;
    }
}

int increase_count(int count) {
    available_resources += count;
    return 0;
}

void* thread_function(void* arg) {
    int requested_resources = *((int*)arg);

    pthread_mutex_lock(&mtx);
    if (decrease_count(requested_resources) == 0) {
        printf("Got %d resources, %d remaining\n", requested_resources, available_resources);
        increase_count(requested_resources);
        printf("Released %d resources, %d remaining\n", requested_resources, available_resources);
    } else {
        printf("Not enough resources for %d\n", requested_resources);
    }
    pthread_mutex_unlock(&mtx);

    return NULL;
}

int main() {
    pthread_mutex_init(&mtx, NULL);

    pthread_t threads[5];
    int thread_resources[5] = {2, 2, 1, 3, 2};

    printf("MAX_RESOURCES = %d\n", MAX_RESOURCES);

    for (int i = 0; i < 5; i++) {
        pthread_create(&threads[i], NULL, thread_function, &thread_resources[i]);
    }

    for (int i = 0; i < 5; i++) {
        pthread_join(threads[i], NULL);
    }

    pthread_mutex_destroy(&mtx);

    return 0;
}

```


# Ex 2 

Scrieti un program care sa sincronizeze executia a N fire de executie construind un obiect de tip bariera. Bariera va fi initializata folosind init(N)
si fiecare thread va apela barrier point() cand va ajunge in dreptul barierei. Cand functia este apelata a N-a oara, aceasta porneste executia
tuturor firelor in asteptare.
Verificati rezultatele dumneavoastra cu ajutorul unui program care porneste
mai multe thread-uri ce se folosesc de bariera pentru a-si sincroniza executia.
Functia executata de fiecare fir poate avea urmatoarea forma:

```c
void * tfun ( void * v )
{
  int * tid = ( int *) v ;
  printf ("% d reached the barrier \ n " , * tid );
  barrier_point ();
  printf ("% d passed the barrier \ n " , * tid );
  free ( tid );
  return NULL ;
}
```

unde tid este numarul threadului pornit

Indiciu: pentru a implementa obiectul de tip bariera folositi un mutex
pentru contorizarea firelor ajunse la bariera si un semafor pentru a astepta
la bariera.


```ssh-session
gcc ex2.c -o ex2
./ex2
```


ex2.c

```c
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <semaphore.h>

pthread_mutex_t mutex;
sem_t sem;
int count = 0, total_threads;

void barrier_init(int n) {
    total_threads = n;
    pthread_mutex_init(&mutex, NULL);
    sem_init(&sem, 0, 0);
}

void barrier_point() {
    pthread_mutex_lock(&mutex);
    count++;
    if (count < total_threads) {
        pthread_mutex_unlock(&mutex);
        sem_wait(&sem);
    } else {
        for (int i = 1; i < total_threads; i++) {
            sem_post(&sem);
        }
        count = 0; // Reset pentru reutilizare
        pthread_mutex_unlock(&mutex);
    }
}

void *tfun(void *v) {
    int tid = *(int *)v;
    printf("%d reached the barrier\n", tid);
    barrier_point();
    printf("%d passed the barrier\n", tid);
    free(v);
    return NULL;
}

int main() {
    int number_of_threads = 5;
    pthread_t threads[number_of_threads];

    printf("Number of threads = %d\n", number_of_threads);

    barrier_init(number_of_threads);

    for (int i = 0; i < number_of_threads; i++) {
        int *tid = malloc(sizeof(int));
        if (tid == NULL) {
            perror("Failed to allocate memory");
            exit(EXIT_FAILURE);
        }
        *tid = i;
        pthread_create(&threads[i], NULL, tfun, tid);
    }

    for (int i = 0; i < number_of_threads; i++) {
        pthread_join(threads[i], NULL);
    }

    pthread_mutex_destroy(&mutex);
    sem_destroy(&sem);

    return 0;
}
```
