# projects-
#include <stdio.h>
#include <string.h>
#define MAX_ORDERS 25
#define MAX_ITEMS 6
#define ITEM_LEN 50
typedef struct {
    int orderId;
    int itemCount;
    char items[MAX_ITEMS][ITEM_LEN];
    int estimatedTime;
} Order;

typedef struct {
    Order data[MAX_ORDERS];
    int front;
    int rear;
    int count;
} Queue;
Queue vipQueue, normalQueue;

int nextOrderId = 1001;
int totalServed = 0;
int timePerItem = 5;   
void clearBuffer() {
    int c;
    while ((c = getchar()) != '\n' && c != EOF);
}

void pauseScreen() {
    printf("\nPress Enter to continue...");
    getchar();
}
void initQueue(Queue *q) {
    q->front = 0;
    q->rear = -1;
    q->count = 0;
}

int isEmpty(Queue *q) {
    return q->count == 0;
}

int isFull(Queue *q) {
    return q->count == MAX_ORDERS;
}

void enqueue(Queue *q, Order o) {
    if (isFull(q)) {
        printf("\nQueue FULL. Cannot add order.\n");
        return;
    }
    q->rear++;
    q->data[q->rear] = o;
    q->count++;
}

Order dequeue(Queue *q) {
    Order empty;
    empty.orderId = -1;

    if (isEmpty(q))
        return empty;

    Order o = q->data[q->front];
    q->front++;
    q->count--;
    return o;
}


void removeAtIndex(Queue *q, int index) {
    int i;
    for (i = index; i < q->rear; i++) {
        q->data[i] = q->data[i + 1];
    }
    q->rear--;
    q->count--;
}



Order createOrder() {
    Order o;
    int i;

    o.orderId = nextOrderId++;

    printf("\nEnter number of items (max %d): ", MAX_ITEMS);
    scanf("%d", &o.itemCount);
    clearBuffer();

    if (o.itemCount < 1) o.itemCount = 1;
    if (o.itemCount > MAX_ITEMS) o.itemCount = MAX_ITEMS;

    for (i = 0; i < o.itemCount; i++) {
        printf("Enter item %d name:\n", i + 1);
        fgets(o.items[i], ITEM_LEN, stdin);
        o.items[i][strcspn(o.items[i], "\n")] = '\0';
    }

    o.estimatedTime = o.itemCount * timePerItem;
    return o;
}



void addOrder() {
    int type;
    Order o;

    printf("\n1. VIP Order\n2. Normal Order\nChoose type: ");
    scanf("%d", &type);
    clearBuffer();

    o = createOrder();

    if (type == 1)
        enqueue(&vipQueue, o);
    else
        enqueue(&normalQueue, o);

    printf("\nOrder %d added successfully.", o.orderId);
    printf("\nEstimated preparation time: %d minutes\n", o.estimatedTime);
    pauseScreen();
}

void serveOrder() {
    Order o;
    int i;

    if (!isEmpty(&vipQueue))
        o = dequeue(&vipQueue);
    else if (!isEmpty(&normalQueue))
        o = dequeue(&normalQueue);
    else {
        printf("\nNo orders to serve.\n");
        pauseScreen();
        return;
    }

    printf("\nServing Order ID %d\n", o.orderId);
    printf("Items:\n");
    for (i = 0; i < o.itemCount; i++)
        printf("- %s\n", o.items[i]);

    totalServed++;
    pauseScreen();
}

void searchOrder() {
    int id, i;
    printf("\nEnter Order ID to search: ");
    scanf("%d", &id);
    clearBuffer();

    for (i = vipQueue.front; i <= vipQueue.rear; i++) {
        if (vipQueue.data[i].orderId == id) {
            printf("\nOrder found in VIP Queue (Position %d)\n",
                   i - vipQueue.front + 1);
            pauseScreen();
            return;
        }
    }

    for (i = normalQueue.front; i <= normalQueue.rear; i++) {
        if (normalQueue.data[i].orderId == id) {
            printf("\nOrder found in Normal Queue (Position %d)\n",
                   i - normalQueue.front + 1);
            pauseScreen();
            return;
        }
    }

    printf("\nOrder not found.\n");
    pauseScreen();
}


void cancelOrder() {
    int id, i;

    printf("\nEnter Order ID to cancel: ");
    scanf("%d", &id);
    clearBuffer();

    for (i = vipQueue.front; i <= vipQueue.rear; i++) {
        if (vipQueue.data[i].orderId == id) {
            removeAtIndex(&vipQueue, i);
            printf("\nOrder %d cancelled from VIP queue.\n", id);
            pauseScreen();
            return;
        }
    }

    for (i = normalQueue.front; i <= normalQueue.rear; i++) {
        if (normalQueue.data[i].orderId == id) {
            removeAtIndex(&normalQueue, i);
            printf("\nOrder %d cancelled from Normal queue.\n", id);
            pauseScreen();
            return;
        }
    }

    printf("\nOrder not found.\n");
    pauseScreen();
}



void displayQueue(Queue *q, const char *name) {
    int i, j, cumulative = 0;

    if (isEmpty(q)) {
        printf("\n%s Queue is empty.\n", name);
        return;
    }

    printf("\n%s Orders:\n", name);
    for (i = q->front; i <= q->rear; i++) {
        cumulative += q->data[i].estimatedTime;
        printf("Order ID %d | ETA %d mins | Cumulative ETA %d mins\n",
               q->data[i].orderId,
               q->data[i].estimatedTime,
               cumulative);

        printf("Items: ");
        for (j = 0; j < q->data[i].itemCount; j++)
            printf("%s ", q->data[i].items[j]);
        printf("\n");
    }
}

void displayAll() {
    displayQueue(&vipQueue, "VIP");
    displayQueue(&normalQueue, "Normal");
    pauseScreen();
}



void changeTimePerItem() {
    printf("\nCurrent time per item: %d minutes", timePerItem);
    printf("\nEnter new time per item: ");
    scanf("%d", &timePerItem);
    clearBuffer();
    printf("\nTime per item updated.\n");
    pauseScreen();
}



void showStats() {
    printf("\n===== STATISTICS =====\n");
    printf("Orders Served: %d\n", totalServed);
    printf("VIP Pending: %d\n", vipQueue.count);
    printf("Normal Pending: %d\n", normalQueue.count);
    printf("Total Pending: %d\n",
           vipQueue.count + normalQueue.count);
    pauseScreen();
}



void menu() {
    printf("\n===== RESTAURANT SYSTEM =====\n");
    printf("1. Add Order\n");
    printf("2. Serve Order\n");
    printf("3. Display Orders\n");
    printf("4. Search Order\n");
    printf("5. Cancel Order\n");
    printf("6. Change Time per Item\n");
    printf("7. Show Statistics\n");
    printf("8. Exit\n");
    printf("Enter choice: ");
}



int main() {
    int choice;

    initQueue(&vipQueue);
    initQueue(&normalQueue);

    while (1) {
        menu();
        scanf("%d", &choice);
        clearBuffer();

        switch (choice) {
            case 1: addOrder(); break;
            case 2: serveOrder(); break;
            case 3: displayAll(); break;
            case 4: searchOrder(); break;
            case 5: cancelOrder(); break;
            case 6: changeTimePerItem(); break;
            case 7: showStats(); break;
            case 8:
                printf("\nSystem closed. Thank you!\n");
                return 0;
            default:
                printf("\nInvalid choice.\n");
        }
    }
}
