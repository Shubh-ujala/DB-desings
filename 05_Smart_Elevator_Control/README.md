# Smart Elevator Control - DB Design

This folder contains the database design for a **Smart Elevator Control System**.

The model is focused on:
- Managing buildings and their floors
- Tracking elevator inventory and live status
- Capturing floor requests and ride assignments
- Logging completed trips
- Recording maintenance issues and resolution lifecycle

## Problem Statement

In a multi-floor building, elevator operations should be trackable and optimizable. The system must:
- Register buildings, floors, and elevators
- Accept floor requests (`up` / `down`)
- Assign an elevator to each request
- Track elevator position and movement state
- Keep logs for ride analytics
- Capture maintenance events and repair status

## Core Entities

### 1. Building
Stores building-level metadata.

Key fields:
- `id` (PK)
- `location`
- `building_name`
- `floors`
- `createdAt`, `updatedAt`

### 2. Building Floor
Represents each floor in a building.

Key fields:
- `id` (PK)
- `floor_number`
- `building_id` (FK -> Building)
- `createdAt`, `updatedAt`

### 3. Elevator
Represents elevator units installed in a building.

Key fields:
- `id` (PK)
- `capacity`
- `building_id` (FK -> Building)
- `elevator_installed`
- `elevator_servicing`
- `createdAt`, `updatedAt`

### 4. Floor Request
Raised when someone requests an elevator from a floor.

Key fields:
- `id` (PK)
- `floor_id` (FK -> Building Floor)
- `building_id` (FK -> Building)
- `request_time`
- `direction` (`up`, `down`)
- `createdAt`, `updatedAt`

### 5. Ride Assignment
Maps a floor request to an elevator selected by scheduling logic.

Key fields:
- `id` (PK)
- `request_id` (FK -> Floor Request)
- `elevator_id` (FK -> Elevator)
- `assignedAt`
- `createdAt`, `updatedAt`

### 6. Elevator Status
Stores near real-time status snapshots for each elevator.

Key fields:
- `id` (PK)
- `elevator_id` (FK -> Elevator)
- `building_floor_id` (FK -> Building Floor)
- `moving_direction` (`up`, `down`, `idle`)
- `status` (`idle`, `moving`, `stopped`)
- `need_repairing` (boolean)
- `createdAt`, `updatedAt`

### 7. Ride Logs
Historical logs for completed or in-progress trips.

Key fields:
- `id` (PK)
- `request_id` (FK -> Floor Request)
- `elevator_id` (FK -> Elevator)
- `floor_id` (FK -> Building Floor)
- `startedAt`
- `endAt`

### 8. Maintenance Record
Tracks issues and repair lifecycle.

Key fields:
- `id` (PK)
- `elevator_id` (FK -> Elevator)
- `issue_details`
- `status` (`open`, `in_progress`, `resolved`)
- `startedAt`, `resolvedAt`
- `createdAt`, `updatedAt`

## Relationship Summary

- One **Building** has many **Building Floors**
- One **Building** has many **Elevators**
- One **Building Floor** has many **Floor Requests**
- One **Floor Request** can have one **Ride Assignment**
- One **Elevator** can have many **Ride Assignments**
- One **Floor Request** can generate one or more **Ride Logs**
- One **Elevator** can have many **Ride Logs**
- One **Elevator** can have many **Elevator Status** snapshots
- One **Elevator** can have many **Maintenance Records**

## Operational Flow

1. User places a request from a floor (`floor_request`).
2. Scheduler assigns the best elevator (`ride_assignment`).
3. Elevator movement/position updates are tracked (`elevator_status`).
4. Trip lifecycle gets logged (`ride_logs`).
5. Faults and servicing are tracked (`maintenance_record`).

## Suggested Improvements

To make this production-ready, consider:
- Add indexes on high-frequency columns:
	- `floor_request(building_id, request_time)`
	- `elevator_status(elevator_id, createdAt)`
	- `ride_logs(elevator_id, startedAt)`
- Add `UNIQUE(building_id, floor_number)` on `building_floor`
- Use `CHECK` constraints for enum-like fields (or native ENUM where supported)
- Add soft-delete columns (`deletedAt`) if historical restore is needed
- Add `response_time_ms` or computed metrics for SLA analytics

## Notes

- This README reflects the current schema draft from `code.txt` with naming normalized for clarity.
- You can use this as the base documentation before converting to SQL DDL / Prisma / DBML.
