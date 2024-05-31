#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <unistd.h>

#define NUM_PHILOSOPHERS 5

pthread_mutex_t mutex;
pthread_cond_t cond[NUM_PHILOSOPHERS];

char philosopher_state[NUM_PHILOSOPHERS] = {'o', 'o', 'o', 'o', 'o'}; // 'o' - razmišlja, 'O' - čeka, 'X' - jede

void enter_critical_section() {
    pthread_mutex_lock(&mutex);
}

void exit_critical_section() {
    pthread_mutex_unlock(&mutex);
}

void wait_in_line(int philosopher_id) {
    pthread_cond_wait(&cond[philosopher_id], &mutex);
}

void signal(int philosopher_id) {
    pthread_cond_signal(&cond[philosopher_id]);
}

void think() {
    printf("Filozof misli\n");
    sleep(3);
}

void eat(int philosopher_id) {
    enter_critical_section();
    philosopher_state[philosopher_id] = 'X';
    printf("Filozof %d jede\n", philosopher_id);
    printf("Stanje filozofa: ");
    for (int i = 0; i < NUM_PHILOSOPHERS; i++) {
        printf("%c ", philosopher_state[i]);
    }
    printf("\n");
    exit_critical_section();
    sleep(2);
}

void take_forks(int philosopher_id) {
    enter_critical_section();
    philosopher_state[philosopher_id] = 'O';
    printf("Filozof %d čeka\n", philosopher_id);
    while (philosopher_state[philosopher_id] == 'O' && (philosopher_state[(philosopher_id + 1) % NUM_PHILOSOPHERS] == 'X' || philosopher_state[(philosopher_id + 4) % NUM_PHILOSOPHERS] == 'X')) {
        wait_in_line(philosopher_id);
    }
    philosopher_state[philosopher_id] = 'X';
    printf("Filozof %d uzima vilice\n", philosopher_id);
    printf("Stanje filozofa: ");
    for (int i = 0; i < NUM_PHILOSOPHERS; i++) {
        printf("%c ", philosopher_state[i]);
    }
    printf("\n");
    exit_critical_section();
}

void return_forks(int philosopher_id) {
    enter_critical_section();
    philosopher_state[philosopher_id] = 'o';
    printf("Filozof %d vraća vilice\n", philosopher_id);
    printf("Stanje filozofa: ");
    for (int i = 0; i < NUM_PHILOSOPHERS; i++) {
        printf("%c ", philosopher_state[i]);
    }
    printf("\n");
    signal((philosopher_id + 1) % NUM_PHILOSOPHERS);
    signal((philosopher_id + NUM_PHILOSOPHERS - 1) % NUM_PHILOSOPHERS);
    exit_critical_section();
}

void *philosopher(void *arg) {
    int philosopher_id = *(int *)arg;

    while (1) {
        think();
        take_forks(philosopher_id);
        eat(philosopher_id);
        return_forks(philosopher_id);
    }

    return NULL;
}

int main() {
    pthread_t threads[NUM_PHILOSOPHERS];
    int philosopher_ids[NUM_PHILOSOPHERS];

    pthread_mutex_init(&mutex, NULL);
    for (int i = 0; i < NUM_PHILOSOPHERS; i++) {
        pthread_cond_init(&cond[i], NULL);
        philosopher_ids[i] = i;
        pthread_create(&threads[i], NULL, philosopher, &philosopher_ids[i]);
    }

    for (int i = 0; i < NUM_PHILOSOPHERS; i++) {
        pthread_join(threads[i], NULL);
        pthread_cond_destroy(&cond[i]);
    }
    pthread_mutex_destroy(&mutex);

    return 0;
}
