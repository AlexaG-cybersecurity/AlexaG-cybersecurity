import java.io.IOException;
import java.util.*;

/**
 * Главен клас за библиотечната информационна система.
 * Отговаря за комуникацията с потребителя (чрез конзолата),
 * управлението на командите и координацията между библиотеката и потребителите.
 * <p>
 * Този клас поддържа операции като:
 * <ul>
 *   <li>Зареждане и запис на книги и потребители във файлове.</li>
 *   <li>Влизане и излизане на потребители с различни нива на достъп.</li>
 *   <li>Добавяне, редактиране, изтриване и търсене на книги.</li>
 *   <li>Управление на потребители (добавяне, изтриване, изброяване).</li>
 *   <li>Оценяване на книги от регистрирани потребители.</li>
 * </ul>
 */
public class LibraryApp {
    private static final Scanner scanner = new Scanner(System.in);
    private static final Library library = new Library();
    private static final UserManager userManager = new UserManager();

    /**
     * Структура за съхранение на оценки на книги.
     * Ключ: ISBN номер на книгата (Integer).
     * Стойност: Списък от оценки (BookRating), направени от потребители.
     */
    private static final Map<Integer, List<BookRating>> bookRatings = new HashMap<>();

    /**
     * Словар (Map) за съхранение на наличните команди и свързаните им действия.
     * Ключ: команда (низ).
     * Стойност: действие (Runnable) - метод, който се изпълнява при въвеждане на командата.
     */
    private static final Map<String, Runnable> commands = new HashMap<>();

    /**
     * Основната точка на вход в приложението.
     * Тук се инициализират примерни книги, зареждат се данни от файлове,
     * регистрират се командите и стартира потребителския интерфейс.
     *
     * @param args аргументи от командния ред (не се използват).
     */
    public static void main(String[] args) {
        System.out.println("Welcome to the Library Information System!");
        initCommands();
        addSampleBooks();

        // Опитваме се да заредим вече съществуващи данни от файлове
        try {
            library.loadBooks("books.dat");
            userManager.loadUsers("users.dat");
        } catch (IOException | ClassNotFoundException e) {
            System.out.println("No saved data found, starting fresh.");
        }

        // Основен цикъл за обработка на потребителски команди
        while (true) {
            System.out.print("\nEnter command: ");
            String input = scanner.nextLine().trim().toLowerCase();

            if (commands.containsKey(input)) {
                commands.get(input).run();
            } else {
                System.out.println("Unknown command. Type 'help' for available commands.");
            }
        }
    }

    /**
     * Регистрира всички команди в системата, свързвайки низовете с методите за тяхното изпълнение.
     * Това позволява лесно добавяне и управление на командите от един централен обект.
     */
    private static void initCommands() {
        commands.put("help", LibraryApp::help);
        commands.put("open", LibraryApp::open);
        commands.put("close", LibraryApp::close);
        commands.put("save", LibraryApp::save);
        commands.put("saveas", LibraryApp::saveAs);
        commands.put("login", LibraryApp::login);
        commands.put("logout", LibraryApp::logout);
        commands.put("books all", LibraryApp::booksAll);
        commands.put("books add", LibraryApp::booksAdd);
        commands.put("books update", LibraryApp::booksUpdate);
        commands.put("books delete", LibraryApp::booksDelete);
        commands.put("books find", LibraryApp::booksFind);
        commands.put("books sort", LibraryApp::booksSort);
        commands.put("books rate", LibraryApp::booksRate);
        commands.put("users add", LibraryApp::usersAdd);
        commands.put("users delete", LibraryApp::usersDelete);
        commands.put("users list", LibraryApp::usersList);
        commands.put("exit", LibraryApp::exit);
    }

    /**
     * Добавя няколко примерни книги в библиотеката, ако тя е празна.
     * Това улеснява тестването и демонстрирането на функционалностите.
     */
    private static void addSampleBooks() {
        if (!library.hasBooks()) {
            library.addBook(new Book("J.R.R. Tolkien", "The Hobbit", "Fantasy", 1937, 1001));
            library.addBook(new Book("George Orwell", "1984", "Dystopian", 1949, 1002));
            library.addBook(new Book("Jane Austen", "Pride and Prejudice", "Romance", 1813, 1003));
            library.addBook(new Book("F. Scott Fitzgerald", "The Great Gatsby", "Classic", 1925, 1004));
            library.addBook(new Book("Harper Lee", "To Kill a Mockingbird", "Fiction", 1960, 1005));
        }
    }

