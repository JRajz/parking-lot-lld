# Parking lot lld

Low-level architecture for a backend system of a smart parking lot, handles vehicle entry and exit management, parking space allocation, and fee calculation.

## Design Aspects

**1. Data Model**

* **Purpose:** Efficiently store, retrieve, and manage parking data.

### Classes and Relationships

***Key Classes:**

* **ParkingSpot:**
  * Represents a single parking space.
  * Attributes: ID, Floor, Location, Size, Availability, OccupiedBy, Version.
  * Methods: isSuitableFor(), markOccupied(), markVacant().
* **Vehicle:**
  * Represents a vehicle using the parking lot.
  * Attributes: ID, Type, LicensePlate, CurrentTransaction, ParkingFloorPreference.
  * Methods: getTypeMultiplier() (optional).
* **ParkingTransaction:**
  * Represents a parking session for a vehicle.
  * Attributes: ID, Vehicle, EntryTime, ExitTime, Duration, Fee, PaymentStatus, Version.
  * Methods: calculateFee(), markPaid().
* **SpotAllocator:**
  * Allocates available parking spots to vehicles.
  * Methods: getAvailableSpots(), assignSpot().
* **ParkingManager:**
  * Manages overall parking operations.
  * Methods: registerVehicleEntry(), registerVehicleExit(), updateAvailability(), getParkingStats().

**Relationships:**

* One-to-one between Vehicle and ParkingTransaction (a vehicle has at most one active transaction).
* Many-to-one between ParkingTransaction and ParkingSpot (multiple transactions can use a spot over time).
* One-to-one between ParkingSpot and Vehicle (if occupied) (a spot can be occupied by only one vehicle at a time).

### Diagrams

![Table Diagram](tables.png)

![UML Diagram](uml-diagram.png)

**2. Algorithm for Spot Allocation**

* **Purpose:** Optimize spot assignment for efficient space utilization and user convenience.

**Steps:**

* **Retrieve available spots:** Query the ParkingSpot table for spots with Availability = True and Size matching the vehicle's Type.

* **Prioritize based on preferences:**  If available, prioritize spots matching the user's ParkingFloorPreference.

* **Apply additional criteria (optional):** Consider factors like proximity to exits, distance from other parked vehicles, or real-time occupancy patterns.

* **Assign spot:** Select the most suitable spot and mark it as occupied.


**3. Fee Calculation Logic**

* **Purpose:** Determine accurate parking fees based on duration and vehicle type.

**Formula**

  ```
    Fee = BaseFee + (Duration * RateMultiplier * VehicleTypeMultiplier)
   ```
* **BaseFee:** Fixed parking fee.
* **Duration:** Parking duration in hours or minutes.
* **RateMultiplier:** Hourly or minutely rate based on time of day, day of the week, or other factors.
* **VehicleTypeMultiplier:** Fee multiplier based on vehicle type (e.g., larger vehicles might have higher fees).


**4. Concurrency Handling**

* **Purpose:** Prevent data inconsistencies and errors with multiple users.

    **Techniques:**

    * **Optimistic locking:** Use versioning to detect and handle conflicts when multiple processes try to update the same data simultaneously.
    * **Transactions:** Ensure database operations are atomic and consistent, preventing partial updates in case of errors.
    * **Queues:** Consider using queues to manage incoming requests and process them sequentially, avoiding race conditions.
