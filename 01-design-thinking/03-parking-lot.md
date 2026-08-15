# Session 3 — Parking Lot LLD

## Learning approach

We learned LLD by starting with simple requirements and allowing the design to evolve as new requirements appeared.

```text
Requirement
→ First thought
→ Responsibility
→ Design
→ Requirement change
→ Improve design
→ Learn concept
```

We intentionally avoided adding unnecessary patterns or abstractions too early.

---

## 1. ParkingLot

The main system responsible for coordinating parking operations is `ParkingLot`.

```text
ParkingLot
    |
    +-- ParkingManager
    +-- TicketManager
    +-- PriceCalculator
```

### Concept: Coordinator

`ParkingLot` coordinates the overall operation. It does not perform every operation itself.

---

## 2. Vehicle

A vehicle has:

```text
registrationNumber
type
```

Vehicle types:

```text
BIKE
CAR
TRUCK
```

Example:

```ts
class Vehicle {
    constructor(
        public registrationNumber: string,
        public type: VehicleType
    ) {}
}
```

---

## 3. ParkingSpot

A parking spot has:

```text
type
status
```

For example:

```text
ParkingSpot
├── type: BIKE / CAR / TRUCK
└── status: EMPTY / OCCUPIED
```

`ParkingManager` will validate whether a vehicle can use a particular spot.

---

## 4. ParkingFloor

A parking floor contains parking spots.

```text
ParkingFloor
├── floorNumber
└── parkingSpots
```

We decided **not** to restrict a floor to one vehicle type unless the requirement asks for it.

A floor can contain:

```text
Bike spots
Car spots
Truck spots
```

---

## 5. ParkingManager

We introduced `ParkingManager`, similar to `InventoryManager` from the Coffee Machine design.

### Responsibility

`ParkingManager` manages parking-related state and rules.

Potential responsibilities:

```text
findAvailableSpot()
validateSpot()
bookSpot()
freeSpot()
```

It also manages the parking floors:

```text
ParkingManager
    |
    ├── Floor 1
    │     ├── Spot 1
    │     ├── Spot 2
    │     └── ...
    │
    ├── Floor 2
    │     └── ...
    │
    └── Floor 3
          └── ...
```

Conceptually:

```ts
class ParkingManager {
    constructor(
        private floors: ParkingFloor[]
    ) {}
}
```

The exact implementation of `findAvailableSpot()` was not finalized.

---

## 6. Parking entry flow

When a vehicle arrives, the system needs:

```text
Vehicle
+
Entry Time
```

Flow:

```text
Vehicle arrives
      ↓
ParkingLot
      ↓
ParkingManager
      ↓
Find available spot
      ↓
Validate vehicle ↔ spot
      ↓
Book spot
      ↓
ParkingLot
      ↓
Generate Ticket
```

---

## 7. Ticket

A ticket represents the parking transaction.

It has a unique ticket number because the ticket number is used when the vehicle exits.

Current information:

```text
ticketNumber
vehicle
entryTime
floorNumber
spotNumber
```

Conceptually:

```ts
type Ticket = {
    ticketNumber: string;
    vehicle: Vehicle;
    entryTime: Date;
    floorNumber: number;
    spotNumber: number;
};
```

---

## 8. Why ticket number?

When the vehicle exits, the customer provides the ticket number.

The system uses it to find:

```text
floor number
spot number
```

Then the correct spot can be freed.

```text
Ticket Number
      ↓
ParkingLot
      ↓
Find Ticket
      ↓
Get Floor + Spot
      ↓
ParkingManager.freeSpot()
```

---

## 9. TicketManager

`TicketManager` manages the ticket lifecycle.

Responsibilities:

```text
createTicket()
exitTicket()
```

`TicketManager` manages tickets, not parking spots.

Parking spot state remains the responsibility of `ParkingManager`.

---

## 10. Pricing requirement

When a vehicle exits, the system needs to calculate the parking fee.

Example:

```text
Entry: 10:00 AM
Exit: 1:30 PM
Duration: 3.5 hours
```

We realized pricing rules can vary:

```text
Floor-wise
Day-wise
Time-wise
Morning/evening-wise
etc.
```

Therefore pricing should be separated from ticket management.

---