    /**
     * Извежда списък с наличните команди и кратко описание за тях.
     * Позволява на потребителя да се ориентира какви операции може да извърши.
     */
    private static void help() {
        System.out.println("""
            Available commands:
            open             - Load books and users from files
            close            - Clear loaded data from memory
            save             - Save books and users to files
            saveas           - Save books and users to files (same as save)
            login            - Login to the system
            logout           - Logout from the system
            books all        - List all books with average ratings
            books add        - Add a new book (admin only)
            books update     - Update a book (admin only)
            books delete     - Delete a book (admin only)
            books find       - Find books by author, title or genre
            books sort       - Sort books by author, title or year
            books rate       - Rate a book (logged in users only)
            users add        - Add a new user (admin only)
            users delete     - Delete a user (admin only)
            users list       - List all users (admin only)
            exit             - Exit the application
            """);
    }

    /**
     * Зарежда данни за книги и потребители от файлове, посочени от потребителя.
     * Ако файловете не съществуват или има грешка при четене, показва съобщение.
     */
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

    /**
     * Изчиства всички данни за книги от паметта (но не ги изтрива от файловете).
     * Полезно при нужда за презареждане или рестартиране без изход.
     */
    private static void close() {
        library.getBooks().clear();
        System.out.println("Library data cleared from memory.");
    }

    /**
     * Записва текущите данни за книги и потребители във файловете по подразбиране.
     * При грешка изписва съобщение.
     */
    private static void save() {
        try {
            library.saveBooks("books.dat");
            userManager.saveUsers("users.dat");
            System.out.println("Data saved successfully.");
        } catch (IOException e) {
            System.out.println("Error saving files: " + e.getMessage());
        }
    }

    /**
     * За сега се държи като save() — запазва в стандартните файлове.
     */
    private static void saveAs() {
        save();
    }

    /**
     * Позволява на потребител да влезе в системата.
     * Ако вече е влязъл, информира за това.
     * Изисква въвеждане на потребителско име и парола.
     */
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

    /**
     * Извършва изход на текущия влязъл потребител.
     * Ако няма такъв, показва съответното съобщение.
     */
    private static void logout() {
        userManager.logout();
    }

    /**
     * Извежда всички книги с тяхната средна оценка.
     * Ако няма книги, показва съобщение.
     */
    private static void booksAll() {
        if (library.getBooks().isEmpty()) {
            System.out.println("No books found.");
            return;
        }

        System.out.println("Books:");
        for (Book book : library.getBooks()) {
            double avgRating = calculateAverageRating(book.getIsbn());
            System.out.printf("%s | Average rating: %.2f%n", book, avgRating);
        }
    }

    /**
     * Изчислява средната оценка за книга по ISBN.
     *
     * @param isbn ISBN на книгата
     * @return средна оценка (от 0 до 5), 0 ако няма оценки
     */
    private static double calculateAverageRating(int isbn) {
        List<BookRating> ratings = bookRatings.get(isbn);
        if (ratings == null || ratings.isEmpty()) return 0;

        int sum = 0;
        for (BookRating r : ratings) {
            sum += r.getRating();
        }
        return (double) sum / ratings.size();
    }

    /**
     * Добавя нова книга в библиотеката.
     * Разрешено само за администратор.
     * Изисква въвеждане на всички атрибути на книгата.
     */
    private static void booksAdd() {
        if (!userManager.isLoggedIn() || !userManager.isAdmin()) {
            System.out.println("Only admin users can add books.");
            return;
        }

        System.out.print("Author: ");
        String author = scanner.nextLine().trim();
        System.out.print("Title: ");
        String title = scanner.nextLine().trim();
        System.out.print("Genre: ");
        String genre = scanner.nextLine().trim();
        System.out.print("Year: ");
        int year;
        try {
            year = Integer.parseInt(scanner.nextLine().trim());
        } catch (NumberFormatException e) {
            System.out.println("Invalid year.");
            return;
        }
        System.out.print("ISBN: ");
        int isbn;
        try {
            isbn = Integer.parseInt(scanner.nextLine().trim());
        } catch (NumberFormatException e) {
            System.out.println("Invalid ISBN.");
            return;
        }

        Book newBook = new Book(author, title, genre, year, isbn);
        library.addBook(newBook);
        System.out.println("Book added successfully.");
    }

