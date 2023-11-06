# OS Project
### client-server 모델을 이용한 concurrent 파일 서버 구현

# 순서

1. 프로젝트 소개
2. 기능
3. 요구 사항
4. 원천코드
5. 원천코드 작동 방법
6. 사용예제

# 프로젝트 소개

이 프로젝트는 client-server 구조를 기반으로 하고, 프로세스 간 통신을 위해 named pipe(명명된 파이프)를 사용하여 concurrent 파일 서버를 구현한 것이다. server는 client 요청을 수신하고, 파일 액세스, 파일 읽기 또는 쓰기 작업을 요청하고 서버와 상호 작용 할 수 있다.

# 기능

· 리눅스 환경에서 작동하는 client-server 모델


· named pipe(FIFO)를 사용한 프로세스 간 통신


· 다중 클라이언트 요청을 동시에 처리하는 서버


· client는 파일 액세스,파일 읽기 및 쓰기 작업 요청 가능


· 사용자 정의 사용자 인터페이스



# 요구사항

·리눅스 환경


·GCC

# 원천코드

client.c code

``` #include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <sys/wait.h>

#define FIFO_PATH "myfifo"

int main() {
    int server_fd = open(FIFO_PATH, O_WRONLY);

    if (server_fd == -1) {
        perror("Failed to open the server FIFO");
        return 1;
    }

    char request[256];
    char response[256];

    while (1) {
        printf("File name, file access type (r/w), data (or terminated with 'exit'): ");
        fgets(request, sizeof(request), stdin);
        request[strcspn(request, "\n")] = '\0'; 

        if (strcmp(request, "exit") == 0) {
            close(server_fd);
            break;
        }

        write(server_fd, request, strlen(request) + 1);

        memset(response, 0, sizeof(response));
        read(server_fd, response, sizeof(response));
        printf("Server Response: %s\n", response);
    }

    return 0;
}

```



server.c code

``` #include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <sys/wait.h>

#define FIFO_PATH "myfifo"

int main() {
    mkfifo(FIFO_PATH, 0666);
    int server_fd = open(FIFO_PATH, O_RDWR);

    char request[256];
    char response[256];

    while (1) {
        memset(request, 0, sizeof(request));
        read(server_fd, request, sizeof(request));

        if (strcmp(request, "exit") == 0) {
            close(server_fd);
            unlink(FIFO_PATH);
            break;
        }

        char filename[256];
        char access_type[2];
        char data[256];
        sscanf(request, "%s,%c,%s", filename, access_type, data);

        if (access_type[0] == 'r') {
            FILE* file = fopen(filename, "r");
            if (file == NULL) {
                strcpy(response, "Server: File could not be opened");
            }
            else {
                fgets(response, sizeof(response), file);
                fclose(file);
            }
        }
        else if (access_type[0] == 'w') {
            FILE* file = fopen(filename, "w");
            if (file == NULL) {
                strcpy(response, "Server: File could not be opened");
            }
            else {
                fprintf(file, "%s", data);
                fclose(file);
                strcpy(response, "Server: File write complete");
            }
        }
        else {
            strcpy(response, "Server: Invalid request");
        }

        write(server_fd, response, strlen(response) + 1);
    }

    return 0;
}

```

# 원천코드 작동하는 방법

1.컴파일

``` gcc server.c -o server
gcc client.c -o client
```

2.사용법

1) 서버 실행
``` ./ server
```
2) 새 터미널 창을 열고 client를 실행
``` ./client
```

3) 파일 액세스, 읽기 또는 쓰기 작업을 요청하기 위해 프롬포트의 지시를 따른다.

4) client를 종료하려면 "exit" 를 입력


# 사용 예제

1. 파일 액세스 요청:


·파일 이름, 액세스 유형(r 또는 w) 및 데이터(쓰기인 경우)를 입력

·예: 'file.txt,r,20'

2. 서버 응답


·서버는 요청한 데이터 또는 오류 메시지로 응답


3.종료:


·client 를 종료하려면 "exit"를 입력


