#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/wait.h>
#include <sys/shm.h>

#define N 2  // Broj procesa

//stvaranje dva paralelna procesa koji koriste Dekkerov algoritam za međusobno isključivanje prilikom ulaska u kritični odsječak

int *PRAVO;  // Kome je prednost (0 ili 1)
int *ZASTAVICA;  // Polje zastavica za procese

void udi_u_kriticni_odsjecak(int i, int j) {
    ZASTAVICA[i] = 1;  // Postavi zastavicu da želi ući
    while (ZASTAVICA[j]) {  // Dok drugi proces želi ući
        if (*PRAVO == j) {  // Ako drugi proces ima prednost
            ZASTAVICA[i] = 0;  // Odustani
            while (*PRAVO == j);  // Čekaj dok ne dobiješ prednost
            ZASTAVICA[i] = 1;  // Ponovo pokušaj ući
        }
    }
}

void izadi_iz_kriticnog_odsjecka(int i, int j) {
    *PRAVO = j;  // Daj prednost drugom procesu
    ZASTAVICA[i] = 0;  // Signaliziraj izlazak
}

void proces(int i) {
    int j = (i + 1) % N;  // Drugi proces
    for (int k = 1; k <= 5; k++) {
        udi_u_kriticni_odsjecak(i, j);  // Uđi u kritični odsječak

        // Kritični odsječak
        for (int m = 1; m <= 5; m++) {
            printf("Proces %d, k = %d, m = %d\n", i, k, m);
            sleep(1);  // Usporavanje izvršavanja
        }

        izadi_iz_kriticnog_odsjecka(i, j);  // Izađi iz kritičnog odsječka
    }
}

int main() {
    //shmget za alokaciju dijeljene memorije za varijable PRAVO i ZASTAVICA
    int shm_id1 = shmget(IPC_PRIVATE, sizeof(int), IPC_CREAT | 0666);
    int shm_id2 = shmget(IPC_PRIVATE, N * sizeof(int), IPC_CREAT | 0666);

    //shmat mapira u adresni prostor procesa
    PRAVO = (int *)shmat(shm_id1, NULL, 0);
    ZASTAVICA = (int *)shmat(shm_id2, NULL, 0);

    *PRAVO = 0;
    for (int i = 0; i < N; i++) {
        ZASTAVICA[i] = 0;
    }

    //fork za stvaranje novih procesa
    //Svaki proces izvršava funkciju proces koja simulira ulazak i izlazak iz kritičnog odsječka
    for (int i = 0; i < N; i++) {
        pid_t pid = fork();
        if (pid == 0) {
            proces(i);
            exit(0);
        }
    }

    for (int i = 0; i < N; i++) {
        wait(NULL);  // Čekaj da svi procesi završe
    }

    shmdt(PRAVO);
    shmdt(ZASTAVICA);
    shmctl(shm_id1, IPC_RMID, NULL);
    shmctl(shm_id2, IPC_RMID, NULL);

    return 0;
}