    /**
     * Редактира съществуваща книга по ISBN.
     * Разрешено само за администратор.
     * Позволява промяна на всички атрибути на книгата.
     */
    private static void booksUpdate() {
        if (!userManager.isLoggedIn() || !userManager.isAdmin()) {
            System.out.println("Only admin users can update books.");
            return;
        }

        System.out.print("Enter ISBN of the book to update: ");
        int isbn;
        try {
            isbn = Integer.parseInt(scanner.nextLine().trim());
        } catch (NumberFormatException e) {
            System.out.println("Invalid ISBN.");
            return;
        }

        Book book = library.getBook(isbn);
        if (book == null) {
            System.out.println("Book not found.");
            return;
        }

        System.out.print("New author (leave blank to keep): ");
        String author = scanner.nextLine().trim();
        System.out.print("New title (leave blank to keep): ");
        String title = scanner.nextLine().trim();
        System.out.print("New genre (leave blank to keep): ");
        String genre = scanner.nextLine().trim();
        System.out.print("New year (leave blank to keep): ");
        String yearStr = scanner.nextLine().trim();

        int year = book.year();
        if (!yearStr.isEmpty()) {
            try {
                year = Integer.parseInt(yearStr);
            } catch (NumberFormatException e) {
                System.out.println("Invalid year. Keeping old value.");
            }
        }

        // Създаваме нова книга с актуализирани стойности (ако не са подадени нови - запазваме старите)
        Book updatedBook = new Book(
                author.isEmpty() ? book.author() : author,
                title.isEmpty() ? book.title() : title,
                genre.isEmpty() ? book.genre() : genre,
                year,
                isbn
        );

        library.updateBook(updatedBook);
        System.out.println("Book updated successfully.");
    }

    /**
     * Изтрива книга по ISBN.
     * Разрешено само за администратор.
     */
    private static void booksDelete() {
        if (!userManager.isLoggedIn() || !userManager.isAdmin()) {
            System.out.println("Only admin users can delete books.");
            return;
        }

        System.out.print("Enter ISBN of the book to delete: ");
        int isbn;
        try {
            isbn = Integer.parseInt(scanner.nextLine().trim());
        } catch (NumberFormatException e) {
            System.out.println("Invalid ISBN.");
            return;
        }

        boolean removed = library.removeBook(isbn);
        if (removed) {
            System.out.println("Book deleted successfully.");
            bookRatings.remove(isbn); // премахваме и оценките за тази книга
        } else {
            System.out.println("Book not found.");
        }
    }

    /**
     * Търси книги по автор, заглавие или жанр, използвайки подниз.
     * Извежда намерените резултати.
     */
    private static void booksFind() {
        System.out.print("Search by author, title or genre: ");
        String query = scanner.nextLine().toLowerCase();

        List<Book> foundBooks = library.findBooks(query);
        if (foundBooks.isEmpty()) {
            System.out.println("No books found.");
            return;
        }

        System.out.println("Found books:");
        for (Book b : foundBooks) {
            double avgRating = calculateAverageRating(b.getIsbn());
            System.out.printf("%s | Average rating: %.2f%n", b, avgRating);
        }
    }

    /**
     * Сортира книгите според избран критерий (автор, заглавие или година) и посока (възходяща/низходяща).
     */
    private static void booksSort() {
        System.out.print("Sort by (author/title/year): ");
        String option = scanner.nextLine().toLowerCase();

        if (!List.of("author", "title", "year").contains(option)) {
            System.out.println("Invalid sort option.");
            return;
        }

        System.out.print("Ascending? (yes/no): ");
        String ascInput = scanner.nextLine().trim().toLowerCase();
        boolean ascending = ascInput.equals("yes") || ascInput.equals("y");

        library.sortBooks(option, ascending);
        System.out.println("Books sorted.");
    }

    /**
     * Позволява на влязъл потребител да оцени книга с рейтинг от 1 до 5.
     * Записва оценката в паметта.
     */
    private static void booksRate() {
        if (!userManager.isLoggedIn()) {
            System.out.println("You must be logged in to rate books.");
            return;
        }

        System.out.print("Enter ISBN of the book to rate: ");
        int isbn;
        try {
            isbn = Integer.parseInt(scanner.nextLine().trim());
        } catch (NumberFormatException e) {
            System.out.println("Invalid ISBN.");
            return;
        }

        Book book = library.getBook(isbn);
        if (book == null) {
            System.out.println("Book not found.");
            return;
        }

        System.out.print("Enter rating (1-5): ");
        int rating;
        try {
            rating = Integer.parseInt(scanner.nextLine().trim());
            if (rating < 1 || rating > 5) throw new NumberFormatException();
        } catch (NumberFormatException e) {
            System.out.println("Invalid rating. Must be between 1 and 5.");
            return;
        }

        BookRating bookRating = new BookRating(String.valueOf(isbn), userManager.getLoggedInUser().getUsername(), rating);
        bookRatings.computeIfAbsent(isbn, k -> new ArrayList<>()).add(bookRating);
        System.out.println("Thank you for rating the book!");
    }

