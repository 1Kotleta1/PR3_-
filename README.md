import 'dart:io';

class Book {
  String title;
  String author;
  String isbn;
  bool isAvailable = true;

  Book(this.title, this.author, this.isbn);
}

class Member {
  String fullName;
  String email;
  String phoneNumber;
  String username;
  String password;
  bool isAdmin;

  Member(this.fullName, this.email, this.phoneNumber, this.username, this.password, {this.isAdmin = false});
}

class Loan {
  Member member;
  Book book;
  DateTime loanDate;
  DateTime? returnDate;

  Loan(this.member, this.book, this.loanDate);
}

class Library {
  List<Book> books = [];
  List<Member> members = [];
  List<Loan> loans = [];

  void addBook(Book book) {
    books.add(book);
    print("Книга '${book.title}' добавлена в библиотеку.");
  }

  void addMember(Member member) {
    members.add(member);
    print("Член библиотеки '${member.fullName}' добавлен.");
  }

  Member? authenticateUser() {
    print("Введите логин: ");
    String username = stdin.readLineSync()!;
    
    if (username == "admin") {
      print("Вход выполнен как администратор.");
      return Member("Admin", "admin@mail.com", "1234567890", "admin", "", isAdmin: true);
    }
    
    print("Введите пароль: ");
    String password = stdin.readLineSync()!;

    for (var member in members) {
      if (member.username == username && member.password == password) {
        print("Вход выполнен успешно.");
        return member;
      }
    }
    print("Ошибка: Неверный логин или пароль.");
    return null;
  }

  void issueBook(Member member) {
    if (books.isEmpty) {
      print("Ошибка: В библиотеке нет книг.");
      return;
    }
    
    while (true) {
      print("Выберите книгу из списка:");
      for (int i = 0; i < books.length; i++) {
        print("${i + 1}. ${books[i].title} (${books[i].isAvailable ? 'Доступна' : 'Выдана'})");
      }

      print("Введите номер книги: ");
      int? choice = int.tryParse(stdin.readLineSync()!);
      
      if (choice != null && choice >= 1 && choice <= books.length) {
        Book book = books[choice - 1];
        if (!book.isAvailable) {
          print("Ошибка: Книга уже выдана.");
        } else {
          book.isAvailable = false;
          loans.add(Loan(member, book, DateTime.now()));
          print("Книга '${book.title}' выдана ${member.fullName}.");
          break;
        }
      } else {
        print("Ошибка: Неверный ввод.");
      }
    }
  }

  void returnBook(Member member) {
    List<Loan> memberLoans = loans.where((loan) => loan.member == member && loan.returnDate == null).toList();
    if (memberLoans.isEmpty) {
      print("У вас нет выданных книг.");
      return;
    }

    print("Ваши выданные книги:");
    for (int i = 0; i < memberLoans.length; i++) {
      print("${i + 1}. ${memberLoans[i].book.title}");
    }

    print("Введите номер книги для возврата: ");
    int? choice = int.tryParse(stdin.readLineSync()!);
    if (choice != null && choice >= 1 && choice <= memberLoans.length) {
      Loan loan = memberLoans[choice - 1];
      loan.returnDate = DateTime.now();
      loan.book.isAvailable = true;
      print("Книга '${loan.book.title}' успешно возвращена.");
    } else {
      print("Ошибка: Неверный ввод.");
    }
  }

  void listBooks() {
    print("Список всех книг в библиотеке:");
    for (var book in books) {
      print("${book.title} - ${book.author} (${book.isAvailable ? 'Доступна' : 'Выдана'})");
    }
  }

  void adminAddBook() {
    print("Введите название книги: ");
    String title = stdin.readLineSync()!;
    print("Введите автора: ");
    String author = stdin.readLineSync()!;
    print("Введите код книги: ");
    String isbn = stdin.readLineSync()!;
    addBook(Book(title, author, isbn));
  }
}

void main() {
  var library = Library();

  while (true) {
    print("1. Зарегистрироваться\n2. Войти в систему\n3. Выйти");
    String? choice = stdin.readLineSync();
    if (choice == "1") {
      print("Введите ФИО: ");
      String fullName = stdin.readLineSync()!;
      print("Введите email: ");
      String email = stdin.readLineSync()!;
      print("Введите номер телефона: ");
      String phoneNumber = stdin.readLineSync()!;
      print("Придумайте логин: ");
      String username = stdin.readLineSync()!;
      print("Придумайте пароль: ");
      String password = stdin.readLineSync()!;
      library.addMember(Member(fullName, email, phoneNumber, username, password));
    } else if (choice == "2") {
      var user = library.authenticateUser();
      if (user != null) {
        while (true) {
          print("Выберите действие:\n1. Взять книгу\n2. Вернуть книгу\n3. Просмотреть список книг\n${user.isAdmin ? '4. Добавить книгу\n' : ''}5. Выйти");
          String? action = stdin.readLineSync();
          if (action == "1") {
            library.issueBook(user);
          } else if (action == "2") {
            library.returnBook(user);
          } else if (action == "3") {
            library.listBooks();
          } else if (user.isAdmin && action == "4") {
            library.adminAddBook();
          } else if (action == "5") {
            break;
          } else {
            print("Ошибка: Неверный ввод.");
          }
        }
      }
    } else if (choice == "3") {
      break;
    } else {
      print("Ошибка: Неверный ввод.");
    }
  }
}
