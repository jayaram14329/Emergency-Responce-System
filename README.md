# Emergency-Responce-System



#include <stdio.h>
#include <stdlib.h>
#include <limits.h>
#include <time.h>
#include <string.h>

#define V 10
#define MAX 100

#define RESET   "\033[0m"
#define RED     "\033[31m"
#define GREEN   "\033[32m"
#define YELLOW  "\033[33m"
#define CYAN    "\033[36m"

// Location names
char *locationNames[V] = {
    "Hospital",
    "Fire Station",
    "Police Station",
    "Junction A",
    "Junction B",
    "Market Area",
    "School Zone",
    "Residential Area",
    "Industrial Zone",
    "Accident Site Area"
};

// Emergency structure
typedef struct {
    char type[50];
    int location;
    int priority;
} Emergency;

// Priority Queue
Emergency pq[MAX];
int pqSize = 0;

// Log file pointer
FILE *logFile;

// ---------- Utility: Get Current Time ----------
void logTime() {
    time_t now = time(NULL);
    struct tm *t = localtime(&now);
    fprintf(logFile, "[%02d-%02d-%04d %02d:%02d:%02d]\n",
            t->tm_mday, t->tm_mon + 1, t->tm_year + 1900,
            t->tm_hour, t->tm_min, t->tm_sec);
}

// ---------- Show Locations ----------
void showLocations() {
    printf("\nAvailable Locations:\n");
    for (int i = 0; i < V; i++) {
        printf("%d - %s\n", i, locationNames[i]);
    }
}

// ---------- Safe Integer Input ----------
int readIntInRange(const char *msg, int min, int max) {
    int num;
    char c;
    while (1) {
        printf("%s", msg);
        if (scanf("%d", &num) == 1 && num >= min && num <= max) {
            return num;
        }
        while ((c = getchar()) != '\n' && c != EOF) {}
        printf("Invalid input! Enter number between %d and %d.\n", min, max);
    }
}

// ---------- Priority Queue ----------
void push(Emergency e) {
    pq[pqSize] = e;
    pqSize++;

    for(int i = 0; i < pqSize; i++) {
        for(int j = i+1; j < pqSize; j++) {
            if(pq[j].priority < pq[i].priority) {
                Emergency temp = pq[i];
                pq[i] = pq[j];
                pq[j] = temp;
            }
        }
    }
}

Emergency pop() {
    Emergency top = pq[0];
    for(int i = 1; i < pqSize; i++) {
        pq[i-1] = pq[i];
    }
    pqSize--;
    return top;
}

// ---------- Dijkstra Helpers ----------
int minDistance(int dist[], int visited[]) {
    int min = INT_MAX, index = -1;
    for(int i = 0; i < V; i++) {
        if(!visited[i] && dist[i] <= min) {
            min = dist[i];
            index = i;
        }
    }
    return index;
}

void printPath(int parent[], int j) {
    if (parent[j] == -1) {
        printf("%s", locationNames[j]);
        fprintf(logFile, "%s", locationNames[j]);
        return;
    }
    printPath(parent, parent[j]);
    printf(" -> %s", locationNames[j]);
    fprintf(logFile, " -> %s", locationNames[j]);
}

// ---------- Dijkstra Algorithm ----------
void dijkstra(int graph[V][V], int src, int dest) {
    int dist[V], visited[V], parent[V];

    for(int i = 0; i < V; i++) {
        dist[i] = INT_MAX;
        visited[i] = 0;
        parent[i] = -1;
    }

    dist[src] = 0;

    for(int count = 0; count < V - 1; count++) {
        int u = minDistance(dist, visited);
        if (u == -1) break;

        visited[u] = 1;

        for(int v = 0; v < V; v++) {
            if(!visited[v] && graph[u][v] &&
                dist[u] != INT_MAX &&
                dist[u] + graph[u][v] < dist[v]) {

                parent[v] = u;
                dist[v] = dist[u] + graph[u][v];
            }
        }
    }

    printf("\n\n%sFastest Route:%s ", GREEN, RESET);
    fprintf(logFile, "Fastest Route: ");

    printPath(parent, dest);

    printf("\n%sEstimated Travel Time:%s %d minutes\n", CYAN, RESET, dist[dest]);
    fprintf(logFile, "\nEstimated Travel Time: %d minutes\n\n", dist[dest]);
}

