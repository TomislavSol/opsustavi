#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <unistd.h>

#define BROJ_FILOZOFA 5

pthread_mutex_t mutex;
pthread_cond_t cond[BROJ_FILOZOFA];

char stanje_filozofa[BROJ_FILOZOFA] = {'o', 'o', 'o', 'o', 'o'}; // 'o' - razmišlja, 'O' - čeka, 'X' - jede

void udji_u_kriticni_odsjek() {
    pthread_mutex_lock(&mutex);
}

void izadji_iz_kriticnog_odsjeka() {
    pthread_mutex_unlock(&mutex);
}

void cekaj_u_redu(int id_filozofa) {
    pthread_cond_wait(&cond[id_filozofa], &mutex);
}

void signaliziraj(int id_filozofa) {
    pthread_cond_signal(&cond[id_filozofa]);
}

void misli() {
    printf("Filozof razmišlja\n");
    sleep(3);
}

void jede(int id_filozofa) {
    udji_u_kriticni_odsjek();
    stanje_filozofa[id_filozofa] = 'X';
    printf("Filozof %d jede\n", id_filozofa);
    printf("Stanje filozofa: ");
    for (int i = 0; i < BROJ_FILOZOFA; i++) {
        printf("%c ", stanje_filozofa[i]);
    }
    printf("\n");
    izadji_iz_kriticnog_odsjeka();
    sleep(2);
}

void uzmi_vilice(int id_filozofa) {
    udji_u_kriticni_odsjek();
    stanje_filozofa[id_filozofa] = 'O';
    printf("Filozof %d čeka\n", id_filozofa);
    while (stanje_filozofa[id_filozofa] == 'O' && (stanje_filozofa[(id_filozofa + 1) % BROJ_FILOZOFA] == 'X' || stanje_filozofa[(id_filozofa + 4) % BROJ_FILOZOFA] == 'X')) {
        cekaj_u_redu(id_filozofa);
    }
    stanje_filozofa[id_filozofa] = 'X';
    printf("Filozof %d uzima vilice\n", id_filozofa);
    printf("Stanje filozofa: ");
    for (int i = 0; i < BROJ_FILOZOFA; i++) {
        printf("%c ", stanje_filozofa[i]);
    }
    printf("\n");
    izadji_iz_kriticnog_odsjeka();
}

void vrati_vilice(int id_filozofa) {
    udji_u_kriticni_odsjek();
    stanje_filozofa[id_filozofa] = 'o';
    printf("Filozof %d vraća vilice\n", id_filozofa);
    printf("Stanje filozofa: ");
    for (int i = 0; i < BROJ_FILOZOFA; i++) {
        printf("%c ", stanje_filozofa[i]);
    }
    printf("\n");
    signaliziraj((id_filozofa + 1) % BROJ_FILOZOFA);
    signaliziraj((id_filozofa + BROJ_FILOZOFA - 1) % BROJ_FILOZOFA);
    izadji_iz_kriticnog_odsjeka();
}

void *filozof(void *arg) {
    int id_filozofa = *(int *)arg;

    while (1) {
        misli();
        uzmi_vilice(id_filozofa);
        jede(id_filozofa);
        vrati_vilice(id_filozofa);
    }

    return NULL;
}

int main() {
    pthread_t threads[BROJ_FILOZOFA];
    int id_filozofa[BROJ_FILOZOFA];

    pthread_mutex_init(&mutex, NULL);
    for (int i = 0; i < BROJ_FILOZOFA; i++) {
        pthread_cond_init(&cond[i], NULL);
        id_filozofa[i] = i;
        pthread_create(&threads[i], NULL, filozof, &id_filozofa[i]);
    }

    for (int i = 0; i < BROJ_FILOZOFA; i++) {
        pthread_join(threads[i], NULL);
        pthread_cond_destroy(&cond[i]);
    }
    pthread_mutex_destroy(&mutex);

    return 0;
}
