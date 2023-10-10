/* FinalProject client file submitted by Viutika Rathod 110094621 Section 3*/

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <unistd.h>

#define PORT 8080
#define mirror_port 7001
#define BUFSIZE 4096


/**
 * Checkng if provided date strng is a valid date in format "YYYY-MM-DD".
 * @param date date string to be validated.
 * @return 1 if date is valid, 0 otherwise.
 */
int chk_valid_date(char* date) {
   
    int year, month, day;
    // Tryng to parse date string into year, month, and day components
    if (sscanf(date, "%d-%d-%d", &year, &month, &day) != 3) {
        return 0;
    }
    // Checkng if parsed components represent a valid date
    if (year < 1 || year > 9999 || month < 1 || month > 12 || day < 1 || day > 31) {
        return 0;
    }
    return 1;
}

int main(int arg_c, char const *arg_v[]) {
    int server_fd_info;
    struct sockaddr_in serv_addr_info, mirror_addr_info; 

    char buf_val[1024] = {0};                 //Buffer for receivng data
    char commandPassd[1024] = {0};           // Buffer for command responses
    char gf_ary[2048]={0};                  // General-purpose buffer
    int valid_syntx;                       // Flag to indicate valid syntax

    // Creatng a socket
    if ((server_fd_info = socket(AF_INET, SOCK_STREAM, 0)) < 0) {
        printf("\n Socket creation error \n");
        return -1;
    }

    memset(&serv_addr_info, '0', sizeof(serv_addr_info));

    serv_addr_info.sin_family = AF_INET;
    serv_addr_info.sin_port = htons(PORT);

    // Convertng and settng server address
    if (inet_pton(AF_INET, "127.0.0.1", &serv_addr_info.sin_addr) <= 0) {
        printf("\nInvalid address/ Address not supported \n");
        return -1;
    }

    // Connectng to the server
    if (connect(server_fd_info, (struct sockaddr *)&serv_addr_info, sizeof(serv_addr_info)) < 0) {
        printf("\nConnection Failed \n");
        return -1;
    }

    printf("Connected to server.\n");
    printf("Enter a command (or 'quit' to exit):\n");

    while (1) {
        valid_syntx = 1;                                    // Resettng valid syntax flag for each iteratn
        memset(buf_val, 0, sizeof(buf_val));               // Clearng buffer
        memset(commandPassd, 0, sizeof(commandPassd));    // Clearng command response buffer
        memset(gf_ary, 0, sizeof(gf_ary));               // Clearng general-purpose buffer
        fgets(buf_val, sizeof(buf_val), stdin);         // Readng user input from console
        memcpy(gf_ary, buf_val, sizeof(buf_val));
       
        buf_val[strcspn(buf_val, "\n")] = 0;           // Removng newline charactr from input

     
        char* token_val = strtok(buf_val, " ");
        if (token_val == NULL) {
            valid_syntx = 0;
        } else if (strcmp(token_val, "filesrch") == 0) { 
                char* fil_name = strtok(NULL, " ");
                if (fil_name == NULL) {
                    valid_syntx = 0;
           
                    } else {
                sprintf(commandPassd, "filesrch %s", fil_name);
            }
        } 
 else if (strcmp(token_val, "tarfgetz") == 0) 
    { 
    char* size1_of_str = strtok(NULL, " ");   // Extractng lower size limit
    char* size2_of_str = strtok(NULL, " ");   // Extractng upper size limit
    char* unzip_flg_val = strtok(NULL, " ");  // Extractng optionl unzip flag

    if (size1_of_str == NULL || size2_of_str == NULL) 
    {
        sprintf(commandPassd, "Incorrect syntax. Please try once more.\n");
    } else 
    {
        int size1_taken = atoi(size1_of_str);    // Convertng lower size limit to an integr
        int size2_taken = atoi(size2_of_str);   // Convertng upper size limit to an integr
        if (size1_taken < 0 || size2_taken < 0 || size1_taken > size2_taken) {
            printf("Invalid size range. Please try again.\n");
            sprintf(commandPassd, "Invalid size range. Please try again.\n");
        } else 
        {
            char root_path_info[1024];
            sprintf(root_path_info, "/home/viutika");

            char commandPassd_buf[BUFSIZE];
            sprintf(commandPassd_buf, "find %s -type f -size +%dk -size -%dk -print0 | xargs -0 tar -czf temp.tar.gz",
                    root_path_info, size1_taken, size2_taken);

            char tar_command[BUFSIZE];
            snprintf(tar_command, sizeof(tar_command), "(cd %s && %s) > /dev/null 2>&1", root_path_info, commandPassd_buf);

            int status_recorded = system(tar_command);            

            if (unzip_flg_val != NULL && strcmp(unzip_flg_val, "-u") == 0) 
            {
                sprintf(commandPassd, "tarfgetz %s %s -u", size1_of_str, size2_of_str);
            } 
            else {
                sprintf(commandPassd, "tarfgetz %s %s", size1_of_str, size2_of_str);
            }
        }
    }
} else if (strcmp(token_val, "getdirf") == 0) { 
    char* date1_of_str = strtok(NULL, " ");     // Extractng start date
    char* date2_of_str = strtok(NULL, " ");    // Extractng end date
   

    if (date1_of_str == NULL || date2_of_str == NULL || !chk_valid_date(date1_of_str) || !chk_valid_date(date2_of_str)) {
        valid_syntx = 0;                       
    } else {
        int unzip_flg_val = 0;
        char* arg = strtok(NULL, " ");
        if (arg != NULL && strcmp(arg, "-u") == 0) {
            unzip_flg_val = 1;
        }       

        if (strcmp(date1_of_str, date2_of_str) > 0) { 
            printf("Date range is incorrect. %s is not before %s.\n", date1_of_str, date2_of_str);
            sprintf(commandPassd,"Date range is incorrect. %s is not before %s.\n", date1_of_str, date2_of_str); 
            printf("Enter a command (or 'quit' to exit):\n");
            valid_syntx = 0;
        } else if (valid_syntx) {            
            sprintf(commandPassd, "getdirf %s %s", date1_of_str, date2_of_str);
            if (unzip_flg_val) {
                sprintf(commandPassd + strlen(commandPassd), " -u");
            }
        } else {
            sprintf(commandPassd, "Incorrect syntax. Please try once more.\n");
        }
    }
}


 else if (strcmp(token_val, "fgets") == 0) {  
    gf_ary[strcspn(gf_ary, "\n")] = '\0';

    // Parsng file names and countng number of argumnts
    int num_args_cnt = 0;
    char temp_gf_ary[2048];
    strcpy(temp_gf_ary, gf_ary);
    char* arg_token_val = strtok(temp_gf_ary, " ");
    // Skip the command itself
    arg_token_val = strtok(NULL, " ");
    while (arg_token_val != NULL) {
        num_args_cnt++;
        arg_token_val = strtok(NULL, " ");
    }
    
    if (num_args_cnt == 0) {
        printf("Invalid syntax. Please specify at least one argument.\n");
        printf("Enter a command (or 'quit' to exit):\n");
        continue;
    } else if (num_args_cnt > 4) {
        printf("Invalid syntax. Only up to 4 arguments are allowed.\n");
        printf("Enter a command (or 'quit' to exit):\n");
        continue;
    }

    sprintf(commandPassd, gf_ary);
    printf("%s", commandPassd);
}
else if (strcmp(token_val, "targzf") == 0) {
    char* args_val[6];  // Storng argumnts (up to 4 extensions + -u)
    int num_args_cnt = 0;

    // Readng and storng argumnts
    while (num_args_cnt < 6) {  
        char* arg_val = strtok(NULL, " ");
        if (arg_val == NULL) {
            break;
        }
        args_val[num_args_cnt] = arg_val;
        num_args_cnt++;
    }

    if (num_args_cnt < 1) {
        printf("Invalid syntax. Please specify at least one extension.\n");
        printf("Enter a command (or 'quit' to exit):\n");
        valid_syntx = 0;
    } else if (num_args_cnt > 5) {  
        printf("Invalid syntax. Only up to 4 extensions are allowed.\n");
        printf("Enter a command (or 'quit' to exit):\n");
        valid_syntx = 0;
    } else {
        // Checkng if last argument is -u
        int unzip_flg_val = 0;
        if (num_args_cnt > 1 && strcmp(args_val[num_args_cnt - 1], "-u") == 0) {
            unzip_flg_val = 1;
            num_args_cnt--;
        }

        if (num_args_cnt > 4) {
            printf("Invalid syntax. Only up to 4 extensions are allowed.\n");
            printf("Enter a command (or 'quit' to exit):\n");
            valid_syntx = 0;
        } else {
            // Constructng command
            sprintf(commandPassd, "targzf");
            for (int i = 0; i < num_args_cnt; i++) {
                sprintf(commandPassd + strlen(commandPassd), " %s", args_val[i]);
            }
            if (unzip_flg_val) {
                sprintf(commandPassd + strlen(commandPassd), " -u");
            }
        }
    }
}
    
    else if (strcmp(token_val, "quit") == 0) {
        sprintf(commandPassd, "quit");
    } else {
        valid_syntx = 0;
    }

    if (!valid_syntx) {
        printf("Invalid syntax. Please try again.\n");
        continue;
    }

   // Sendng comand to server
    send(server_fd_info, commandPassd, strlen(commandPassd), 0);

    
    char rsponse[1024]={0};
    int valueread=read(server_fd_info, rsponse, sizeof(rsponse));   // Readng response from server
    printf("%s\n", rsponse);
    rsponse[strcspn(rsponse, "\n")] = '\0';
    if (strcmp(rsponse, "7001") == 0)
    {
        close(server_fd_info);   
        printf("Closing server connection and re-establishing mirror connection. Enter the command again.\n");
        
        // Creatng new socket for mirror connection
        server_fd_info = socket(AF_INET, SOCK_STREAM, 0);
        if (server_fd_info == -1) {
            perror("socket");
            exit(EXIT_FAILURE);
        }

        // Settng up mirror address information
        memset(&mirror_addr_info, '\0', sizeof(mirror_addr_info));
        mirror_addr_info.sin_family = AF_INET;
        mirror_addr_info.sin_port = htons(mirror_port);
        mirror_addr_info.sin_addr.s_addr = inet_addr("127.0.0.1");

        // Connectng to mirror using new socket
        if (connect(server_fd_info, (struct sockaddr *)&mirror_addr_info, sizeof(mirror_addr_info)) == -1) {
            perror("connect");
            exit(EXIT_FAILURE);
        }
     
    }
    else 
    printf("%s\n", rsponse);
    if (strcmp(commandPassd, "quit") == 0) {
        break;
    }

    printf("Enter a command (or 'quit' to exit):\n");
}

close(server_fd_info);
printf("Connection closed.\n");

return 0;
}