# Amazon Locker Service
You need to make sure the locker size > package.

## Requirements.
1. User should have the ability to select a locker close to their area.
2. User should have this option while checking out a particular order.
3. Once the order is placed and expected ETA is known we need a way to estimate the amount of 
locker space that will be required for each package (Small / Medium / Large) lets call this lockerSlot.
4. The deliveryMan should be able to open a particular lockerSlot and put the package in the slot.
(sake of simplicity we assume a 1-to-1 mapping of package and slot)
5. Once the package is inside the lockerSlot he generates and sends a unique OTP valid for 3 days.
6. If the package isn't retrieved in 3 days the package is refunded.



## Workflow
1. User selects locker service
2. Gets notified if the locker would be available for him on the selected date or give him option
of selecting new date or change delivery type.
3. DeliveryMan opens the slot put the package and sends OTP to the user.
4. User retrieves the package.
==
Not so happy path
==
If the chosen locker service is full then notify the user about the earliest delivery date/time 
based on slot availability. Allow user to change the delivery type or accept the new ETA.


## Classes and methods.

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
public class Account {
  private String accountId;
  private String password;
  private AccountStatus status; //Prime, Not a prime, 
  private User user;

  public boolean resetPassword() { ... };
  public boolean packageRetrieved(PackageId packageId) {...};
  public boolean packageRefunded(PackageId packageId, Date date){ ... };
  public Locker searchForLockerNearYou(String address, Date date){ ... };
  public DeliveryStatus getNotifications(){...};
  public boolean updateDeliveryMode(Delivery delivery) {...};
  public boolean updateDeliveryDateForParticularPackage(Delivery delivery, Date date) {...};
}
```
### Package.java
```java
public class Package {
  String packageId;
  int height;
  int weight;
  int width;
  int breadth;
 
  public LockerSize findLockerSizeForPackage(String packageId, int height, int width, int breadth){...}
}
```
### Order.java
```java
public class Order {
  String OrderId;
  Date orderCreationDate;
  List<Packages> listOfPackages;
  
  public void updateOrder(){...};
  public void cancelOrder(){...};
}
```

### Locker.java
```java
public class Locker {
  
  private LockerId lockerId;
  private LockerSize lockerSize;
  private String address;
  private int totalCapacity;
}
```

### LockerSlot.java
```java
public class LockerSlot {
  private List<LockerSlot> listOfSlots;
  private String lockerSlotId;
  private LockerId lockerId;
  private Date lastOccupied;
  private LockerStatus lockerStatus;

  public boolean isOccupied() {...};
  //total available locker slots for a particular date
  public List<LockerSlot> getTotalAvailableLockerSlot(LockerId lockerId, Date date) { ... }
}
```

### Delivery.java
```java
public class Delivery {
  private DeliveryStatus deliveryStatus;
  private DeliveryMode deliveryMode;
  private Date expectedDeliveryDate;
  private DeliveryVehicle deliveryVehicle
  
  // ability to send OTP
  public void sendOTP() {...}
  
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
### Payment.java
```java
public class Payment {
	String paymentId;
	PaymentMode paymentMode;
	PaymentStatus paymentStatus;

	boolean makeTransaction() {...}
}
```

we will use enum class for all the Status like `PaymentStatus` , `NotificationStatus`, `DeliveryStatus` , `LockerStatus` etc.

