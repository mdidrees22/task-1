# Goal Management Module

The Goal Management module is the core of the Spatium Nexum backend. It handles the lifecycle of user-defined health goals, ensuring strict data integrity between the Personal and Analytics databases via a unique Identity Bridge.

## Technical Architecture

This module utilizes a Stateless REST Architecture secured by JWT (JSON Web Tokens).

Multi-DB Sync: Every goal creation triggers a cross-database lookup to fetch the user's archetype from the sn_analytics_local database using the analyticCodePrefix.

Transactional Integrity: All write operations are wrapped in @Transactional blocks to prevent "orphan" records or half-saved data states.

Security: Ownership is validated at the service layer. A user can only view, update, or delete goals they created.

## Endpoint Summary

**Base Path:** `/goal`  
*All requests require an Authorization header: `Authorization: Bearer <JWT_TOKEN>`*


| Action | Method | Endpoint | Request Body | Description |
| :--- | :---: | :--- | :--- | :--- |
| **Create Goal** | `POST` | `/goal/create` | `GoalRequestDTO` | Initializes a new health goal and syncs archetype from Analytics DB. |
| **List All** | `GET` | `/goal/all` | *None* | Retrieves all goals belonging to the authenticated user. |
| **Filter Goals** | `GET` | `/goal/all?theme=X` | *None* | Filters user goals by specific health themes. |
| **Get Details** | `GET` | `/goal/{id}` | *None* | Returns the full details for a specific goal (validates ownership). |
| **Update Progress**| `PUT` | `/goal/{id}` | `GoalRequestDTO` | Updates the title, status, or `tracking_data` logs. |
| **Delete Goal** | `DELETE` | `/goal/{id}` | *None* | Permanently removes a goal from the Personal Database. |


## Postman Guide
1. Authentication
All requests must include the Bearer Token in the Authorization header:
```header
Authorization: Bearer <your_jwt_token> //user login token
```
2. Sample Request Body 
### create goal (post)
http://localhost:8081/goal/create

Body
```
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
Response //check goal table in pg admin
```json
{
    "goal_id": 10,
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
    "goal_image": "https://example.com/sleep.png",
    "tracking_data": null,
    "userAnalyticCode": "GCO",
    "userEmail": "idrees@spatiumnexum.com"
}
```

### See all goals (get)
http://localhost:8081/goal/all
```bash
Body: None.
Response: A JSON array containing the goal you just created
```
### Update Goal (put)
http://localhost:8081/goal/{id} (Replace {id} with the goal_id from Test 1)

Body (JSON)
```bash
{
    "title": "Bio-Hacking Deep Sleep",
    "status": "IN_PROGRESS",
    "tracking_data": [
        { "date": "2026-01-25", "completed": true, "note": "Slept 8 hours" }
    ]
}
```
Response
```bash
add some actual progress data to the tracking_data list
//check goal table in pg admin
```

### Delete Goal (delete)
http://localhost:8081/goal/{id}

Body (JSON)
```bash
none
```
Response
```bash
204 No Content //check goal table in pg admin
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
## Error Handling
This project is proprietary software developed by Spatium Nexum.

## Support
For issues, questions, or contributions, please contact the development team or open an issue on the repository.
