#include <stdio.h>
#include <stdlib.h>
#include <string.h>

// Requirements: Colors
#define RESET   "\033[0m"
#define RED     "\033[31m"
#define GREEN   "\033[32m"
#define CYAN    "\033[36m"

typedef struct {
    char name[50];
    int wins;
} Player;

// THIS IS THE CRITICAL LOGIC FIX
void viewRankings(const char *filename) {
    FILE *f = fopen(filename, "r");
    if (!f) return;

    Player list[100];
    int pCount = 0;
    char p1[50], p2[50];
    int s1, s2;

    while (fscanf(f, "%s %d %s %d", p1, &s1, p2, &s2) == 4) {
        char winner[50] = "";

        // STRICT CHECK: Only a score of EXACTLY 21 counts as a win
        if (s1 == 21 && s2 < 21) {
            strcpy(winner, p1);
        } else if (s2 == 21 && s1 < 21) {
            strcpy(winner, p2);
        } else {
            // If the score is 23, 25, or 19, this match is SKIPPED
            continue; 
        }

        // Update the winner in the ranking list
        int found = -1;
        for(int i=0; i < pCount; i++) {
            if(strcmp(list[i].name, winner) == 0) { found = i; break; }
        }

        if(found != -1) {
            list[found].wins++;
        } else {
            strcpy(list[pCount].name, winner);
            list[pCount].wins = 1;
            pCount++;
        }
    }
    fclose(f);

    // Sort and Display
    printf("\n" CYAN "--- Global Rankings (Strict 21-Point Rule) ---" RESET "\n");
    for (int i = 0; i < pCount; i++) {
        printf("%s: %d Wins\n", list[i].name, list[i].wins);
    }
}

// VALIDATION: Prevent entering bad scores
void addMatch(const char *filename) {
    char p1[50], p2[50];
    int s1, s2;

    printf("Enter P1 Name, Score, P2 Name, Score: ");
    scanf("%s %d %s %d", p1, &s1, p2, &s2);

    // If neither player has exactly 21, we REJECT the entry
    if ((s1 == 21 && s2 < 21) || (s2 == 21 && s1 < 21)) {
        FILE *f = fopen(filename, "a");
        fprintf(f, "%s %d %s %d\n", p1, s1, p2, s2);
        fclose(f);
        printf(GREEN "Match saved." RESET "\n");
    } else {
        printf(RED "REJECTED: A match must end at exactly 21 points. %d or %d is invalid." RESET "\n", s1, s2);
    }
}
