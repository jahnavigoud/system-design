
# Airline Management System

## Requirements 
### As a Customer
1. customer should be able to search flights
2. customer should be able to book , cancel or modify a reservation
3. Customer should be able to select a seat.
4. make a payment

### As an Admin
1. Change the flight schedule / itinerary
2. assign pilot and crew.
3. cancel a flight.

### As a front desk officer
1. Should be able to book , cancel or modify a reservation.


## Workflow
1. Customer logs in
2. Enter details like source, destination, date, time 
3. Gets list of flights
4. Selects a flight
5. Selects a seat
6. Makes reservation
7. Does payment
8. Gets notification.

## Classes and methods

### Airport.java
```java
public class Airport {
	
	String airportName; // for eg: SeaTac, Sky Harbor, Charles Douglas;
	String address;
	String uniqueAirportId; // for each airport

}
```
### Airline.java
```java
public class Airline {
	String airlineName; //Alaska, American, Delta
	String airlineCode; //1,2,3
}
```

### Aircraft.java
```java
public class Aircraft {
	String manufacturer; //boeing, airbus
	String model; //737, A300
	String yearsInService;
	String lastService;
	String manufacturingDate;

}
```

### Flight.java
```java
public class Flight {
	String flightNumber;
	String source;
	String destination;
	String departureTime;
	String expectedArrivalTime;
	Aircraft assignedAircraft; //each flight will be an instance so a flight from seattle to london at a paricular time is 1 instance of flight
	Airport airport;

	boolean addFlightSchedule() {...} //information like source destination date time etc.
	void updateStatus(FlightStatus flightStatus) {...} //update the status of the flight
	boolean cancel() {...} // cancel the flight.

}
```
### FlightReservation.java
To pull out the reservation of the entire flight
```java
/*To pull out the reservation of the entire flight*/
public class FlightReservation {

		Flight flight;
		String reservationNumber;
		Map<Passenger, Seat> seatMap;
		ReservationStatus reservationStatus;

		public List<Passenger> getPassengers() {...}
		public FlightReservation getFlightReservation(String reservationNumber) {...}
	
}
```
### Seat.java
Class to assign seats to passengers for a particular flight instance.
```java
/*Class to assign seats to passengers for a particular flight instance.*/
public class Seat {
	String seatFare;
	String seatType;
	String seatNumber;

	public float getFare() {...}
}
```
### Itinerary.java
Both the customer and front-desk can use the itinerary class to book/ cancel/ update the reservation.

```java
/*
Both the customer and front-desk can use the itinerary class to book/ cancel/ update the reservation.	
*/
public class Itinerary {
	String customerId;
  	Airport startingAirport;
  	Airport finalAirport;
  	Date creationDate;
  	List<FlightReservation> reservations;

  	public List<FlightReservation> getReservations();
  	public boolean makeReservation();
  	public boolean makePayment();
	
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

### Notification.java 
```java
public class Notification {
	String notificationId;
	NotificationMode notificationMode;
	NotificationStatus notificationStatus;
}
```

we will use enum class for all the Status like `PaymentStatus` , `NotificationStatus`, `ReservationStatus` etc.