    /**
     * Добавя нов потребител с клиентски достъп (CLIENT).
     * Разрешено само за администратор.
     */
    private static void usersAdd() {
        if (!userManager.isLoggedIn() || !userManager.isAdmin()) {
            System.out.println("Only admin users can add new users.");
            return;
        }

        System.out.print("New username: ");
        String username = scanner.nextLine().trim();
        if (username.isEmpty()) {
            System.out.println("Invalid username.");
            return;
        }

        System.out.print("Password: ");
        String password = PasswordField.readPassword();
        if (password.isEmpty()) {
            System.out.println("Invalid password.");
            return;
        }

        userManager.addUser(username, password);
    }

    /**
     * Изтрива потребител по потребителско име.
     * Администраторски достъп.
     * Не може да се изтрие администраторският акаунт.
     */
    private static void usersDelete() {
        if (!userManager.isLoggedIn() || !userManager.isAdmin()) {
            System.out.println("Only admin users can delete users.");
            return;
        }

        System.out.print("Username to delete: ");
        String username = scanner.nextLine().trim();

        if ("admin".equalsIgnoreCase(username)) {
            System.out.println("Cannot delete the admin user.");
            return;
        }

        userManager.removeUser(username);
    }

    /**
     * Извежда списък с всички регистрирани потребители.
     * Администраторски достъп.
     */
    private static void usersList() {
        if (!userManager.isLoggedIn() || !userManager.isAdmin()) {
            System.out.println("Only admin users can list users.");
            return;
        }

        System.out.println("Users:");
        userManager.listUsers();
    }

    /**
     * Завършва изпълнението на приложението с прощално съобщение.
     */
    private static void exit() {
        System.out.println("Goodbye!");
        System.exit(0);
    }
import java.io.Serial;
import java.io.Serializable;

/**
 * Класът User представлява потребител на библиотечната система.
 * Всеки потребител има потребителско име, парола и ниво на достъп.
 * <p>
 * Връзка с LibraryApp и UserManager:
 * - User обекти се съхраняват и управляват от UserManager.
 * - LibraryApp използва User за автентикация и авторизация.
 * - Нивото на достъп (AccessLevel) определя правата на потребителя (например администратор или клиент).
 * - Класът е сериализуем, за да може потребителите да се записват и зареждат от файлове.
 */
public class User implements Serializable {
    @Serial
    private static final long serialVersionUID = 1L;

    /** Потребителско име (уникално) */
    private final String username;

    /** Парола на потребителя (съхранява се като plain text - за по-добра сигурност може да се криптира) */
    private final String password;

    /** Ниво на достъп на потребителя (ADMIN или CLIENT) */
    private final AccessLevel accessLevel;

    /**
     * Конструктор за създаване на нов потребител с конкретно ниво на достъп.
     *
     * @param username потребителско име
     * @param password парола
     * @param accessLevel ниво на достъп (например ADMIN или CLIENT)
     */
    public User(String username, String password, AccessLevel accessLevel) {
        this.username = username;
        this.password = password;
        this.accessLevel = accessLevel;
    }

    /**
     * Връща потребителското име.
     *
     * @return потребителско име
     */
    public String getUsername() {
        return username;
    }

    /**
     * Връща паролата на потребителя.
     *
     * @return парола
     */
    public String getPassword() {
        return password;
    }

    /**
     * Връща нивото на достъп на потребителя.
     *
     * @return AccessLevel (ADMIN или CLIENT)
     */
    public AccessLevel getAccessLevel() {
        return accessLevel;
    }

    /**
     * Проверява дали потребителят е администратор.
     *
     * @return true ако потребителят е с администраторски права, иначе false
     */
    public boolean isAdmin() {
        return accessLevel == AccessLevel.ADMIN;
    }

    /**
     * Връща информация за потребителя под формата на текст.
     * Паролата не се показва за сигурност.
     *
     * @return текст с потребителско име и ниво на достъп
     */
    @Override
    public String toString() {
        return "User{username='" + username + "', accessLevel=" + accessLevel + "}";
    }
}
import java.io.Console;
import java.util.Scanner;

/**
 * Клас PasswordField осигурява метод за сигурно въвеждане на парола от конзолата.
 * <p>
 * Той използва системната конзола (Console) за скриване на въвеждането на паролата,
 * ако това е възможно. Ако конзолата не е налична (например при стартиране в IDE),
 * въвеждането става стандартно (без скриване).
 * <p>
 * Връзка с LibraryApp и UserManager:
 * - Използва се при процеса на логин за въвеждане на парола от потребителя.
 * - Подпомага по-сигурната автентикация на потребителите.
 */
public class PasswordField {

