# Online Book Store

## Requirements
1. User should be able to search books by title, author, genre.
2. Each book should have a unique identification number.
3. There could be multiple copies of the same book. lets call it BookItem which we will need to keep track of.
4. System should have the ability to tell who took a particular book, when its due for return
5. It should also have the ability to tell a particular user has how many books.
6. Limit on max books and max days for each book.
7. User can have the ability to reserve books that aren't available and notify them when available.
8. Ability to take payments and process fines.

## Workflow
1. User logs in and searches a book.
2. User finds the book he is looking for
3. They order the book for N days
4. The make the payment and get and receipt
5. They can access the book.
6. User doesn't find the book as all copies of it are out.
7. User reserves the book and will get notified when someone returns it based on FCFS.



Note: This is to lease books to read online. If the question is to buy physical copy
you'll replace `BookLeasing` by `BookSell` and add basically follow online shopping strategy.

## Classes and methods

### Book.java
```java
  private String ISBN;
  private String title;
  private String subject;
  private String publisher;
  private String language;
  private int numberOfPages;
  private List<Author> authors;

```

### BookItem.java
One unit of a Book is a BookItem.
```java
  private String barcode;
  private boolean isReferenceOnly;
  private Date borrowed;
  private Date dueDate;
  private double price;
  private BookStatus status;

  public boolean checkout(String memberId) {...}
```

### User.java
```java
public class User {
  private String name;
  private Address address;
  private String email;
  private String phone;
}
```
### Account.java
```java
public abstract class Account {
  private String accountId;
  private String password;
  private AccountStatus status; //Prime, Not a prime, 
  private User user;

  public boolean resetPassword() { ... };
  //total number of books that the account current has leased; 
  public int getTotalBooksCheckedout() {...};
  public void setTotalBooksCheckedout() {...};
  
  //account reserving books that are currently out of stock.
  public boolean reserveBookItem(BookItem bookItem) {...};
  //account reserving books that are currently out of stock.
  public boolean returnBookIten(BookItem bookItem) {...};
  //account reserving books that are currently out of stock.
  public boolean renewBookItem(BookItem bookItem) {...};
  //check for fine.
  void checkForFine(String bookItemBarcode) { ... }
}
```

### BookReservation.java
```java
public class BookReservation {  
  private Date creationDate;
  private ReservationStatus status;
  private String bookItemBarcode;
  private String memberId;

  public static BookReservation fetchReservationDetails(String barcode) {...};
}

```
### BookLeasing.java
```java
public class BookLeasing {
  private Date creationDate;
  private Date dueDate;
  private Date returnDate;
  private String bookItemBarcode;
  private String accountId;
  private boolean isAvailable;

  // lend the book to a particular accountId
  public static void lendBook(String barcode, String accountId) { ... };
  public static BookLeasing fetchLeasingDetails(String barcode);
 //check if a particular book that is selected is available.
  public boolean isBookAvailableForLease(String barcode);
}

```

### Catalog.java
This class will be used by the user to search respective books based on title and author or genre.
```java
  private HashMap<String, List<Book>> bookTitles;
  private HashMap<String, List<Book>> bookAuthors;
  private HashMap<String, List<Book>> bookGenre;
  private HashMap<String, List<Book>> bookPublicationDates;

  public List<Book> searchByTitle(String query) {
    // return all books containing the string query in their title.
    return bookTitles.get(query);
  }

  public List<Book> searchByAuthor(String query) {
    // return all books containing the string query in their author's name.
    return bookAuthors.get(query);
  }

  public List<Book> searchByGenre(String query) {
    // return all books containing the string query in their author's name.
    return bookGenre.get(query);
  }
  //update the catalog for a particular book if it is leased.
  public int updateCatalog(Book book) {...};
  
  //notify all the account if a particular book become available.
  public List<AccountId> notifyAccounts() { ... };
  
```

### Fine.java
```java
  private Date creationDate;
  private double bookItemBarcode;
  private String memberId;

  public static void collectFine(String accountId, long days) { ... }
```
### Payment.java
```java
public class Payment {
	String paymentId;
	PaymentMode paymentMode;
	PaymentStatus paymentStatus;

	boolean makeTransaction() {...}
}
```
### Notification.java
```java
public class Notification {
	String notificationId;
	NotificationMode notificationMode;
	NotificationStatus notificationStatus;
}
```

we will use enum class for all the Status like `PaymentStatus` , `NotificationStatus`, `BookStatus` etc.