// ---------- Traffic Simulation ----------
void simulateTraffic(int graph[V][V]) {
    printf("\n%s[Traffic Simulation Active]%s\n", YELLOW, RESET);
    fprintf(logFile, "Traffic Updated:\n");

    for(int i = 0; i < V; i++) {
        for(int j = i+1; j < V; j++) {
            if(graph[i][j] != 0) {
                int change = (rand() % 7) - 3;
                graph[i][j] += change;
                if(graph[i][j] < 1) graph[i][j] = 1;
                graph[j][i] = graph[i][j];

                fprintf(logFile, "%s <-> %s = %d mins\n",
                        locationNames[i], locationNames[j], graph[i][j]);
            }
        }
    }
    fprintf(logFile, "\n");
}

// ---------- Main Program ----------
int main() {
    srand(time(NULL));

    logFile = fopen("emergency_log.txt", "a");

    if (!logFile) {
        printf("Error opening log file!\n");
        return 1;
    }

    int graph[V][V] = {
        {0,5,2,0,0,0,0,0,0,0},
        {5,0,3,6,0,0,0,0,0,0},
        {2,3,0,4,7,0,0,0,0,0},
        {0,6,4,0,2,3,0,0,0,0},
        {0,0,7,2,0,5,4,0,0,0},
        {0,0,0,3,5,0,6,2,0,0},
        {0,0,0,0,4,6,0,5,3,0},
        {0,0,0,0,0,2,5,0,4,6},
        {0,0,0,0,0,0,3,4,0,5},
        {0,0,0,0,0,0,0,6,5,0}
    };

    printf("%s=== Emergency Response System ===%s\n", CYAN, RESET);

    int emergencyCount = readIntInRange("Enter number of emergencies: ", 1, 20);

    for(int i = 0; i < emergencyCount; i++) {
        Emergency e;

        printf("\nSelect Emergency Type:\n");
        printf("1 - Fire\n");
        printf("2 - Accident\n");
        printf("3 - Crime\n");

        int t = readIntInRange("Enter choice: ", 1, 3);

        switch(t) {
            case 1: strcpy(e.type, "Fire"); break;
            case 2: strcpy(e.type, "Accident"); break;
            case 3: strcpy(e.type, "Crime"); break;
        }

        showLocations();
        e.location = readIntInRange("Enter emergency location (0-9): ", 0, 9);
        e.priority = readIntInRange("Enter priority (1=High, 5=Low): ", 1, 5);

        push(e);
    }

    printf("\n%sProcessing emergencies by priority...%s\n", YELLOW, RESET);

    while(pqSize > 0) {
        Emergency current = pop();

        printf("\n\n%sHandling Emergency: %s at %s (Priority %d)%s\n",
            RED, current.type, locationNames[current.location],
            current.priority, RESET);

        logTime();
        fprintf(logFile, "Emergency Type: %s\n", current.type);
        fprintf(logFile, "Location: %s\n", locationNames[current.location]);
        fprintf(logFile, "Priority: %d\n", current.priority);

        simulateTraffic(graph);

        int vehicle;

        if (strcmp(current.type, "Fire") == 0)
            vehicle = 1;
        else if (strcmp(current.type, "Crime") == 0)
            vehicle = 2;
        else
            vehicle = 0;

        printf("Dispatch Vehicle Starting From: %s\n", locationNames[vehicle]);
        fprintf(logFile, "Dispatch Vehicle From: %s\n", locationNames[vehicle]);

        dijkstra(graph, vehicle, current.location);
    }

    fclose(logFile);

    return 0;
}