    /**
     * Чете парола от конзолата, като скрива въвеждането, ако е възможно.
     *
     * @return Въведената парола като String.
     */
    public static String readPassword() {
        Console console = System.console();
        if (console != null) {
            // Ако има системна конзола - въвеждането се скрива
            return new String(console.readPassword());
        } else {
            // Ако няма системна конзола (например IDE), паролата се въвежда нормално
            System.out.print("(Password hidden input not supported, type normally): ");
            return new Scanner(System.in).nextLine();
        }
    }
}
import java.io.IOException;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.util.*;

/**
 * Клас Library управлява колекцията от книги.
 * <p>
 * Предоставя операции за добавяне, търсене, сортиране, обновяване, изтриване,
 * както и зареждане и записване на книги във файл чрез сериализация.
 */
public class Library {

    /** Колекция от книги, където ключът е ISBN (уникален идентификатор) */
    private Map<Integer, Book> books = new HashMap<>();

    /**
     * Зарежда колекцията от книги от файл (сериализиран обект).
     *
     * @param filename име на файла, от който да зареди книгите.
     * @throws IOException при проблем с четене на файла.
     * @throws ClassNotFoundException при несъвпадение на класове.
     */
    public void loadBooks(String filename) throws IOException, ClassNotFoundException {
        try (ObjectInputStream in = new ObjectInputStream(new FileInputStream(filename))) {
            books = (Map<Integer, Book>) in.readObject();
        }
    }

    /**
     * Записва текущата колекция от книги във файл чрез сериализация.
     *
     * @param filename име на файла, където да се запишат книгите.
     * @throws IOException при проблем с писане във файла.
     */
    public void saveBooks(String filename) throws IOException {
        try (ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream(filename))) {
            out.writeObject(books);
        }
    }

    /**
     * Извежда всички книги в библиотеката.
     * Ако няма книги - извежда съобщение.
     */
    public void listBooks() {
        if (books.isEmpty()) {
            System.out.println("No books available.");
            return;
        }
        books.values().forEach(System.out::println);
    }

    /**
     * Показва информация за книга по ISBN.
     *
     * @param isbn уникален номер на книгата.
     */
    public void showBookInfo(int isbn) {
        Book book = books.get(isbn);
        if (book != null) {
            System.out.println(book);
        } else {
            System.out.println("Book not found.");
        }
    }

    /**
     * Търси книги по критерий (title, author, genre) и текст.
     * Търсенето е case-insensitive.
     *
     * @param option критерий за търсене.
     * @param value текст, който да се търси.
     */
    public void findBooks(String option, String value) {
        boolean found = false;
        String valLower = value.toLowerCase();
        for (Book book : books.values()) {
            switch (option.toLowerCase()) {
                case "title":
                    if (book.title().toLowerCase().contains(valLower)) found = printAndFlag(book, found);
                    break;
                case "author":
                    if (book.author().toLowerCase().contains(valLower)) found = printAndFlag(book, found);
                    break;
                case "genre":
                    if (book.genre().toLowerCase().contains(valLower)) found = printAndFlag(book, found);
                    break;
                default:
                    System.out.println("Invalid search option.");
                    return;
            }
        }
        if (!found) System.out.println("No books found matching your criteria.");
    }

    /** Помощен метод за извеждане и маркиране, че има резултати */
    private boolean printAndFlag(Book book, boolean found) {
        System.out.println(book);
        return true;
    }

    /**
     * Сортира книгите по критерий и посока (възходящ/низходящ).
     *
     * @param option критерий за сортиране (title, author, year).
     * @param ascending дали е възходящо сортиране.
     */
    public void sortBooks(String option, boolean ascending) {
        List<Book> list = new ArrayList<>(books.values());
        Comparator<Book> comparator;

        switch (option.toLowerCase()) {
            case "title" -> comparator = Comparator.comparing(Book::title, String.CASE_INSENSITIVE_ORDER);
            case "author" -> comparator = Comparator.comparing(Book::author, String.CASE_INSENSITIVE_ORDER);
            case "year" -> comparator = Comparator.comparingInt(Book::year);
            default -> {
                System.out.println("Invalid sort option.");
                return;
            }
        }

        if (!ascending) comparator = comparator.reversed();
        list.sort(comparator);
        list.forEach(System.out::println);
    }

