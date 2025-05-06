import java.io.IOException;
import java.util.*;

public class LibraryApp {
    private static final Scanner scanner = new Scanner(System.in);
    private static final Library library = new Library();
    private static final UserManager userManager = new UserManager();

    private static final Map<String, Runnable> commands = new HashMap<>();

    public static void main(String[] args) {
        System.out.println("Welcome to the Library Information System!");
        initCommands();
        addSampleBooks();

        while (true) {
            System.out.print("\nEnter command: ");
            String input = scanner.nextLine().trim();

            if (commands.containsKey(input)) {
                commands.get(input).run();
            } else if (input.startsWith("books info")) {
                handleBooksInfo(input);
            } else if (input.startsWith("books find")) {
                handleBooksFind(input);
            } else if (input.startsWith("books sort")) {
                handleBooksSort(input);
            } else if (input.startsWith("users add")) {
                handleUsersAdd(input);
            } else if (input.startsWith("users remove")) {
                handleUsersRemove(input);
            } else {
                System.out.println("Unknown command. Type 'help' for available commands.");
            }
        }
    }

    private static void initCommands() {
        commands.put("help", LibraryApp::help);
        commands.put("open", LibraryApp::open);
        commands.put("close", LibraryApp::close);
        commands.put("save", LibraryApp::save);
        commands.put("saveas", LibraryApp::saveAs);
        commands.put("login", LibraryApp::login);
        commands.put("logout", LibraryApp::logout);
        commands.put("books all", LibraryApp::booksAll);
        commands.put("exit", LibraryApp::exit);
    }

    private static void addSampleBooks() {
        library.addBook(new Book("J.R.R. Tolkien", "The Hobbit", "Fantasy", 1937, 1001));
        library.addBook(new Book("George Orwell", "1984", "Dystopian", 1949, 1002));
        library.addBook(new Book("Jane Austen", "Pride and Prejudice", "Romance", 1813, 1003));
        library.addBook(new Book("F. Scott Fitzgerald", "The Great Gatsby", "Classic", 1925, 1004));
        library.addBook(new Book("Harper Lee", "To Kill a Mockingbird", "Fiction", 1960, 1005));
    }

    private static void help() {
        System.out.println("""
            Available commands:
            open
            close
            save
            saveas
            login
            logout
            books all
            books info <isbn>
            books find <option> <option_string>
            books sort <option> [asc|desc]
            users add <username> <password>
            users remove <username>
            exit
        """);
    }

    private static void open() {
        System.out.print("Enter books filename: ");
        String bookFile = scanner.nextLine();
        System.out.print("Enter users filename: ");
        String userFile = scanner.nextLine();
        try {
            library.loadBooks(bookFile);
            userManager.loadUsers(userFile);
            System.out.println("Files opened successfully.");
        } catch (ClassNotFoundException | IOException e) {
            System.out.println("Error loading files: " + e.getMessage());
        }
    }

    private static void close() {
        library.getBooks().clear();
        System.out.println("Library data cleared from memory.");
    }

    private static void save() {
        System.out.print("Enter books filename to save: ");
        String bookFile = scanner.nextLine();
        System.out.print("Enter users filename to save: ");
        String userFile = scanner.nextLine();
        try {
            library.saveBooks(bookFile);
            userManager.saveUsers(userFile);
            System.out.println("Data saved successfully.");
        } catch (IOException e) {
            System.out.println("Error saving files: " + e.getMessage());
        }
    }

    private static void saveAs() {
        save();
    }

    private static void login() {
        if (userManager.isLoggedIn()) {
            System.out.println("You are already logged in.");
            return;
        }
        System.out.print("Username: ");
        String username = scanner.nextLine();
        if (username == null || username.isEmpty()) {
            System.out.println("Invalid username.");
            return;
        }

        System.out.print("Password: ");
        String password = PasswordField.readPassword();
        if (password == null || password.isEmpty()) {
            System.out.println("Invalid password.");
            return;
        }

        userManager.login(username, password);
    }

    private static void logout() {
        userManager.logout();
    }

