# Comic-con Parking System (DB Design)

This folder contains a database design for a Comic-con parking management system.
It supports vehicle registration, zone and spot management, ticketing, parking sessions,
and payment tracking.

## Goals

- Organize parking spots by zone and category.
- Track vehicle entry and exit using sessions.
- Generate a ticket for each parking event.
- Reserve selected spots for category-based usage (for example VVIP or EV charging).
- Record payment details and payment status.

## Core Entities

### 1. `vehicle`
- Stores vehicle details.
- Key fields: `id`, `vehicle_number`, `vehicle_owner`, `createdAt`, `updatedAt`.

### 2. `vehicle_category`
- Maps a vehicle to a category.
- Categories: `car`, `bike`, `EV`, `cab`.

### 3. `parking_category`
- Defines parking access level/category.
- Categories: `VVIP`, `VIP`, `staff`, `EV charging`.

### 4. `parking_zone`
- Defines physical location blocks.
- Zones: `BlockA`, `BlockB`, `BlockC`, `basement`.

### 5. `parking_spot`
- Represents individual spots.
- Belongs to one zone and one parking category.
- Tracks availability using `is_available`.

### 6. `parking_ticket`
- Ticket generated for a parking event.
- Uses unique `ticket_number`.

### 7. `spot_reservation`
- Reserves a spot under a parking category.
- Helps enforce reserved allocations.

### 8. `parking_session`
- Captures active/completed parking lifecycle.
- Links: vehicle, spot, ticket, and parking category.
- Tracks `entry_time`, `exit_time`, and `status`.

### 9. `payment`
- Stores billing and payment outcome for a session.
- Includes amount, payment time, method, and status.

## Relationship Summary

- One zone -> many spots.
- One parking category -> many spots.
- One vehicle -> many sessions.
- One spot -> many sessions (over time).
- One ticket -> one session.
- One session -> one payment record (typical flow).
- One spot reservation links one spot and one parking category.

## Functional Flow

1. Register vehicle and assign vehicle category.
2. Select a valid spot from available spots in a zone/category.
3. Create ticket and open parking session (`status = active`).
4. On exit, close session (`status = completed`) and set `exit_time`.
5. Record payment with method and status.

## Suggested Constraints and Validation

- `ticket_number` should be unique.
- Prevent assigning unavailable spots to active sessions.
- Allow only one active session per vehicle at a time.
- Ensure `exit_time >= entry_time`.
- Restrict enum values to defined categories and statuses.

## Files

- `data.txt`: schema/model draft in diagram-style format.
- `README.md`: human-readable explanation of the database design.

## Notes

The source schema draft uses mixed naming styles (for example `parking_Session` and `parking_session_id`).
If you implement this in SQL/ORM, keep naming consistent (recommended: snake_case).