    /**
     * Добавя или обновява книга в библиотеката по ISBN.
     *
     * @param book книга, която да добави или обнови.
     */
    public void addBook(Book book) {
        books.put(book.isbn(), book);
    }

    /**
     * Проверява дали библиотеката има книги.
     *
     * @return true, ако има книги.
     */
    public boolean hasBooks() {
        return !books.isEmpty();
    }

    /**
     * Връща всички книги като колекция.
     *
     * @return колекция от всички книги.
     */
    public Collection<Book> getBooks() {
        return books.values();
    }

    /**
     * Търси книги по текст, като гледа в заглавия и автори.
     *
     * @param query търсен текст.
     * @return списък с намерените книги.
     */
    public List<Book> findBooks(String query) {
        List<Book> result = new ArrayList<>();
        String lowerQuery = query.toLowerCase();

        for (Book book : books.values()) {
            if (book.title().toLowerCase().contains(lowerQuery) || book.author().toLowerCase().contains(lowerQuery)) {
                result.add(book);
            }
        }
        return result;
    }

    /**
     * Връща книга по ISBN.
     *
     * @param isbn уникален номер.
     * @return книга или null ако няма такава.
     */
    public Book getBook(int isbn) {
        return books.get(isbn);
    }

    /**
     * Премахва книга по ISBN.
     *
     * @param isbn уникален номер.
     * @return true ако е премахната, false ако не е намерена.
     */
    public boolean removeBook(int isbn) {
        return books.remove(isbn) != null;
    }

    /**
     * Обновява книга в колекцията по ISBN.
     *
     * @param updatedBook книга с нови данни.
     */
    public void updateBook(Book updatedBook) {
        int isbn = updatedBook.isbn();
        if (books.containsKey(isbn)) {
            books.put(isbn, updatedBook);
            System.out.println("Book updated successfully.");
        } else {
            System.out.println("Book not found.");
        }
    }
}import java.io.*;
import java.util.*;

/**
 * Клас FileManager предоставя статични методи за запис и зареждане на данни
 * за потребители и книги чрез сериализация в/от файлове.
 * <p>
 * Използва се в LibraryApp и UserManager/Library за управление на
 * постоянството на данните.
 */
public class FileManager {

