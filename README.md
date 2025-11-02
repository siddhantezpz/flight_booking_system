#include <stdio.h>
#include <stdlib.h>
#include <string.h>

// ------------------- Structures -------------------

typedef struct Flight {
    int flightNo;
    char destination[50];
    int seatsAvailable;
    struct Flight *next;
} Flight;

typedef struct Passenger {
    int ticketNo;
    char name[50];
    int flightNo;
    struct Passenger *left, *right;
} Passenger;

typedef struct Booking {
    char name[50];
    int flightNo;
    struct Booking *next;
} Booking;

// ------------------- Linked List for Flights -------------------

Flight *flightHead = NULL;
Passenger *passengerRoot = NULL;
Booking *front = NULL, *rear = NULL;

// ------------------- Helper Functions -------------------

Flight* createFlight(int no, char *dest, int seats) {
    Flight *newFlight = (Flight*)malloc(sizeof(Flight));
    newFlight->flightNo = no;
    strcpy(newFlight->destination, dest);
    newFlight->seatsAvailable = seats;
    newFlight->next = NULL;
    return newFlight;
}

void addFlight() {
    int no, seats;
    char dest[50];
    printf("\nEnter Flight Number: ");
    scanf("%d", &no);
    printf("Enter Destination: ");
    scanf("%s", dest);
    printf("Enter Seats Available: ");
    scanf("%d", &seats);

    Flight *newFlight = createFlight(no, dest, seats);
    newFlight->next = flightHead;
    flightHead = newFlight;

    printf("\n✅ Flight Added Successfully!\n");
}

void displayFlights() {
    if (flightHead == NULL) {
        printf("\n⚠️ No Flights Available.\n");
        return;
    }

    Flight *temp = flightHead;
    printf("\n✈️  Available Flights:\n");
    printf("------------------------------------------\n");
    printf("Flight No\tDestination\tSeats\n");
    while (temp != NULL) {
        printf("%d\t\t%s\t\t%d\n", temp->flightNo, temp->destination, temp->seatsAvailable);
        temp = temp->next;
    }
    printf("------------------------------------------\n");
}

// ------------------- Queue for Booking -------------------

void enqueue(char *name, int flightNo) {
    Booking *newBooking = (Booking*)malloc(sizeof(Booking));
    strcpy(newBooking->name, name);
    newBooking->flightNo = flightNo;
    newBooking->next = NULL;

    if (rear == NULL) {
        front = rear = newBooking;
    } else {
        rear->next = newBooking;
        rear = newBooking;
    }

    printf("\n🧾 Booking Request Added to Queue!\n");
}

Booking* dequeue() {
    if (front == NULL) {
        printf("\n⚠️ No pending bookings.\n");
        return NULL;
    }

    Booking *temp = front;
    front = front->next;
    if (front == NULL)
        rear = NULL;
    return temp;
}

// ------------------- BST for Passengers -------------------

Passenger* createPassengerNode(int ticketNo, char *name, int flightNo) {
    Passenger *newP = (Passenger*)malloc(sizeof(Passenger));
    newP->ticketNo = ticketNo;
    strcpy(newP->name, name);
    newP->flightNo = flightNo;
    newP->left = newP->right = NULL;
    return newP;
}

Passenger* insertPassenger(Passenger *root, int ticketNo, char *name, int flightNo) {
    if (root == NULL)
        return createPassengerNode(ticketNo, name, flightNo);
    if (ticketNo < root->ticketNo)
        root->left = insertPassenger(root->left, ticketNo, name, flightNo);
    else
        root->right = insertPassenger(root->right, ticketNo, name, flightNo);
    return root;
}

Passenger* searchPassenger(Passenger *root, int ticketNo) {
    if (root == NULL || root->ticketNo == ticketNo)
        return root;
    if (ticketNo < root->ticketNo)
        return searchPassenger(root->left, ticketNo);
    else
        return searchPassenger(root->right, ticketNo);
}

// ------------------- Booking Process -------------------

void processBooking() {
    Booking *b = dequeue();
    if (b == NULL) return;

    Flight *temp = flightHead;
    while (temp != NULL && temp->flightNo != b->flightNo)
        temp = temp->next;

    if (temp == NULL) {
        printf("\n❌ Flight not found!\n");
        free(b);
        return;
    }

    if (temp->seatsAvailable > 0) {
        temp->seatsAvailable--;
        int ticketNo = rand() % 10000 + 1000; // random ticket no
        passengerRoot = insertPassenger(passengerRoot, ticketNo, b->name, b->flightNo);
        printf("\n🎟️ Booking Confirmed for %s | Flight %d | Ticket #%d\n", b->name, b->flightNo, ticketNo);
    } else {
        printf("\n🚫 No seats available on this flight!\n");
    }

    free(b);
}

void searchPassengerByTicket() {
    int t;
    printf("\nEnter Ticket Number: ");
    scanf("%d", &t);
    Passenger *p = searchPassenger(passengerRoot, t);
    if (p)
        printf("\n✅ Passenger Found: %s | Flight: %d | Ticket: %d\n", p->name, p->flightNo, p->ticketNo);
    else
        printf("\n⚠️ Passenger Not Found.\n");
}

// ------------------- Main Menu -------------------

void menu() {
    printf("\n================= 🛫 Flight Booking System 🛬 =================\n");
    printf("1. Add Flight\n");
    printf("2. Display Flights\n");
    printf("3. Add Booking Request\n");
    printf("4. Process Booking\n");
    printf("5. Search Passenger by Ticket No\n");
    printf("6. Exit\n");
    printf("===============================================================\n");
}

int main() {
    int choice;
    char name[50];
    int flightNo;

    while (1) {
        menu();
        printf("Enter your choice: ");
        scanf("%d", &choice);

        switch (choice) {
            case 1:
                addFlight();
                break;
            case 2:
                displayFlights();
                break;
            case 3:
                printf("\nEnter Passenger Name: ");
                scanf("%s", name);
                printf("Enter Flight No: ");
                scanf("%d", &flightNo);
                enqueue(name, flightNo);
                break;
            case 4:
                processBooking();
                break;
            case 5:
                searchPassengerByTicket();
                break;
            case 6:
                printf("\n👋 Thank you for using Flight Booking System!\n");
                exit(0);
            default:
                printf("\n⚠️ Invalid choice, try again!\n");
        }
    }
    return 0;
}
