#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <signal.h>

// Globalne varijable
int broj;
const char* status_datoteka = "status.txt";
const char* obrada_datoteka = "obrada.txt";

// Funkcija za obradu broja
int obrada(int broj) {
    return broj * broj; // Primjer: računanje kvadrata broja
}

// Funkcija za odgodu izvršavanja
void odgodi(int sekunde) {
    sleep(sekunde);
}

// Funkcija za rukovanje signalom SIGUSR1
void handle_usr1(int sig) {
    printf("Trenutni broj u obradi: %d\n", broj);
}

// Funkcija za rukovanje signalom SIGTERM
void handle_term(int sig) {
    FILE *status_file = fopen(status_datoteka, "w");
    if (status_file != NULL) {
        fprintf(status_file, "%d", broj);
        fclose(status_file);
    }
    printf("Završavanje rada...\n");
    exit(0);
}

// Funkcija za rukovanje signalom SIGINT
void handle_int(int sig) {
    printf("Prekid rada...\n");
    exit(0);
}

int main() {
    // Maskiranje signala
    sigset_t mask;
    sigemptyset(&mask);
    sigaddset(&mask, SIGUSR1);
    sigaddset(&mask, SIGTERM);
    sigaddset(&mask, SIGINT);
    sigprocmask(SIG_BLOCK, &mask, NULL);

    // Postavljanje handlera signala
    signal(SIGUSR1, handle_usr1);
    signal(SIGTERM, handle_term);
    signal(SIGINT, handle_int);

    // Čitanje broja iz status.txt
    FILE *status_file = fopen(status_datoteka, "r");
    if (status_file != NULL) {
        fscanf(status_file, "%d", &broj);
        fclose(status_file);
    }

    // Ako je broj 0, nastavi obradu iz obrada.txt
    if (broj == 0) {
        FILE *obrada_file = fopen(obrada_datoteka, "r");
        if (obrada_file != NULL) {
            int procitani_broj;
            while (fscanf(obrada_file, "%d", &procitani_broj) != EOF) {
                broj = procitani_broj;
            }
            fclose(obrada_file);
        }
        // Postavljanje statusa na 0
        status_file = fopen(status_datoteka, "w");
        if (status_file != NULL) {
            fprintf(status_file, "0");
            fclose(status_file);
        }
    }

    // Glavna petlja programa
    while (1) {
        // Obrada broja
        broj++;
        int rezultat = obrada(broj);

        // Dodavanje rezultata u obrada.txt
        FILE *obrada_file = fopen(obrada_datoteka, "a");
        if (obrada_file != NULL) {
            fprintf(obrada_file, "%d\n", rezultat);
            fclose(obrada_file);
        }

        // Odgoda izvršavanja
        odgodi(5);
    }

    return 0;
}