## 11. PriceCalculator

We introduced an abstraction:

```ts
interface PriceCalculator {
    calculate(
        ticket: Ticket,
        exitTime: Date
    ): number;
}
```

Different implementations can provide different pricing rules:

```text
PriceCalculator
       ↑
       |
       +-- FloorWisePriceCalculator
       +-- DayWisePriceCalculator
       +-- TimeBasedPriceCalculator
```

### Concept: Abstraction

The rest of the system depends on the pricing contract rather than a specific pricing algorithm.

This allows pricing rules to change independently.

---

## 12. Parking exit flow

`ParkingLot` coordinates the complete exit operation.

```text
Ticket Number
      ↓
ParkingLot
      ↓
TicketManager
      ↓
Get ticket
      ↓
PriceCalculator
      ↓
Calculate fee
      ↓
TicketManager marks ticket exited
      ↓
ParkingManager.freeSpot()
      ↓
ParkingLot returns fee
```

---

## 13. Important responsibility distinction

We explicitly established:

> `ParkingLot` coordinates the operation, but does not perform every operation itself.

For example:

```text
ParkingLot
→ "Calculate the fee."

PriceCalculator
→ knows HOW to calculate it.
```

```text
ParkingLot
→ "Free this spot."

ParkingManager
→ knows HOW to manage parking spots.
```

```text
ParkingLot
→ "Close this ticket."

TicketManager
→ knows HOW to manage tickets.
```

### Concept: Separation of Responsibilities

A coordinator can orchestrate several objects without owning all their internal logic.

---

## 14. Overall design so far

```text
                         ParkingLot
                       (Coordinator)
                     /       |       \
                    /        |        \
                   ↓         ↓         ↓
        ParkingManager  TicketManager  PriceCalculator
              |              |               |
              ↓              ↓               ↓
        Parking Floors     Tickets       Pricing Rules
              |
              ↓
        Parking Spots
```

Supporting entities:

```text
Vehicle
ParkingFloor
ParkingSpot
Ticket
```

---

## 15. Current responsibilities

| Component | Responsibility |
|---|---|
| `ParkingLot` | Coordinates parking entry/exit operations |
| `ParkingManager` | Manages parking floors/spots and parking rules |
| `ParkingFloor` | Represents a parking floor and its spots |
| `ParkingSpot` | Represents an individual parking spot |
| `Vehicle` | Represents a vehicle and its type |
| `Ticket` | Represents a parking transaction |
| `TicketManager` | Manages ticket creation and exit/closure |
| `PriceCalculator` | Defines the pricing calculation contract |
| Concrete price calculators | Implement different pricing rules |

---

## 16. Concepts discovered

### Responsibility

Each class should have a clear responsibility.

Do not create classes just to make the design bigger.

### Abstraction

`PriceCalculator` is an abstraction that hides the pricing implementation.

### Separation of Concerns

We separated:

```text
Parking management
Ticket management
Price calculation
Overall coordination
```

### Composition

`ParkingLot` uses specialized components:

```text
ParkingLot
    |
    +-- ParkingManager
    +-- TicketManager
    +-- PriceCalculator
```

### Don't over-design

Only introduce an abstraction when a real requirement justifies it.

Examples of things we intentionally did not introduce yet:

```text
ParkingFloorManager
TicketRepository
Multiple ParkingLot implementations
Floor-level vehicle restrictions
```

---

## 17. Where we stopped

The last unresolved question was:

`ParkingManager.findAvailableSpot(vehicle)` should return what?

Option 1:

```ts
ParkingSpot
```

Option 2:

```ts
{
    floorNumber: number;
    spotNumber: number;
}
```

We had not finalized this decision yet.

This is where the next Parking Lot session can continue.

---

## Core takeaway

The most important lesson from this session:

> **`ParkingLot` is the coordinator, not the owner of every piece of logic.**

The system evolved into:

```text
ParkingLot
    ↓
coordinates
    |
    +── ParkingManager
    |      → parking state/rules
    |
    +── TicketManager
    |      → ticket lifecycle
    |
    +── PriceCalculator
           → pricing algorithm
```

We discovered the design by asking:

> **"Who should be responsible for this?"**

rather than starting with predefined design patterns.
