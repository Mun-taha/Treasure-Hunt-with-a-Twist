#include <stdio.h>
#include <stdlib.h>
#include <time.h>

#define GRID_SIZE 5
#define MAX_MOVES 20

typedef struct {
    int x;
    int y;
} Position;

char grid[GRID_SIZE][GRID_SIZE];
Position player = {0, 0};
int score = 0;
int moves = MAX_MOVES;
int health = 3;
int difficulty;
int boardSize, trapCount, boostCount;
int playerX, playerY, treasureX, treasureY;
char playerName[30];

void initializeGrid() {
    srand(time(NULL));

    printf("Welcome to the Ultimate Treasure Hunt Game!\n");
    printf("Pick your level of madness:\n");
    printf("1. Easy (5x5 grid, few traps)\n");
    printf("2. Medium (7x7 grid, more traps)\n");
    printf("3. Hard (10x10 grid, max traps)\n");
    scanf("%d", &difficulty);

    if (difficulty == 1) {
        boardSize = 5;
        trapCount = 5;
        boostCount = 2;
    } else if (difficulty == 2) {
        boardSize = 7;
        trapCount = 10;
        boostCount = 3;
    } else {
        boardSize = 10;
        trapCount = 20;
        boostCount = 5;
    }

    printf("Enter your name, brave adventurer: ");
    scanf("%s", playerName);

    for (int i = 0; i < GRID_SIZE; i++) {
        for (int j = 0; j < GRID_SIZE; j++) {
            grid[i][j] = '*';
        }
    }

    for (int i = 0; i < trapCount + boostCount; i++) {
        int x = rand() % GRID_SIZE;
        int y = rand() % GRID_SIZE;
        char items[] = {'T', 'X', '+'};
        grid[x][y] = items[rand() % 3];
    }
}

void displayGrid() {
    for (int i = 0; i < GRID_SIZE; i++) {
        for (int j = 0; j < GRID_SIZE; j++) {
            if (i == player.x && j == player.y) {
                printf("P ");
            } else {
                printf("%c ", grid[i][j]);
            }
        }
        printf("\n");
    }
}

void updatePosition(char move) {
    int oldX = player.x;
    int oldY = player.y;

    if (move == 'w' && player.x > 0) player.x--;
    if (move == 's' && player.x < GRID_SIZE - 1) player.x++;
    if (move == 'a' && player.y > 0) player.y--;
    if (move == 'd' && player.y < GRID_SIZE - 1) player.y++;

    char cell = grid[player.x][player.y];
    if (cell == 'T') {
        score += 10;
    } else if (cell == 'X') {
        health--;
        if (health <= 0) {
            printf("Game Over! You hit a trap!\n");
            exit(0);
        }
    } else if (cell == '+') {
        moves += 5;
    }

    grid[oldX][oldY] = '*';
    moves--;
    if (moves <= 0) {
        printf("Game Over! You ran out of moves!\n");
        exit(0);
    }
}

void giveHint() {
    int closestDistance = GRID_SIZE * GRID_SIZE;
    int hintX = -1, hintY = -1;

    for (int i = 0; i < GRID_SIZE; i++) {
        for (int j = 0; j < GRID_SIZE; j++) {
            if (grid[i][j] == 'T' || grid[i][j] == '+') {
                int distance = abs(player.x - i) + abs(player.y - j);
                if (distance < closestDistance) {
                    closestDistance = distance;
                    hintX = i;
                    hintY = j;
                }
            }
        }
    }

    if (hintX != -1 && hintY != -1) {
        printf("Hint: Move closer to (%d, %d) to find a treasure or boost.\n", hintX, hintY);
    } else {
        printf("No treasures or boosts left on the grid.\n");
    }
}

int main() {
    initializeGrid();
    while (1) {
        displayGrid();
        printf("Score: %d, Health: %d, Moves left: %d\n", score, health, moves);
        printf("Enter move (w/a/s/d) or 'h' for a hint: ");
        char move;
        scanf(" %c", &move);

        if (move == 'h') {
            giveHint();
        } else {
            updatePosition(move);
        }

        if (score >= 50) {
            printf("You win! You collected enough treasures!\n");
            break;
        }
    }
    return 0;
}
