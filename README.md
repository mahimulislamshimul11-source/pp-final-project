#include <stdio.h>
#include <string.h>
#include <stdlib.h>

// --- (5p) Colors Configuration ---
#define RESET   "\033[0m"
#define RED     "\033[1;31m"
#define GREEN   "\033[1;32m"
#define YELLOW  "\033[1;33m"
#define CYAN    "\033[1;36m"
#define BOLD    "\033[1m"

#define MAX_PLAYERS 50
#define MAX_NAME 30
#define MAX_HISTORY 20
#define FILE_NAME "matches.txt"

typedef struct {
    char name[MAX_NAME];
    int wins;
    int losses;
    char history[MAX_HISTORY][100]; 
    int match_count;
} Player;

Player players[MAX_PLAYERS];
int player_count = 0;

// --- (5p) Functions: File Handling ---

void save_to_file() {
    FILE *f = fopen(FILE_NAME, "w");
    if (f == NULL) return;

    fprintf(f, "%d\n", player_count);
    for (int i = 0; i < player_count; i++) {
        fprintf(f, "%s %d %d %d\n", players[i].name, players[i].wins, players[i].losses, players[i].match_count);
        for (int j = 0; j < players[i].match_count; j++) {
            fprintf(f, "%s\n", players[i].history[j]);
        }
    }
    fclose(f);
}

void load_from_file() {
    FILE *f = fopen(FILE_NAME, "r");
    if (f == NULL) return; 

    if (fscanf(f, "%d", &player_count) != 1) {
        fclose(f);
        return;
    }

    for (int i = 0; i < player_count; i++) {
        fscanf(f, "%s %d %d %d", players[i].name, &players[i].wins, &players[i].losses, &players[i].match_count);
        fgetc(f); 
        for (int j = 0; j < players[i].match_count; j++) {
            fgets(players[i].history[j], 100, f);
            players[i].history[j][strcspn(players[i].history[j], "\n")] = 0; 
        }
    }
    fclose(f);
}

int get_or_create_player(char *name) {
    for (int i = 0; i < player_count; i++) {
        if (strcmp(players[i].name, name) == 0) return i;
    }
    if (player_count < MAX_PLAYERS) {
        strcpy(players[player_count].name, name);
        players[player_count].wins = 0;
        players[player_count].losses = 0;
        players[player_count].match_count = 0;
        return player_count++;
    }
    return -1;
}

// --- Logic Update: 21 Point Validation ---
void add_match() {
    char name1[MAX_NAME], name2[MAX_NAME];
    int score1, score2;

    printf("\n--- " YELLOW "Match Entry (Max 21 Points)" RESET " ---");
    printf("\nFormat: [Name1] [Score1] [Score2] [Name2]");
    printf("\nInput: ");
    
    // (5p) Validation for Format
    if (scanf("%s %d %d %s", name1, &score1, &score2, name2) != 4) {
        printf(RED "Invalid input format!\n" RESET);
        while(getchar() != '\n'); 
        return;
    }

    // (5p) Validation for Table Tennis Rules (0-21)
    if (score1 > 21 || score2 > 21 || score1 < 0 || score2 < 0) {
        printf(RED "Error: Scores must be between 0 and 21!\n" RESET);
        return;
    }
    if (score1 == score2) {
        printf(RED "Error: A match cannot end in a tie!\n" RESET);
        return;
    }

    int p1_idx = get_or_create_player(name1);
    int p2_idx = get_or_create_player(name2);

    if (score1 > score2) {
        players[p1_idx].wins++;
        players[p2_idx].losses++;
    } else {
        players[p2_idx].wins++;
        players[p1_idx].losses++;
    }

    char match_info[100];
    sprintf(match_info, "%s %d / %d %s", name1, score1, score2, name2);
    
    if (players[p1_idx].match_count < MAX_HISTORY)
        strcpy(players[p1_idx].history[players[p1_idx].match_count++], match_info);
    if (players[p2_idx].match_count < MAX_HISTORY)
        strcpy(players[p2_idx].history[players[p2_idx].match_count++], match_info);

    save_to_file(); 
    printf(GREEN "Result saved to matches.txt!\n" RESET);
}

void view_rankings() {
    printf("\n--- " YELLOW "Rankings" RESET " ---\n");
    // Sort logic (Bubble Sort)
    for (int i = 0; i < player_count - 1; i++) {
        for (int j = 0; j < player_count - i - 1; j++) {
            if (players[j].wins < players[j+1].wins) {
                Player temp = players[j];
                players[j] = players[j+1];
                players[j+1] = temp;
            }
        }
    }
    printf("%-15s | %-5s | %-5s\n", "Player", "Wins", "Losses");
    for (int i = 0; i < player_count; i++) {
        printf(CYAN "%-15s" RESET " | %-5d | %-5d\n", players[i].name, players[i].wins, players[i].losses);
    }
}

void view_player_history() {
    char search_name[MAX_NAME];
    printf("\nEnter player name: ");
    scanf("%s", search_name);

    for (int i = 0; i < player_count; i++) {
        if (strcmp(players[i].name, search_name) == 0) {
            printf("\nHistory for " CYAN "%s" RESET ":\n", players[i].name);
            for (int j = 0; j < players[i].match_count; j++) {
                printf("  • %s\n", players[i].history[j]);
            }
            return;
        }
    }
    printf(RED "Player not found.\n" RESET);
}

int main() {
    load_from_file(); 
    
    int choice;
    while (1) {
        printf("\n" BOLD "TABLE TENNIS MANAGER" RESET);
        printf("\n1. Add Match | 2. Rankings | 3. History | 4. Exit\nChoice: ");
        if (scanf("%d", &choice) != 1) {
            while(getchar() != '\n');
            continue;
        }

        switch (choice) {
            case 1: add_match(); break;
            case 2: view_rankings(); break;
            case 3: view_player_history(); break;
            case 4: save_to_file(); return 0;
            default: printf("Try again.\n");
        }
    }
    return 0;
}
