# -Smart-Parking-Lot-System-LLD-
## Problem Statement
Imagine a parking lot in an urban area with multiple floors and numerous parking spots. Your task is to create a low-level design for a system that efficiently manages the parking process. The system should automatically assign parking spots based on vehicle size and availability, track the time each vehicle spends in the parking lot, and calculate parking fees upon exit.

## Functional Requirements
- **Parking Spot Allocation**: Automatically assign an available parking spot to a vehicle when it enters, based on the vehicle’s size (e.g., motorcycle, car, bus).
- **Check-In and Check-Out**: Record the entry and exit times of vehicles.
- **Parking Fee Calculation**: Calculate fees based on the duration of stay and vehicle type.
- **Real-Time Availability Update**: Update the availability of parking spots in real-time as vehicles enter and leave.

## Design Aspects to Consider
- **Data Model**: Design a database schema to manage parking spots, vehicles, and parking transactions.
- **Algorithm for Spot Allocation**: Develop an algorithm to efficiently assign parking spots to incoming vehicles.
- **Fee Calculation Logic**: Implement logic to calculate fees based on parking duration and vehicle type.
- **Concurrency Handling**: Ensure the system can handle multiple vehicles entering or exiting simultaneously.

## 1. Data Model

### Tables and Entities

```java
public class ParkingSpot {
    private int id;
    private VehicleSize size;
    private boolean isOccupied;
    private int floor;
    // Getters and Setters
}

public class Vehicle {
    private String licensePlate;
    private VehicleSize size;
    private LocalDateTime entryTime;
    private LocalDateTime exitTime;
    // Getters and Setters
}

public class ParkingTransaction {
    private int id;
    private String vehicleLicensePlate;
    private int parkingSpotId;
    private LocalDateTime entryTime;
    private LocalDateTime exitTime;
    private BigDecimal fee;
    // Getters and Setters
}

public enum VehicleSize {
    MOTORCYCLE, CAR, BUS
}
```

## 2. Algorithm for Spot Allocation

## Vehicle Entry:

- Determine the size of the vehicle.
- Query available parking spots of the required size.
- Allocate the nearest available spot (could be based on floor or distance logic).
- Mark the spot as occupied.
Record entry time.

## Vehicle Exit:

- Retrieve the vehicle's entry record.
- Calculate the parking fee based on entry and exit times.
- Mark the parking spot as available.
- Update the exit time and fee in the transaction record.

## Psuedo Code
```java
public class ParkingService {

    public ParkingSpot allocateParkingSpot(Vehicle vehicle) {
        ParkingSpot spot = findAvailableSpot(vehicle.getSize());
        if (spot != null) {
            spot.setOccupied(true);
            ParkingTransaction transaction = new ParkingTransaction();
            transaction.setVehicleLicensePlate(vehicle.getLicensePlate());
            transaction.setParkingSpotId(spot.getId());
            transaction.setEntryTime(LocalDateTime.now());
            saveTransaction(transaction);
            saveParkingSpot(spot);
            return spot;
        }
        return null;
    }

    public void releaseParkingSpot(String vehicleLicensePlate) {
        ParkingTransaction transaction = getActiveTransaction(vehicleLicensePlate);
        if (transaction != null) {
            transaction.setExitTime(LocalDateTime.now());
            BigDecimal fee = calculateFee(transaction.getEntryTime(), transaction.getExitTime(), getVehicleSize(vehicleLicensePlate));
            transaction.setFee(fee);
            ParkingSpot spot = getParkingSpot(transaction.getParkingSpotId());
            spot.setOccupied(false);
            saveTransaction(transaction);
            saveParkingSpot(spot);
        }
    }

    private ParkingSpot findAvailableSpot(VehicleSize size) {
        // Logic to find and return an available parking spot of the given size
    }

    private void saveTransaction(ParkingTransaction transaction) {
        // Logic to save the transaction record to the database
    }

    private void saveParkingSpot(ParkingSpot spot) {
        // Logic to save the parking spot record to the database
    }

}

```
## 3.Fee Calculation Logic
```java
public class FeeCalculator {

    public BigDecimal calculateFee(LocalDateTime entryTime, LocalDateTime exitTime, VehicleSize vehicleSize) {
        Duration duration = Duration.between(entryTime, exitTime);
        long hours = duration.toHours();
        BigDecimal fee = BigDecimal.ZERO;

        switch (vehicleSize) {
            case MOTORCYCLE:
                fee = BigDecimal.valueOf(5).multiply(BigDecimal.valueOf(hours));
                break;
            case CAR:
                fee = BigDecimal.valueOf(10).multiply(BigDecimal.valueOf(hours));
                break;
            case BUS:
                fee = BigDecimal.valueOf(20).multiply(BigDecimal.valueOf(hours));
                break;
        }
        return fee;
    }
}
```
## 4. Concurrency Handling
-> Strategies
- Database Locking: Use row-level locking in the database to prevent race conditions.
- Optimistic Locking: Implement versioning on records to ensure consistency.
- Concurrency Control: Use transactions to manage concurrent access to shared data.
```java
public class ParkingLotSystem {

    private ParkingService parkingService;
    private FeeCalculator feeCalculator;

    public ParkingLotSystem() {
        this.parkingService = new ParkingService();
        this.feeCalculator = new FeeCalculator();
    }

    public void vehicleEntry(String licensePlate, VehicleSize size) {
        Vehicle vehicle = new Vehicle();
        vehicle.setLicensePlate(licensePlate);
        vehicle.setSize(size);
        vehicle.setEntryTime(LocalDateTime.now());

        ParkingSpot spot = parkingService.allocateParkingSpot(vehicle);
        if (spot != null) {
            System.out.println("Vehicle " + licensePlate + " parked at spot " + spot.getId());
        } else {
            System.out.println("No available spot for vehicle " + licensePlate);
        }
    }

    public void vehicleExit(String licensePlate) {
        parkingService.releaseParkingSpot(licensePlate);
        System.out.println("Vehicle " + licensePlate + " has exited.");
    }
}

```
