#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <semaphore.h>
#include <unistd.h>
#include <time.h>

#define N 5 // Ukupan broj mjesta na vrtuljku

sem_t mutex;             // Zaštita dijela koda od višestrukih pristupa
sem_t posjetitelji_sem;  // Semafor za čekanje posjetitelja
sem_t vrtuljak_puni_sem; // Semafor za čekanje punog vrtuljka
sem_t vrtuljak_prazni_sem; // Semafor za čekanje praznog vrtuljka

int broj_posjetitelja = 0; // Broj posjetitelja na vrtuljku

void *posjetitelj(void *arg) {
    srand(time(NULL));
    sleep(rand() % 3); // Simulacija dolaska posjetitelja
    printf("Posjetitelj dolazi.\n");

    sem_wait(&mutex);
    if (broj_posjetitelja < N) {
        broj_posjetitelja++;
        printf("Posjetitelj se ukrcava na vrtuljak. Broj posjetitelja: %d\n", broj_posjetitelja);
        if (broj_posjetitelja == N) {
            sem_post(&vrtuljak_puni_sem); // Signalizacija da je vrtuljak pun
        }
    } else {
        printf("Vrtuljak je pun, posjetitelj se vraća kasnije.\n");
    }
    sem_post(&mutex);

    sem_wait(&posjetitelji_sem); // Čekanje na signal za vožnju

    // Simulacija vožnje vrtuljka
    printf("Posjetitelj se vozi na vrtuljku...\n");
    sleep(3); 

    // Iskrcavanje posjetitelja
    sem_wait(&mutex);
    broj_posjetitelja--;
    printf("Posjetitelj je sišao s vrtuljka. Broj posjetitelja na vrtuljku: %d\n", broj_posjetitelja);
    if (broj_posjetitelja == 0) {
        printf("Vrtuljak je prazan. Vožnja je završena.\n");
        sem_post(&vrtuljak_prazni_sem); // Signalizacija da su svi posjetitelji sišli
    }
    sem_post(&mutex);

    return NULL;
}

void *vrtuljak(void *arg) {
    while (1) {
        sem_wait(&vrtuljak_puni_sem); // Čekanje na pun vrtuljak
        printf("Vrtuljak je pokrenut!\n");

        // Simulacija vožnje vrtuljka
        printf("Vrtuljak se vozi...\n");
        sleep(5); 

        printf("Vožnja je gotova. Posjetitelji se iskrcavaju.\n");

        // Signalizacija posjetiteljima da se iskrcaju
        for (int i = 0; i < N; i++) {
            sem_post(&posjetitelji_sem);
        }

        // Čekanje da se svi posjetitelji iskrcaju
        sem_wait(&vrtuljak_prazni_sem);
    }
}

int main() {
    pthread_t posjetitelji[10]; // Simulacija dolaska 10 posjetitelja
    pthread_t vrtuljak_thread;

    sem_init(&mutex, 0, 1);
    sem_init(&posjetitelji_sem, 0, 0);
    sem_init(&vrtuljak_puni_sem, 0, 0);
    sem_init(&vrtuljak_prazni_sem, 0, 0);

    pthread_create(&vrtuljak_thread, NULL, vrtuljak, NULL);

    for (int i = 0; i < 10; i++) {
        pthread_create(&posjetitelji[i], NULL, posjetitelj, NULL);
    }

    for (int i = 0; i < 10; i++) {
        pthread_join(posjetitelji[i], NULL);
    }

    pthread_join(vrtuljak_thread, NULL);

    sem_destroy(&mutex);
    sem_destroy(&posjetitelji_sem);
    sem_destroy(&vrtuljak_puni_sem);
    sem_destroy(&vrtuljak_prazni_sem);

    return 0;
}