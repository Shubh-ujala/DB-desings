# Fitness Influencer Coaching Platform - DB Design

This folder contains the database design for a fitness coaching platform where:

- Users can subscribe to trainer plans
- Trainers can publish multiple coaching plans
- Sessions can be scheduled between users and trainers
- Payments are tracked against active subscriptions

## Core Modules

1. Identity and Profiles
2. Trainer Plan Management
3. Subscriptions
4. Session Scheduling
5. Payments

## Entities

### 1) `user_profile`
Stores authentication-level and account-level information.

Fields:
- `id` (PK)
- `name`
- `email`
- `role` (`user` or `trainer`)
- `createdAt`, `updatedAt`

### 2) `user_details`
Extended fitness profile for users.

Fields:
- `id` (PK)
- `user_id` (FK -> `user_profile.id`)
- `age`
- `height`
- `gender` (`Male`, `Female`, `Other`)
- `fitness_goal`
- `body_weight`

### 3) `trainer_details`
Extended profile for trainers.

Fields:
- `id` (PK)
- `trainer_id` (FK -> `user_profile.id`)
- `specialization`
- `bio`
- `experience_years`
- `rating`

### 4) `plans`
Coaching plans created by trainers.

Fields:
- `id` (PK)
- `trainer_id` (FK -> `trainer_details.id`)
- `plan_title`
- `plan_description`
- `plan_duration`
- `plan_price`
- `plan_type` (`consultation`, `coaching`, `live session`)
- `createdAt`, `updatedAt`

### 5) `subscriptions`
Tracks user enrollments into plans.

Fields:
- `id` (PK)
- `user_id` (FK -> `user_profile.id`)
- `plan_id` (FK -> `plans.id`)
- `start`
- `end`
- `status` (`active`, `expired`, `cancelled`)

### 6) `sessions`
Scheduled trainer-user interactions under a subscription.

Fields:
- `id` (PK)
- `trainer_id` (FK -> `trainer_details.id`)
- `user_id` (FK -> `user_details.id`)
- `subscription_id` (FK -> `subscriptions.id`)
- `session_type` (`consultation`, `live`, `coaching`)
- `scheduled_at`
- `session_duration`
- `meeting_link`
- `createdAt`, `updatedAt`

### 7) `payments`
Payment records for subscriptions.

Fields:
- `id` (PK)
- `user_id` (FK -> `user_profile.id`)
- `subscription_id` (FK -> `subscriptions.id`)
- `amount`
- `payment_method` (`Cash`, `Online`)
- `status` (`failed`, `success`, `pending`)
- `transaction_date`

## Relationships

- One `user_profile` -> many `subscriptions`
- One `plans` -> many `subscriptions`
- One `trainer_details` -> many `plans`
- One `trainer_details` -> many `sessions`
- One `user_details` -> many `sessions`
- One `subscriptions` -> many `sessions`
- One `subscriptions` -> many `payments`

## Suggested Constraints

- Add `UNIQUE (email)` in `user_profile`
- Add `CHECK (plan_price >= 0)` and `CHECK (amount >= 0)`
- Add date validation: `end >= start` in `subscriptions`
- Add indexes:
	- `subscriptions(user_id)`
	- `subscriptions(plan_id)`
	- `plans(trainer_id)`
	- `sessions(subscription_id, scheduled_at)`
	- `payments(subscription_id, transaction_date)`

## Notes

- Table name normalized as `subscriptions` (instead of misspelled variants).
- Subscription status uses `expired` for consistency.
- `plans.trainer_id` should reference trainer details, not user details.

This schema is suitable as a base for MySQL or PostgreSQL implementation and can be extended with progress tracking, workout logs, and nutrition plans.
