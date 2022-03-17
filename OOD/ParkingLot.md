
# Design a Parking lot system

## Requirements 
* Parking lot has multiple levels. Each level has multiple rows of spots.
* Motorcycles, cars and buses
* parking lot has compact slot, large slot and motorcycle slot
* motorcycle can park in any slot
* car can park in single compact spot or single large spot
* bus can park in 5 consecutive large spots. can't park in small spots

## Classes and Methods.

#### VehicleSize.java 
This class is a simple enum class for different vehicle size.
```java
public enum VehicleSize { Motorcycle, Large, Compact}
```

#### Vehicle.java
Abstract class that coulbe extended for different vehicles in the parking lot.

```java
public abstract class Vehicle {
	
	List<ParkingSpot> parkingSpots = new ArrayList<>();
	String licensePlate;
	VehicleSize sizeOfVehicle;
	int spotsRequired;

	// park the vehicle in a paricular spot.
	public void parkInSpot(ParkingSpot parkingSpot) { parkingSpots.add(parkingSpot); }

	//clear the spot 
	public void removeSpot() { ... }

	// can fit in a spot ?
	public boolean canFitInParkingSpot(ParkingSpot parkingSpot);

}
```

#### Motorcycle.java
Extends the vehicle abstract class.

```java
public Motorcycle extends Vehicle {
	
	Motorcycle(){
		spotsRequired=1;
		sizeOfVehicle = VehicleSize.MotorCycle;
	}

	// can a motorcycle fit in the slot 
	public boolean canFitInParkingSpot(ParkingSpot parkingSpot) { ... }

}
```

#### Car.java
Extends the vehicle abstract class.

```java
public Car extends Vehicle {
	
	Car(){
		spotsRequired=1;
		sizeOfVehicle = VehicleSize.Compact;
	}

	// can a car fit in the slot 
	public boolean canFitInParkingSpot(ParkingSpot parkingSpot) { ... }

}
```

#### Bus.java
Extends the vehicle abstract class.

```java
public Bus extends Vehicle {
	
	Bus(){
		spotsRequired=5;
		sizeOfVehicle = VehicleSize.Large;

	}

	// can a bus fit in the slot 
	public boolean canFitInParkingSpot(ParkingSpot parkingSpot) { ... }

}
```

#### ParkingSlot.java
The ParkingSlot is used to keep a track of all slot information.

```java
class ParkingSpot {

	private Level level;
	private int row;
	private int spot;
	VehicleSize vehicleSize;
	private Vehicle vehicle;

	ParkingSpot(Level level, int spot, int row, Vehicle vehicle){}

	//getters and some util methods.

	
}
```

#### ParkingLot.java
ParkingLot is the class that is where the main control lies or where the client may call
this class to park the vehicle.

```java
class ParkingLot {
	
	//total levels
	private final int NUM_LEVELS = 10;
	private Level[] levels; 


	public void park(Vehicle vehicle) {

		Steps
		1. Iterate each level
		2. at each level if there is a free ParkingSlot check if the vehicle fits
		3. If yes park else keep iterating and keep moving up level
		4. If parking lot is full throw an error saying parking lot full

	}

}
```

#### Level.java
Storing all the level information required by the parking lot.

```java
class Level {


	//Remove a vehicle from a spot
	public void freeSpot() {...}

	//Park a vehicle starting at the spot spotNumber
	public boolean parkStartingAtSpot(int spotNumber, Vehicle v) { ... }

	// Find a spot to park the vehicle. if not possible return -1.
	public int findAvaialableSpot(Vehicle vehicle) { ... }
	
}
```