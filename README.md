import java.util.Scanner;

public class LibraryApp {

    final static Library library = new Library();
    final static Scanner scanner = new Scanner(System.in);
    private static User currentUser = null;

    public static void main(String[] args) {
        while (true) {
            System.out.println("\nLibrary System");
            System.out.println("1. Log in");
            System.out.println("2. Register a new user");
            System.out.println("3. List books");
            System.out.println("4. Search books");
            System.out.println("5. Sort books");
            System.out.println("6. Borrow a book");
            System.out.println("7. Return a book");
            System.out.println("8. Add book rating");
            System.out.println("9. Show book ratings");
            System.out.println("10. Save and Exit");
            if (currentUser != null && currentUser.isAdmin()) {
                System.out.println("11. Add book");
                System.out.println("12. Remove book");
                System.out.println("13. Add user");
                System.out.println("14. Remove user");
            }
            System.out.print("Choose an option: ");

            int choice = Integer.parseInt(scanner.nextLine());

            switch (choice) {
                case 1:
                    login();
                    break;
                case 2:
                    registerUser();
                    break;
                case 3:
                    library.listBooks();
                    break;
                case 4:
                    searchBooks();
                    break;
                case 5:
                    sortBooks();
                    break;
                case 6:
                    borrowBook();
                    break;
                case 7:
                    returnBook();
                    break;
                case 8:
                    addBookRating();
                    break;
                case 9:
                    library.showBookRatings();
                    break;
                case 10:
                    FileManager.saveUsers(library.getUsers(), "users.dat");
                    FileManager.saveBooks(library.getBooks(), "books.dat");
                    System.out.println("Data saved successfully. Exiting...");
                    return;
                case 11:
                    if (currentUser.isAdmin()) {
                        addBook();
                    } else {
                        System.out.println("You need to be an admin to add books.");
                    }
                    break;
                case 12:
                    if (currentUser.isAdmin()) {
                        removeBook();
                    } else {
                        System.out.println("You need to be an admin to remove books.");
                    }
                    break;
                case 13:
                    if (currentUser.isAdmin()) {
                        addUser();
                    } else {
                        System.out.println("You need to be an admin to add users.");
                    }
                    break;
                case 14:
                    if (currentUser.isAdmin()) {
                        removeUser();
                    } else {
                        System.out.println("You need to be an admin to remove users.");
                    }
                    break;
                default:
                    System.out.println("Invalid choice. Please try again.");
            }
        }
    }

    private static void login() {
        System.out.print("Enter username: ");
        String username = scanner.nextLine();
        System.out.print("Enter password: ");
        String password = new String(System.console().readPassword("Enter password: ")); // Парола със звездички

        User user = library.authenticate(username, password);
        if (user != null) {
            currentUser = user;
            System.out.println("Login successful! Welcome " + user.getUsername());
        } else {
            System.out.println("Invalid username or password.");
        }
    }

    private static void registerUser() {
        System.out.print("Enter username: ");
        String username = scanner.nextLine();
        System.out.print("Enter password: ");
        String password = scanner.nextLine();
        System.out.print("Is this user an admin? (true/false): ");
        boolean isAdmin = Boolean.parseBoolean(scanner.nextLine());

        library.addUser(username, password, isAdmin);
    }

    private static void searchBooks() {
        System.out.print("Enter search keyword: ");
        String keyword = scanner.nextLine();
        library.searchBooks(keyword);
    }

    private static void sortBooks() {
        System.out.print("Sort books by (title/author/rating): ");
        String criteria = scanner.nextLine();
        library.sortBooks(criteria);
    }

    private static void borrowBook() {
        System.out.print("Enter user ID: ");
        String userId = scanner.nextLine();
        System.out.print("Enter book ID: ");
        String bookId = scanner.nextLine();
        library.borrowBook(userId, bookId);
    }

    private static void returnBook() {
        System.out.print("Enter user ID: ");
        String userId = scanner.nextLine();
        System.out.print("Enter book ID: ");
        String bookId = scanner.nextLine();
        library.returnBook(userId, bookId);
    }

    private static void addBookRating() {
        System.out.print("Enter book ID: ");
        String bookId = scanner.nextLine();
        System.out.print("Enter user ID: ");
        String userId = scanner.nextLine();
        System.out.print("Enter rating (1-5): ");
        int rating = Integer.parseInt(scanner.nextLine());

        if (rating < 1 || rating > 5) {
            System.out.println("Invalid rating. It should be between 1 and 5.");
            return;
        }

        library.addBookRating(bookId, userId, rating);
    }

    private static void addBook() {
        System.out.print("Enter book title: ");
        String title = scanner.nextLine();
        System.out.print("Enter book author: ");
        String author = scanner.nextLine();
        System.out.print("Enter book ID: ");
        String id = scanner.nextLine();
        Book book = new Book(title, author, id);
        library.addBook(book);
        System.out.println("Book added successfully.");
    }

    private static void removeBook() {
        System.out.print("Enter book ID to remove: ");
        String id = scanner.nextLine();
        library.removeBook(id);
    }

    private static void addUser() {
        System.out.print("Enter username: ");
        String username = scanner.nextLine();
        System.out.print("Enter password: ");
        String password = scanner.nextLine();
        System.out.print("Is this user an admin? (true/false): ");
        boolean isAdmin = Boolean.parseBoolean(scanner.nextLine());
        library.addUser(username, password, isAdmin);
    }

    private static void removeUser() {
        System.out.print("Enter username to remove: ");
        String username = scanner.nextLine();
        library.removeUser(username);
    }
}
