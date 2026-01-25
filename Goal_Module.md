# Goal Management Module

The Goal Management module is the core of the Spatium Nexum backend. It handles the lifecycle of user-defined health goals, ensuring strict data integrity between the Personal and Analytics databases via a unique Identity Bridge.

## Technical Architecture

This module utilizes a Stateless REST Architecture secured by JWT (JSON Web Tokens).

Multi-DB Sync: Every goal creation triggers a cross-database lookup to fetch the user's archetype from the sn_analytics_local database using the analyticCodePrefix.

Transactional Integrity: All write operations are wrapped in @Transactional blocks to prevent "orphan" records or half-saved data states.

Security: Ownership is validated at the service layer. A user can only view, update, or delete goals they created.

## Endpoint Summary
```bash
pip install foobar
```

## Postman Guide
1. Authentication
All requests must include the Bearer Token in the Authorization header:
```header
Authorization: Bearer <your_jwt_token> //user login token
```
2. Sample Request Body 
(POST /goal/create)
```body
{
    "title": "Deep Sleep Recovery",
    "duration": "14 days",
    "daily_check_in": true,
    "relevant_biomarkers": ["Melatonin", "Cortisol"],
    "goal_start": "2026-01-25",
    "goal_end": "2026-02-08",
    "theme": "Circadian Rhythm",
    "tracker_type": "BINARY",
    "goal_type": "biomarker",
    "status": "IN_PROGRESS",
    "goal_image": "https://example.com/sleep.png"
}
```
3. Response Structure
```json
{
    "goal_id": 9,
    "title": "Deep Sleep Recovery",
    "duration": "14 days",
    "daily_check_in": true,
    "relevant_biomarkers": [
        "Melatonin",
        "Cortisol"
    ],
    "goal_start": "2026-01-25",
    "goal_end": "2026-02-08",
    "theme": "Circadian Rhythm",
    "archetype": "explorer",
    "tracker_type": "BINARY",
    "goal_type": "biomarker",
    "status": "IN_PROGRESS",
    "completion_date": null,
    "tracking_data": null,
    "created_at": "2026-01-25T19:34:11.279297",
    "goal_image": "https://example.com/sleep.png",
    "user": {
        "id": 1,
        "email": "idrees@spatiumnexum.com",
        "firstName": "mohd",
        "lastName": "idrees",
        "identityBridgeCode": "GCO507",
        "analyticCodePrefix": "GCO",
        "createdAt": "2026-01-24T18:29:30.972374Z"
    },
    "userAnalyticCode": "GCO"
}
```
## Error Handling
```bash
The module utilizes a Global Exception Handler to return standardized JSON error messages:

401 Unauthorized: Token is missing, expired, or invalid.

403 Forbidden: Attempting to access/modify a goal ID that belongs to another user.

404 Not Found: The goal ID does not exist or the user's Analytic Profile is missing.

500 Internal Server Error: Database connectivity issues or transactional rollbacks.
```
## Spatium Nexum: Logical Data Bridge

```bash
[ DATABASE: sn_personal_local ]          [ DATABASE: sn_analytics_local ]
  ===============================          ================================
          
      +--------------+                            +-------------------+
      |     USER     |                            |   USER_ANALYTIC   |
      +--------------+                            +-------------------+
      | id (PK)      |                            | analytic_code (PK)| <---+
      | email        |                            | archetype         |     |
      | password     |                            | responses_json    |     |
      | prefix_code  | --- ( VIRTUAL BRIDGE ) --- |                   |     |
      +--------------+             |              +-------------------+     |
             |                     |                                        |
             | (1-to-Many)         |                                        |
             v                     |                                        |
      +--------------+             |                                        |
      |     GOAL     |             |                                        |
      +--------------+             |                                        |
      | goal_id (PK) |             |                                        |
      | user_id (FK) |             |                                        |
      | title        |             |                                        |
      | analytic_code| <-----------+----------------------------------------+
      +--------------+
```

