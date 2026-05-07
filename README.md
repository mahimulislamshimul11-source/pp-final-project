#include <stdio.h>
#include <string.h>
#include <stdlib.h>

// --- (5p) Colors Configuration ---
// Using ANSI Escape Codes for selective coloring
#define RESET   "\033[0m"
#define RED     "\033[1;31m"
#define GREEN   "\033[1;32m"
#define YELLOW  "\033[1;33m"
#define CYAN    "\033[1;36m"
#define BOLD    "\033[1m"

#define MAX_PLAYERS 50
#define MAX_NAME 30
#define MAX_HISTORY 20

// --- (5p) Structs Implementation ---
typedef struct {
    char name[MAX_NAME];
    int wins;
    int losses;
    char history[MAX_HISTORY][100]; 
    int match_count;
} Player;

Player players[MAX_PLAYERS];
int player_count = 0;

// --- (5p) Functions: Logic Extraction ---

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

void add_match() {
    char name1[MAX_NAME], name2[MAX_NAME];
    int score1, score2;

    printf("\n--- " YELLOW "Add Match Entry" RESET " ---");
    
    // (5p) Validations: Explicit format and input checking
    printf("\nEnter " BOLD "Player 1 Name" RESET " (single word): ");
    if (scanf("%s", name1) != 1) return;

    printf("Enter " CYAN "%s's" RESET " score (integer 0-30): ", name1);
    while (scanf("%d", &score1) != 1 || score1 < 0) {
        printf(RED "Error:" RESET " Please enter a valid positive integer for score: ");
        while(getchar() != '\n'); // clear buffer
    }

    printf("Enter " BOLD "Player 2 Name" RESET " (single word): ");
    if (scanf("%s", name2) != 1) return;

    printf("Enter " CYAN "%s's" RESET " score (integer 0-30): ", name2);
    while (scanf("%d", &score2) != 1 || score2 < 0) {
        printf(RED "Error:" RESET " Please enter a valid positive integer for score: ");
        while(getchar() != '\n'); 
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

    printf("\n" GREEN "✔" RESET " Match between " CYAN "%s" RESET " and " CYAN "%s" RESET " recorded!\n", name1, name2);
}

void view_rankings() {
    printf("\n--- " YELLOW "Global Rankings" RESET " ---\n");
    if (player_count == 0) {
        printf("No records found.\n");
        return;
    }

    // Sort players by wins
    for (int i = 0; i < player_count - 1; i++) {
        for (int j = 0; j < player_count - i - 1; j++) {
            if (players[j].wins < players[j+1].wins) {
                Player temp = players[j];
                players[j] = players[j+1];
                players[j+1] = temp;
            }
        }
    }

    printf("%-15s | %-5s | %-5s\n", "Player Name", "Wins", "Losses");
    printf("----------------------------------\n");
    for (int i = 0; i < player_count; i++) {
        // (5p) Colors: Only coloring specific elements (the name)
        printf(CYAN "%-15s" RESET " | %-5d | %-5d\n", players[i].name, players[i].wins, players[i].losses);
    }
}

void view_player_history() {
    char search_name[MAX_NAME];
    printf("\nEnter " BOLD "Player Name" RESET " to search: ");
    scanf("%s", search_name);

    for (int i = 0; i < player_count; i++) {
        if (strcmp(players[i].name, search_name) == 0) {
            printf("\nHistory for " CYAN "%s" RESET ":\n", players[i].name);
            printf(GREEN "Wins: %d" RESET " | " RED "Losses: %d" RESET "\n", players[i].wins, players[i].losses);
            for (int j = 0; j < players[i].match_count; j++) {
                printf("  • %s\n", players[i].history[j]);
            }
            return;
        }
    }
    printf(RED "✖ Player '%s' not found." RESET "\n", search_name);
}

int main() {
    int choice;
    while (1) {
        printf("\n" BOLD "=== TABLE TENNIS MGMT SYSTEM ===" RESET "\n");
        printf("1. " YELLOW "Add" RESET " Match Result\n");
        printf("2. " YELLOW "View" RESET " Rankings\n");
        printf("3. " YELLOW "Search" RESET " Player History\n");
        printf("4. " RED "Exit" RESET "\n");
        printf("Select Option (1-4): ");

        if (scanf("%d", &choice) != 1) {
            printf(RED "Invalid input!" RESET " Use numbers 1-4.\n");
            while(getchar() != '\n');
            continue;
        }

        switch (choice) {
            case 1: add_match(); break;
            case 2: view_rankings(); break;
            case 3: view_player_history(); break;
            case 4: printf("Exiting...\n"); return 0;
            default: printf(RED "Invalid choice." RESET "\n");
        }
    }
    return 0;
}
