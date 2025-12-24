#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <ctype.h>

#define MAX_BOOKS 100
#define MAX_NAME 100
#define DB_FILE "library.csv"

typedef struct {
    char title[MAX_NAME];
    char author[MAX_NAME];
    char id[20];
    int total;
    int available;
} Book;

typedef struct {
    Book books[MAX_BOOKS];
    int total_books;
} Library;

void show_menu();
void setup_library(Library *lib);
void add_new_book(Library *lib);
void show_all_books(Library *lib);
void find_book(Library *lib);
void borrow_book(Library *lib);
void return_book(Library *lib);
void remove_book(Library *lib);
int find_book_by_id(Library *lib, char *id);
void clear_input();

void save_to_csv(Library *lib);
void load_from_csv(Library *lib);
void create_csv_if_not_exists();

int main() {
    Library my_library;
    int choice;

    setup_library(&my_library);
    create_csv_if_not_exists();
    load_from_csv(&my_library);

    printf("📚 Welcome to Simple Library Manager!\n");
    printf("💾 Database: %s\n", DB_FILE);

    do {
        show_menu();
        printf("What would you like to do? ");
        scanf("%d", &choice);
        clear_input();

        switch(choice) {
            case 1: add_new_book(&my_library); break;
            case 2: show_all_books(&my_library); break;
            case 3: find_book(&my_library); break;
            case 4: borrow_book(&my_library); break;
            case 5: return_book(&my_library); break;
            case 6: remove_book(&my_library); break;
            case 7:
                save_to_csv(&my_library);
                printf("💾 Data saved! Thanks for using our library! 👋\n");
                break;
            default:
                printf("❌ Oops! Please choose 1-7\n");
        }

        if(choice != 7) {
            save_to_csv(&my_library);
        }

    } while(choice != 7);

    return 0;
}

void create_csv_if_not_exists() {
    FILE *file = fopen(DB_FILE, "r");
    if(file == NULL) {
        file = fopen(DB_FILE, "w");
        if(file != NULL) {
            fprintf(file, "title,author,id,total,available\n");
            fclose(file);
            printf("📁 Created new database file: %s\n", DB_FILE);
        }
    } else {
        fclose(file);
    }
}

void save_to_csv(Library *lib) {
    FILE *file = fopen(DB_FILE, "w");
    if(file == NULL) {
        printf("❌ Error: Could not save to database!\n");
        return;
    }

    fprintf(file, "title,author,id,total,available\n");

    for(int i = 0; i < lib->total_books; i++) {
        fprintf(file, "%s,%s,%s,%d,%d\n",
                lib->books[i].title,
                lib->books[i].author,
                lib->books[i].id,
                lib->books[i].total,
                lib->books[i].available);
    }

    fclose(file);
}

void load_from_csv(Library *lib) {
    FILE *file = fopen(DB_FILE, "r");
    if(file == NULL) {
        printf("ℹ️ No existing database found. Starting fresh.\n");
        return;
    }

    char line[500];

    if(fgets(line, sizeof(line), file)) {}

    while(fgets(line, sizeof(line), file)) {
        if(lib->total_books >= MAX_BOOKS) {
            printf("⚠️ Database has more books than capacity.\n");
            break;
        }

        char *token;
        Book new_book;
        int field = 0;

        token = strtok(line, ",");
        while(token != NULL && field < 5) {
            token[strcspn(token, "\n")] = 0;

            switch(field) {
                case 0: strcpy(new_book.title, token); break;
                case 1: strcpy(new_book.author, token); break;
                case 2: strcpy(new_book.id, token); break;
                case 3: new_book.total = atoi(token); break;
                case 4: new_book.available = atoi(token); break;
            }

            token = strtok(NULL, ",");
            field++;
        }

        if(field == 5) {
            lib->books[lib->total_books++] = new_book;
        }
    }

    fclose(file);
    printf("📥 Loaded %d books from database.\n", lib->total_books);
}

