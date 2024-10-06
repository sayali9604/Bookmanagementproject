#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define INITIAL_CAPACITY 10 // Initial size of array

// Structure to store book details
typedef struct {
    int bookID;
    char bookName[50];
    char authorName[50];
    char category[30];
    float price;
    float rating;
} Book;

// Global pointer use here cause we can acces it globally
Book* books = NULL;
int numBooks = 0;
int capacity = INITIAL_CAPACITY;

// Function declarations
void addBook();
void removeBook();
void searchBook();
void showAuthorsBooks();
void showCategoryBooks();
void updateBookData();
void displaySortedBooks(int sortByPrice, int ascending);
void displayAllBooks();

//  functions for printing
void printHeader();
void printBook(const Book* b);

//  functions for dynamic memory management
void ensureCapacity();
void freeMemory();

//  functions for searching
int findBookByID(int id);
int findBookByName(char* name);

// Function to add a book to the system
void addBook() {
    ensureCapacity(); // Ensure enough capacity for new book

    Book newBook;
    printf("Enter Book ID: ");
    scanf("%d", &newBook.bookID);
    getchar(); // Clear newline
    printf("Enter Book Name: ");
    fgets(newBook.bookName, sizeof(newBook.bookName), stdin);
    newBook.bookName[strcspn(newBook.bookName, "\n")] = 0; // Remove newline

    printf("Enter Author Name: ");
    fgets(newBook.authorName, sizeof(newBook.authorName), stdin);
    newBook.authorName[strcspn(newBook.authorName, "\n")] = 0;

    printf("Enter Category: ");
    fgets(newBook.category, sizeof(newBook.category), stdin);
    newBook.category[strcspn(newBook.category, "\n")] = 0;

    printf("Enter Price: ");
    scanf("%f", &newBook.price);

    printf("Enter Rating (0.0 to 5.0): ");
    scanf("%f", &newBook.rating);

    books[numBooks++] = newBook;
    printf("Book added successfully!\n");
}

// Function to remove a book by ID
void removeBook() {
    int id;
    printf("Enter Book ID to remove: ");
    scanf("%d", &id);
//we called bookbyid to search
    int index = findBookByID(id);
    if (index == -1) {
        printf("Book ID not found!\n");
        return;
    }

    for (int i = index; i < numBooks - 1; i++) {
        books[i] = books[i + 1];
    }
    numBooks--;

    printf("Book removed successfully!\n");
}

// Function to search for a book by ID or Name
void searchBook() {
    int choice;
    printf("Search by: 1. Book ID 2. Book Name\n");
    scanf("%d", &choice);
    getchar(); //  it use to Clear newline

    if (choice == 1) {
        int id;
        printf("Enter Book ID to search: ");
        scanf("%d", &id);
        int index = findBookByID(id);
        if (index != -1) {
            printHeader();
            printBook(&books[index]);
        } else {
            printf("Book ID not found!\n");
        }
    } else if (choice == 2) {
        char name[50];
        printf("Enter Book Name to search: ");
        fgets(name, sizeof(name), stdin);
        name[strcspn(name, "\n")] = 0;
        int index = findBookByName(name);
        if (index != -1) {
            printHeader();
            printBook(&books[index]);
        } else {
            printf("Book Name not found!\n");
        }
    } else {
        printf("Invalid choice!\n");
    }
}

// Function to show all books author
void showAuthorsBooks() {
    char author[50];
    printf("Enter Author Name: ");
    getchar(); // Clear newline
    fgets(author, sizeof(author), stdin);
    author[strcspn(author, "\n")] = 0;

    int found = 0;
    printHeader();
    for (int i = 0; i < numBooks; i++) {
        if (strcmp(books[i].authorName, author) == 0) {
            printBook(&books[i]);
            found = 1;
        }
    }

    if (!found) {
        printf("| No books found by this author.                                                                             |\n");
    }
}

// Function to show all books by category
void showCategoryBooks() {
    char category[30];
    printf("Enter Category: ");
    getchar(); // Clear newline
    fgets(category, sizeof(category), stdin);
    category[strcspn(category, "\n")] = 0;

    int found = 0;
    printHeader();
    for (int i = 0; i < numBooks; i++) {
        if (strcmp(books[i].category, category) == 0) {
            printBook(&books[i]);
            found = 1;
        }
    }

    if (!found) {
        printf("| No books found in this category.                                                                           |\n");
    }
}

// Function to update a book's price and rating
void updateBookData() {
    int id;
    printf("Enter Book ID to update: ");
    scanf("%d", &id);

    int index = findBookByID(id);
    if (index == -1) {
        printf("Book ID not found!\n");
        return;
    }

    printf("Updating book '%s' by '%s'.\n", books[index].bookName, books[index].authorName);
    printf("Enter new Price: ");
    scanf("%f", &books[index].price);
    printf("Enter new Rating (0.0 to 5.0): ");
    scanf("%f", &books[index].rating);

    printf("Book updated successfully!\n");
}

