# osi7-Kang-BPE218
##  7-ая домашняя работа Кан Олеся
### Код программы-клиента на языке си
```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <fcntl.h>
#include <sys/mman.h>
#include <sys/stat.h>

int main(int argc, char *argv[]) {
    int mem_size = 4096;
    char shared_mem_name[] = "my_shared_memory.c";                      // Путь до сервера
    int fd;
    char *ptr;
    struct stat sbuf;

    fd = shm_open(shared_mem_name, O_RDWR, 0666);                       // Создаю разеляемую память
    if (fd == -1) {
        perror("Can't create shared memory!\n");
        exit(EXIT_FAILURE);
    }

    if (fstat(fd, &sbuf) == -1) {                                       // Получаю информацию об объекте разделяемой памяти
        perror("fstat error");
        exit(EXIT_FAILURE);
    }

    ptr = mmap(0, mem_size, PROT_WRITE|PROT_READ, MAP_SHARED, fd, 0);   // Отображаю в адресное пространство процесса объект разделяемой памяти
    if (ptr == (char*)-1 ) {
        printf("Error getting pointer to shared memory\n");
        perror("mmap");
        return 1;
    }

    srand(time(NULL));                                                  // Генерирую слцучайные числа по заданной памяти и передаю их на сервер
    for (int i = 0; i < atoi(argv[1]); i++) {
        int num = rand() % atoi(argv[2]) + 1;
        sprintf(ptr, "%d", num);
        ptr += sizeof(int);
        printf("Sent: %d\n", num);
        sleep(1);
    }

    if (close(fd) == -1) {                                              // Отключаю клиент от сервера
        perror("close");
        exit(EXIT_FAILURE);
    }

    return 0;
}
```
### Код программы-сервера на языке си
```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <fcntl.h>
#include <sys/mman.h>

int main() {
    int fd;
    char *ptr;
    struct stat sbuf;
    int mem_size = 4096;
    char shared_mem_name[] = "my_shared_memory.c";

    fd = shm_open(shared_mem_name, O_RDWR | O_CREAT, 0666);                       // Открываю разделяемую память
    if (fd == -1) {
        perror("shm_open");
        exit(EXIT_FAILURE);
    }

    if (ftruncate(fd, mem_size) == -1) {                                          // Изменяю размер сегмента разделяемой памяти
        perror("ftruncate");
        exit(EXIT_FAILURE);
    }

    if (fstat(fd, &sbuf) == -1) {                                                 // Получаю информацию об объекте разделяемой памяти
        perror("fstat");
        exit(EXIT_FAILURE);
    }

    ptr = mmap(0, mem_size, PROT_WRITE|PROT_READ, MAP_SHARED, fd, 0);             // Отображаю разделяемую память в адресное пространство процесса
    if (ptr == (char*)-1 ) {
        printf("Error getting pointer to shared memory\n");
        return 1;
    }

    while (1) {                                                                   // Вывожу на экран переданные данные
        int num = atoi(ptr);
        if (num == -1) {
            break;
        }
        printf("%d\n", num);
        ptr += sizeof(int);
        sleep(1);
    }

    if (munmap(ptr, sbuf.st_size) == -1) {
        perror("munmap");
        exit(EXIT_FAILURE);
    }

    if (close(fd) == -1) {                                                          // Отключаю сервер
        perror("close");
        exit(EXIT_FAILURE);
    }

    if (shm_unlink(shared_mem_name) == -1) {                                        // освобождаю память
        perror("shm_unlink");
        exit(EXIT_FAILURE);
    }

    return 0;
}
```
## В этой домашней работе требовалось с помощью POSIX в программе-клиенте сгенгерировать массив случайных чисел, с через буффер внести этот массив в разделяемую память, затем, в программе-сервере считать данные из разделяемой памяти, вывести их на экран и отключиться от сервера.
#### Сервер и клиент могут быть запущены на разных терминалах или в разных окнах одного терминала с помощью команды: $ ./server


#### Для запуска клиента необходимо ввести команду: $ ./client <number of messages> <max number>


#### Например: $ ./client 10 100
 

#### После этой команды сгенерируетсят 10 случайных чисел в диапазоне от 1 до 100, с которыми в дальнейшем будет происходить работа.