void show_menu() {
    printf("\n========== LIBRARY MENU ==========\n");
    printf("1. 📖 Add New Book\n");
    printf("2. 📚 Show All Books\n");
    printf("3. 🔍 Find a Book\n");
    printf("4. 📥 Borrow Book\n");
    printf("5. 📤 Return Book\n");
    printf("6. 🗑️ Remove Book\n");
    printf("7. 💾 Save & Exit\n");
    printf("===================================\n");
}

void setup_library(Library *lib) {
    lib->total_books = 0;
}

void add_new_book(Library *lib) {
    if(lib->total_books >= MAX_BOOKS) {
        printf("❌ Library is full!\n");
        return;
    }

    Book new_book;

    printf("Book title: ");
    fgets(new_book.title, MAX_NAME, stdin);
    new_book.title[strcspn(new_book.title, "\n")] = 0;

    printf("Author: ");
    fgets(new_book.author, MAX_NAME, stdin);
    new_book.author[strcspn(new_book.author, "\n")] = 0;

    printf("Book ID: ");
    fgets(new_book.id, 20, stdin);
    new_book.id[strcspn(new_book.id, "\n")] = 0;

    if(find_book_by_id(lib, new_book.id) != -1) {
        printf("❌ Book ID already exists!\n");
        return;
    }

    printf("How many copies? ");
    scanf("%d", &new_book.total);
    clear_input();

    new_book.available = new_book.total;
    lib->books[lib->total_books++] = new_book;

    printf("✅ Book added successfully!\n");
}

void show_all_books(Library *lib) {
    if(lib->total_books == 0) {
        printf("📭 The library is empty!\n");
        return;
    }

    for(int i = 0; i < lib->total_books; i++) {
        printf("%d. %s by %s (%d/%d)\n",
               i + 1,
               lib->books[i].title,
               lib->books[i].author,
               lib->books[i].available,
               lib->books[i].total);
    }
}

void find_book(Library *lib) {
    char search[MAX_NAME];
    printf("Enter title or ID: ");
    fgets(search, MAX_NAME, stdin);
    search[strcspn(search, "\n")] = 0;

    int found = 0;
    for(int i = 0; i < lib->total_books; i++) {
        if(strstr(lib->books[i].title, search) ||
           strstr(lib->books[i].id, search)) {
            printf("%s by %s (%d/%d)\n",
                   lib->books[i].title,
                   lib->books[i].author,
                   lib->books[i].available,
                   lib->books[i].total);
            found = 1;
        }
    }

    if(!found) {
        printf("❌ No book found\n");
    }
}

void borrow_book(Library *lib) {
    char id[20];
    printf("Enter book ID: ");
    fgets(id, 20, stdin);
    id[strcspn(id, "\n")] = 0;

    int index = find_book_by_id(lib, id);
    if(index == -1) {
        printf("❌ Book not found\n");
        return;
    }

    if(lib->books[index].available > 0) {
        lib->books[index].available--;
        printf("✅ Book borrowed\n");
    } else {
        printf("❌ No copies available\n");
    }
}

void return_book(Library *lib) {
    char id[20];
    printf("Enter book ID: ");
    fgets(id, 20, stdin);
    id[strcspn(id, "\n")] = 0;

    int index = find_book_by_id(lib, id);
    if(index == -1) {
        printf("❌ Book not found\n");
        return;
    }

    if(lib->books[index].available < lib->books[index].total) {
        lib->books[index].available++;
        printf("✅ Book returned\n");
    } else {
        printf("❌ All copies already returned\n");
    }
}

void remove_book(Library *lib) {
    char id[20];
    printf("Enter book ID: ");
    fgets(id, 20, stdin);
    id[strcspn(id, "\n")] = 0;

    int index = find_book_by_id(lib, id);
    if(index == -1) {
        printf("❌ Book not found\n");
        return;
    }

    for(int i = index; i < lib->total_books - 1; i++) {
        lib->books[i] = lib->books[i + 1];
    }

    lib->total_books--;
    printf("✅ Book removed\n");
}

int find_book_by_id(Library *lib, char *id) {
    for(int i = 0; i < lib->total_books; i++) {
        if(strcmp(lib->books[i].id, id) == 0) {
            return i;
        }
    }
    return -1;
}

void clear_input() {
    int c;
    while((c = getchar()) != '\n' && c != EOF);
}
