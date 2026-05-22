# EventHub — Event Ticketing API

A backend REST API for a simplified event ticketing platform, built with Django and Django REST Framework.

---

## How to Run the Project

```bash
# 1. Clone the repository
git clone <your-repo-url>
cd eventhub

# 2. Create and activate a virtual environment
python -m venv venv
source venv/bin/activate        # macOS/Linux
venv\Scripts\activate           # Windows

# 3. Install dependencies
pip install -r requirements.txt

# 4. Apply migrations
python manage.py migrate

# 5. Start the development server
python manage.py runserver
```

The API is now available at `http://127.0.0.1:8000/api/`

---

## Endpoints

### Event Endpoints

| Method | URL | Description |
|--------|-----|-------------|
| GET | `/api/events/` | List all events (supports `?status=` and `?venue=` filters) |
| POST | `/api/events/` | Create a new event |
| GET | `/api/events/{id}/` | Retrieve a single event |
| PUT | `/api/events/{id}/` | Update an event |
| PATCH | `/api/events/{id}/` | Partially update an event |
| DELETE | `/api/events/{id}/` | Delete an event |

**Query Parameters:**
- `?status=upcoming` — filter events by status (`upcoming`, `ongoing`, `completed`, `cancelled`)
- `?venue=bangalore` — case-insensitive partial match on venue name

### Reservation Endpoints

| Method | URL | Description |
|--------|-----|-------------|
| GET | `/api/reservations/` | List all reservations (supports `?event_id=` filter) |
| POST | `/api/reservations/` | Create a reservation (deducts seats from event) |
| GET | `/api/reservations/{id}/` | Retrieve a single reservation |
| PUT | `/api/reservations/{id}/` | Update a reservation |
| PATCH | `/api/reservations/{id}/` | Partially update a reservation |
| DELETE | `/api/reservations/{id}/` | Delete a reservation |
| POST | `/api/reservations/{id}/cancel/` | Cancel a reservation (restores seats to event) |

**Query Parameters:**
- `?event_id=1` — filter reservations by event

---

## Sample Requests

### Create an Event
```json
POST /api/events/
{
  "title": "PyCon India 2025",
  "venue": "NIMHANS Convention Centre, Bangalore",
  "date": "2025-09-20",
  "total_seats": 500,
  "available_seats": 500,
  "status": "upcoming"
}
```

### Reserve Seats
```json
POST /api/reservations/
{
  "event": 1,
  "attendee_name": "Priya Sharma",
  "attendee_email": "priya@example.com",
  "seats_reserved": 2
}
```

### Cancel a Reservation
```
POST /api/reservations/1/cancel/
```
No request body needed.

---

## Design Decision

**Seat deduction inside the serializer's `create()` method**

I chose to handle the seat deduction (`event.available_seats -= seats_reserved`) directly inside `ReservationSerializer.create()`, rather than in the ViewSet. This keeps both related writes — updating the event seat count and creating the reservation record — in one place, making the logic easy to follow and test. The serializer already has access to the validated event object, so it's the natural place to perform this side effect. The same pattern is applied in reverse in the `cancel` action: when a reservation is cancelled, the seats are immediately restored to the event.

---

## Middleware

`RequestLoggingMiddleware` logs every request to the terminal in the format:

```
METHOD /path - STATUS_CODE - duration_in_seconds
```

Example:
```
POST /api/reservations/ - 201 - 0.03s
GET /api/events/ - 200 - 0.01s
```
