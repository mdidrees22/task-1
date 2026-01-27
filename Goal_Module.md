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

Body for personal goal
```
{
    "title": "Summit Mount Everest (or a Seven Summit)",
    "duration": "Ongoing",
    "daily_check_in": true,
    "relevant_biomarkers": ["Hypoxic Tolerance"],
    "goal_start": "2026-01-19",
    "goal_end": "2026-12-31",
    "theme": "Physical Excellence",
    "tracker_type": "BINARY",
    "goal_type": "personal",
    "status": "IN_PROGRESS",
    "goal_image": "@/assets/goal_images/goal_7.png",
    "tracking_data": []
}
```
Response //check goal table in pg admin
```json
{
    "goal_id": 4,
    "goalTitle": "Summit Mount Everest (or a Seven Summit)",
    "sinceDays": "8",
    "days": null,
    "daysCompleted": null,
    "last5DaysProgress": "00000",
    "goalImage": "@/assets/goal_images/goal_7.png",
    "completed": false,
    "checkInQuestion": "Did you train altitude simulation or technical skills today?",
    "about": "Engineering a physiology that thrives on scarcity to conquer the vertical limit.",
    "inAppNotificationMessage": null,
    "lastValue": null,
    "themeIcons": [],
    "theme": "Physical Excellence",
    "archetype": "explorer",
    "goal_type": "personal",
    "status": "IN_PROGRESS",
    "userAnalyticCode": "GCO",
    "tracking_data": [],
    "goal_start": "2026-01-19",
    "goal_end": "2026-12-31"
}
```
Body for archetype goal
```
{
    "title": "Novelty Exposure",
    "duration": "31 days",
    "daily_check_in": true,
    "relevant_biomarkers": [],
    "goal_start": "2026-01-01",
    "goal_end": "2026-01-31",
    "theme": "Hormone Balance",
    "tracker_type": "BINARY",
    "goal_type": "archetype",
    "status": "IN_PROGRESS",
    "goal_image": "@/assets/goal_images/goal_1.png",
    "tracking_data": [
        {"date": "2026-01-25", "status": true},
        {"date": "2026-01-26", "status": false}
    ]
}
```
Response //check goal table in pg admin
```
{
    "goal_id": 5,
    "goalTitle": "Novelty Exposure",
    "sinceDays": null,
    "days": "31",
    "daysCompleted": "12",
    "last5DaysProgress": "10011",
    "goalImage": "@/assets/goal_images/goal_1.png",
    "completed": false,
    "checkInQuestion": "Have you engaged in 15 minutes of divergent thinking exercises today?",
    "about": null,
    "inAppNotificationMessage": "Keep your mind flexible. Did you do your divergent thinking exercise?",
    "lastValue": null,
    "themeIcons": [
        "CalmIcon"
    ],
    "theme": "Hormone Balance",
    "archetype": "explorer",
    "goal_type": "archetype",
    "status": "IN_PROGRESS",
    "userAnalyticCode": "GCO",
    "tracking_data": [
        {
            "date": "2026-01-25",
            "status": true
        },
        {
            "date": "2026-01-26",
            "status": false
        }
    ],
    "goal_start": "2026-01-01",
    "goal_end": "2026-01-31"
}
```
Body for biomarker goal
```
{
    "title": "Tyrosine Loading",
    "duration": "8 days",
    "daily_check_in": true,
    "relevant_biomarkers": ["Dopamine", "Tyrosine"],
    "goal_start": "2026-01-19",
    "goal_end": "2026-01-27",
    "theme": "Cognitive Performance",
    "tracker_type": "BINARY",
    "goal_type": "biomarker",
    "status": "IN_PROGRESS",
    "goal_image": "@/assets/goal_images/goal_4.png",
    "tracking_data": []
}
```
Response //check goal table in pg admin
```
{
    "goal_id": 6,
    "goalTitle": "Tyrosine Loading",
    "sinceDays": "8",
    "days": null,
    "daysCompleted": null,
    "last5DaysProgress": "00000",
    "goalImage": "@/assets/goal_images/goal_4.png",
    "completed": false,
    "checkInQuestion": "Have you consumed your specific protein source 60 mins before your session?",
    "about": null,
    "inAppNotificationMessage": "Ready to brainstorm? Did you take your Tyrosine 60 mins ago?",
    "lastValue": "45 µmol/L",
    "themeIcons": [
        "BoltIcon",
        "EyeIcon"
    ],
    "theme": "Cognitive Performance",
    "archetype": "explorer",
    "goal_type": "biomarker",
    "status": "IN_PROGRESS",
    "userAnalyticCode": "GCO",
    "tracking_data": [],
    "goal_start": "2026-01-19",
    "goal_end": "2026-01-27"
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