// Function to display sorted books
void displaySortedBooks(int sortByPrice, int ascending) {
    if (numBooks == 0) {
        printf("No books to display!\n");
        return;
    }

    Book* sortedBooks = malloc(numBooks * sizeof(Book));
    memcpy(sortedBooks, books, numBooks * sizeof(Book));

    // Sort books based on price or rating
    for (int i = 0; i < numBooks - 1; i++) {
        for (int j = i + 1; j < numBooks; j++) {
            int condition;//here we used ternary operator
            if (sortByPrice) {
                condition = ascending ? sortedBooks[i].price > sortedBooks[j].price
                                      : sortedBooks[i].price < sortedBooks[j].price;
            } else {
                condition = ascending ? sortedBooks[i].rating > sortedBooks[j].rating
                                      : sortedBooks[i].rating < sortedBooks[j].rating;
            }

            if (condition) {
                Book temp = sortedBooks[i];
                sortedBooks[i] = sortedBooks[j];
                sortedBooks[j] = temp;
            }
        }
    }

    printHeader();
    for (int i = 0; i < numBooks; i++) {
        printBook(&sortedBooks[i]);
    }

    free(sortedBooks); // Free the memory allocated for sorted books
}

// Function to display all books
void displayAllBooks() {
    if (numBooks == 0) {
        printf("No books to display!\n");
        return;
    }

    printHeader();
    for (int i = 0; i < numBooks; i++) {
        printBook(&books[i]);
    }
}
 //this is used for making our project element in table format
//  function to print table header
void printHeader() {
    printf("+--------+-------------------------------+---------------------------+---------------------+--------+--------+\n");
    printf("| BookID | Book Name                     | Author Name               | Category            | Price  | Rating |\n");
    printf("+--------+-------------------------------+---------------------------+---------------------+--------+--------+\n");
}

//  function to print a book row
void printBook(const Book* b) {
    printf("| %-6d | %-29s | %-25s | %-19s | %-6.2f | %-6.2f |\n",
           b->bookID, b->bookName, b->authorName, b->category, b->price, b->rating);
}//->it is showing use this book id ..etc

//  function to find a book by ID
int findBookByID(int id) {
    for (int i = 0; i < numBooks; i++) {
        if (books[i].bookID == id) {
            return i;
        }
    }
    return -1;
}

// function to find a book by Name
int findBookByName(char* name) {
    for (int i = 0; i < numBooks; i++) {
        if (strcmp(books[i].bookName, name) == 0) {
            return i;
        }
    }
    return -1;
}

//  this is used increase capacity using realloc .whic is DMA
void ensureCapacity() {
    if (numBooks >= capacity) {
        capacity *= 2; // Double the capacity
        books = realloc(books, capacity * sizeof(Book));
        if (books == NULL) {
            printf("Error: Memory allocation failed!\n");
            exit(1);
        }
    }
}

// Function to free the dynamically allocated memory
void freeMemory() {
    if (books != NULL) {
        free(books);
    }
}

// Main function
int main() {
    books = malloc(INITIAL_CAPACITY * sizeof(Book));
    if (books == NULL) {
        printf("Error: Memory allocation failed!\n");
        return 1;
    }

    int choice;
    while (1) {
        printf("\n--- Book Management System ---\n");
        printf("1. Add Book\n");
        printf("2. Remove Book\n");
        printf("3. Search Book by ID or Name\n");
        printf("4. Show Author's Books\n");
        printf("5. Show Category's Books\n");
        printf("6. Update Book Data\n");
        printf("7. Display Sorted Books by Price (1) or Rating (0)\n");
        printf("8. Display All Books\n");
        printf("9. Exit\n");
        printf("Enter your choice: ");
        scanf("%d", &choice);

        switch (choice) {
            case 1:
                addBook();
                break;
            case 2:
                removeBook();
                break;
            case 3:
                searchBook();
                break;
            case 4:
                showAuthorsBooks();
                break;
            case 5:
                showCategoryBooks();
                break;
            case 6:
                updateBookData();
                break;
            case 7:
                printf("Sort by: 1. Price 0. Rating\n");
                int sortByPrice;
                scanf("%d", &sortByPrice);
                printf("Sort Order: 1. Ascending 0. Descending\n");
                int ascending;
                scanf("%d", &ascending);
                displaySortedBooks(sortByPrice, ascending);
                break;
            case 8:
                displayAllBooks();
                break;
            case 9:
                freeMemory();
                printf("Exiting...\n");
                return 0;
            default:
                printf("Invalid choice!\n");
        }
    }

    return 0;
}