    private static void booksAll() {
        if (!userManager.isLoggedIn()) {
            System.out.println("Please log in first.");
            return;
        }
        library.listBooks();
    }

    private static void handleBooksInfo(String input) {
        if (!userManager.isLoggedIn()) {
            System.out.println("Please log in first.");
            return;
        }
        String[] parts = input.split(" ");
        if (parts.length < 3) {
            System.out.println("Usage: books info <isbn>");
            return;
        }
        try {
            int isbn = Integer.parseInt(parts[2]);
            library.showBookInfo(isbn);
        } catch (NumberFormatException e) {
            System.out.println("Invalid ISBN.");
        }
    }

    private static void handleBooksFind(String input) {
        if (!userManager.isLoggedIn()) {
            System.out.println("Please log in first.");
            return;
        }
        String[] parts = input.split(" ", 4);
        if (parts.length < 4) {
            System.out.println("Usage: books find <option> <option_string>");
            return;
        }
        library.findBooks(parts[2], parts[3]);
    }

    private static void handleBooksSort(String input) {
        if (!userManager.isLoggedIn()) {
            System.out.println("Please log in first.");
            return;
        }
        String[] parts = input.split(" ");
        if (parts.length < 3) {
            System.out.println("Usage: books sort <option> [asc|desc]");
            return;
        }
        String option = parts[2];
        boolean ascending = true;
        if (parts.length == 4) {
            ascending = !parts[3].equalsIgnoreCase("desc");
        }
        library.sortBooks(option, ascending);
    }

    private static void handleUsersAdd(String input) {
        if (!userManager.isLoggedIn() || !userManager.isAdmin()) {  // <-- ПРОМЕНЕНО
            System.out.println("Admin access required.");
            return;
        }
        String[] parts = input.split(" ");
        if (parts.length < 4) {
            System.out.println("Usage: users add <username> <password>");
            return;
        }
        userManager.addUser(parts[2], parts[3]);
    }

    private static void handleUsersRemove(String input) {
        if (!userManager.isLoggedIn() || !userManager.isAdmin()) {  // <-- ПРОМЕНЕНО
            System.out.println("Admin access required.");
            return;
        }
        String[] parts = input.split(" ");
        if (parts.length < 3) {
            System.out.println("Usage: users remove <username>");
            return;
        }
        userManager.removeUser(parts[2]);
    }

    private static void exit() {
        System.out.println("Exiting program...");
        System.exit(0);
    }
}

        }
        String[] parts = input.split(" ", 4);
        if (parts.length < 4) {
            System.out.println("Usage: books find <option> <option_string>");
            return;
        }
        library.findBooks(parts[2], parts[3]);
    }

    private static void handleBooksSort(String input) {
        if (!userManager.isLoggedIn()) {
            System.out.println("Please log in first.");
            return;
        }
        String[] parts = input.split(" ");
        if (parts.length < 3) {
            System.out.println("Usage: books sort <option> [asc|desc]");
            return;
        }
        String option = parts[2];
        boolean ascending = true;
        if (parts.length == 4) {
            ascending = !parts[3].equalsIgnoreCase("desc");
        }
        library.sortBooks(option, ascending);
    }

    private static void handleUsersAdd(String input) {
        if (!userManager.isLoggedIn() || userManager.isAdmin()) {
            System.out.println("Admin access required.");
            return;
        }
        String[] parts = input.split(" ");
        if (parts.length < 4) {
            System.out.println("Usage: users add <username> <password>");
            return;
        }
        userManager.addUser(parts[2], parts[3]);
    }

    private static void handleUsersRemove(String input) {
        if (!userManager.isLoggedIn() || userManager.isAdmin()) {
            System.out.println("Admin access required.");
            return;
        }
        String[] parts = input.split(" ");
        if (parts.length < 3) {
            System.out.println("Usage: users remove <username>");
            return;
        }
        userManager.removeUser(parts[2]);
    }

    private static void exit() {
        System.out.println("Exiting program...");
        System.exit(0);
    }
}

            System.out.println("No ratings available.");
        } else {
            System.out.println("Book ratings:");
            for (BookRating rating : ratings) {
                System.out.println(rating);
            }
        }
    }
}