    /**
     * Записва карта с потребители в даден файл чрез сериализация.
     *
     * @param users    Карта от потребители, където ключът е потребителското име.
     * @param filename Името на файла, в който ще се запишат данните.
     */
    public static void saveUsers(Map<String, User> users, String filename) {
        try (ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream(filename))) {
            out.writeObject(users);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    /**
     * Записва карта с книги в даден файл чрез сериализация.
     *
     * @param books    Карта от книги, където ключът е ISBN или друг уникален идентификатор.
     * @param filename Името на файла, в който ще се запишат данните.
     */
    public static void saveBooks(Map<String, Book> books, String filename) {
        try (ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream(filename))) {
            out.writeObject(books);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    /**
     * Зарежда карта с потребители от даден файл чрез десериализация.
     *
     * @param filename Името на файла, от който ще се заредят данните.
     * @return Карта с потребители; ако файлът липсва или има грешка, връща празна карта.
     */
    public static Map<String, User> loadUsers(String filename) {
        try (ObjectInputStream in = new ObjectInputStream(new FileInputStream(filename))) {
            return (Map<String, User>) in.readObject();
        } catch (IOException | ClassNotFoundException e) {
            return new HashMap<>();
        }
    }

    /**
     * Зарежда карта с книги от даден файл чрез десериализация.
     *
     * @param filename Името на файла, от който ще се заредят данните.
     * @return Карта с книги; ако файлът липсва или има грешка, връща празна карта.
     */
    public static Map<String, Book> loadBooks(String filename) {
        try (ObjectInputStream in = new ObjectInputStream(new FileInputStream(filename))) {
            return (Map<String, Book>) in.readObject();
        } catch (IOException | ClassNotFoundException e) {
            return new HashMap<>();
        }
    }
}
/**
 * Класът ClientUser представлява клиентски потребител на системата.
 * Наследява класа User и задава нивото на достъп CLIENT по подразбиране.
 * <p>
 * Връзка с LibraryApp и UserManager:
 * - Използва се за създаване на обекти с тип клиент, които нямат администраторски права.
 * - Позволява разграничаване на потребителите по роли.
 */
public class ClientUser extends User {

    /**
     * Конструктор за създаване на нов клиентски потребител.
     * Нивото на достъп винаги се задава като CLIENT.
     *
     * @param username потребителско име
     * @param password парола
     */
    public ClientUser(String username, String password) {
        super(username, password, AccessLevel.CLIENT);
    }
}
import java.io.Serial;
import java.io.Serializable;

/**
 * Класът BookRating представлява оценка на книга, направена от потребител.
 * <p>
 * Връзка с LibraryApp:
 * - Позволява съхранение на оценките, поставени от потребителите за конкретни книги.
 * - Полетата включват уникален идентификатор на книгата, идентификатор на потребителя и самата оценка.
 * - Може да се използва за функционалности като средна оценка, филтриране по рейтинги и др.
 * - Класът е сериализуем, за да може оценките да се запазват и зареждат от файлове.
 */
public class BookRating implements Serializable {
    @Serial
    private static final long serialVersionUID = 1L;

    /** Уникален идентификатор на книгата (например ISBN или друг ключ) */
    final String bookId;

    /** Идентификатор на потребителя, който е поставил оценката */
    String userId;

    /** Оценка, дадена на книгата, обикновено в диапазон от 1 до 5 */
    final int rating;

    /**
     * Конструктор за създаване на нова оценка.
     *
     * @param bookId уникален идентификатор на книгата
     * @param loggedInUser идентификатор на потребителя, който дава оценката
     * @param rating числова стойност на оценката (напр. 1-5)
     */
    public BookRating(String bookId, String loggedInUser, int rating) {
        this.bookId = bookId;
        this.userId = loggedInUser.intern();  // примерно взимаш username
        this.rating = rating;
    }



    /**
     * Връща числовата оценка, зададена на книгата.
     * @return оценката като цяло число
     */
    public int getRating() {
        return rating;
    }

    /**
     * Връща стринг представяне на оценката, подходящо за изход в конзолата.
     *
     * @return текст с информация за оценката на книга от потребител
     */
    @Override
    public String toString() {
        return "BookRating{bookId='" + bookId + "', userId='" + userId + "', rating=" + rating + "}";
    }
}
import java.io.Serial;
import java.io.Serializable;

/**
 * Класът Book е immutable (непроменим) модел, който представя една книга.
 * Използва Java Record, за да дефинира основните характеристики на книгата.
 * <p>
 * Връзка с Library и LibraryApp:
 * <ul>
 *   <li>Обектите Book се съхраняват и управляват в Library.</li>
 *   <li>LibraryApp използва Book при добавяне, търсене и показване на информация.</li>
 * </ul>
 * Полетата включват:
 * <ul>
 *   <li>author - автор на книгата</li>
 *   <li>title - заглавие на книгата</li>
 *   <li>genre - жанр на книгата</li>
 *   <li>year - година на издаване</li>
 *   <li>isbn - уникален ISBN номер на книгата</li>
 * </ul>
 * <p>
 * Класът е сериализуем, за да може да се запазва и зарежда от файлове.
 * Record типът предоставя автоматично getter-и за полетата.
 */
public record Book(String author, String title, String genre, int year, int isbn) implements Serializable {
    @Serial
    private static final long serialVersionUID = 1L;

    /**
     * Връща описателен стринг за книгата, подходящ за показване в конзолата или логове.
     *
     * @return Форматиран стринг с данни за книгата
     */
    @Override
    public String toString() {
        return String.format("Book{title='%s', author='%s', genre='%s', year=%d, isbn=%d}",
                title, author, genre, year, isbn);
    }

    /**
     * Явно getter метод за ISBN.
     * Важно: Record-ите имат автоматични getter-и със същото име като полето (isbn()),
     * но понякога е удобно да има и стандартен get метод за съвместимост с други библиотеки или код.
     *
     * @return уникалния ISBN номер на книгата
     */
    public int getIsbn() {
        return isbn;
    }
}
/**
 * Enum AccessLevel дефинира различните нива на достъп на потребителите в системата.
 * <p>
 * В LibraryApp и UserManager:
 * - Нивата на достъп се използват за разграничаване на администратори и обикновени клиенти.
 * - Администраторите (ADMIN) имат права за добавяне, редактиране и изтриване на книги и потребители.
 * - Клиентите (CLIENT) имат ограничен достъп, например само за четене и оценяване на книги.
 */
public enum AccessLevel {
    /**
     * Администраторски достъп с пълни права в системата.
     */
    ADMIN,

    /**
     * Клиентски достъп с ограничени права.
     */
    CLIENT
}
import java.io.*;
import java.util.HashMap;
import java.util.Map;

/**
 * Класът UserManager управлява потребителите в системата.
 * Отговаря за регистрация, логин, логаут и проверка на права.
 * Потребителите се съхраняват в Map с ключ - username и стойност - User обект.
 * Логнатият потребител се пази в поле loggedInUser.
 */
public class UserManager implements Serializable {
    @Serial
    private static final long serialVersionUID = 1L;

    /** Карта с всички потребители в системата */
    private final Map<String, User> users = new HashMap<>();

    /** Текущо логнат потребител */
    private User loggedInUser;

    /**
     * Конструктор - създава администратор с потребителско име "admin" и парола "i<2Java"
     * Добавя го в потребителската карта.
     */
    public UserManager() {
        users.put("admin", new User("admin", "i<2Java", AccessLevel.ADMIN));
    }

    /**
     * Метод за логин на потребител по потребителско име и парола.
     * Ако има съвпадение, задава го като текущо логнат потребител.
     *
     * @param username Потребителско име
     * @param password Парола
     * @return true ако логина е успешен, false ако не
     */
    public boolean login(String username, String password) {
        if (loggedInUser != null) {
            System.out.println("Вече има логнат потребител: " + loggedInUser.getUsername());
            return false; // вече има логнат
        }

        User user = users.get(username);
        if (user != null && user.getPassword().equals(password)) {
            loggedInUser = user;
            System.out.println("Успешен вход: " + username);
            return true;
        } else {
            System.out.println("Грешно потребителско име или парола.");
            return false;
        }
    }

    /**
     * Метод за логаут на текущо логнат потребител.
     * Ако няма логнат потребител, отпечатва съобщение.
     */
    public void logout() {
        if (loggedInUser != null) {
            System.out.println("Потребител " + loggedInUser.getUsername() + " излезе.");
            loggedInUser = null;
        } else {
            System.out.println("Няма логнат потребител.");
        }
    }

    /**
     * Връща текущо логнатия потребител или null ако няма такъв.
     *
     * @return User текущо логнат потребител
     */
    public User getLoggedInUser() {
        return loggedInUser;
    }

    /**
     * Проверява дали текущо логнатия потребител е администратор.
     *
     * @return true ако има логнат потребител и той е админ, иначе false
     */
    public boolean isAdmin() {
        return loggedInUser != null && loggedInUser.isAdmin();
    }

    /**
     * Добавя нов потребител с ниво CLIENT.
     * Ако потребителят вече съществува, отпечатва съобщение.
     *
     * @param username Потребителско име
     * @param password Парола
     */
    public void addUser(String username, String password) {
        if (users.containsKey(username)) {
            System.out.println("Потребителят вече съществува.");
        } else {
            users.put(username, new User(username, password, AccessLevel.CLIENT));
            System.out.println("Потребителят е добавен успешно.");
        }
    }

    /**
     * Премахва потребител по потребителско име.
     * Ако потребителят не съществува, отпечатва съобщение.
     *
     * @param username Потребителско име
     */
    public void removeUser(String username) {
        if (users.containsKey(username)) {
            users.remove(username);
            System.out.println("Потребителят е премахнат успешно.");
        } else {
            System.out.println("Потребителят не е намерен.");
        }
    }

    /**
     * Извежда списък с всички потребители.
     * Показва само потребителското име и нивото на достъп.
     */
    public void listUsers() {
        if (users.isEmpty()) {
            System.out.println("Няма регистрирани потребители.");
        } else {
            System.out.println("Списък с потребители:");
            for (User user : users.values()) {
                System.out.println(user);
            }
        }
    }

    /**
     * Записва потребителите във файл.
     *
     * @param fileName Името на файла за запис
     */
    public void saveUsers(String fileName) {
        try (ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream(fileName))) {
            oos.writeObject(users);
            System.out.println("Потребителите са записани успешно.");
        } catch (IOException e) {
            System.out.println("Грешка при запис на потребителите: " + e.getMessage());
        }
    }

    /**
     * Зарежда потребителите от файл.
     *
     * @param fileName Името на файла за зареждане
     */
    @SuppressWarnings("unchecked")
    public void loadUsers(String fileName) {
        try (ObjectInputStream ois = new ObjectInputStream(new FileInputStream(fileName))) {
            Object obj = ois.readObject();
            if (obj instanceof Map) {
                users.clear();
                users.putAll((Map<String, User>) obj);
                System.out.println("Потребителите са заредени успешно.");
            }
        } catch (FileNotFoundException e) {
            System.out.println("Файлът за потребители не е намерен.");
        } catch (IOException | ClassNotFoundException e) {
            System.out.println("Грешка при зареждане на потребителите: " + e.getMessage());
        }
    }

    /**
     * Проверява дали има логнат потребител.
     *
     * @return true ако има логнат потребител, иначе false
     */
    public boolean isLoggedIn() {
        return loggedInUser != null;
    }
}

