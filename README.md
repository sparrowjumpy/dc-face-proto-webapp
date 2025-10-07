# DC Face Proto Web App

This repository contains a full‑stack web application that brings the
command‑line capabilities of **dc‑face‑proto** to the browser.  It
provides a modern, responsive user interface built with React and a
FastAPI backend that uses InsightFace for face detection and
embedding.  The application supports live monitoring, enrolment of
new people and attendance reporting.

## Architecture

The project is organised into two top‑level folders:

| Folder  | Description                                                   |
|-------- |---------------------------------------------------------------|
| `backend/` | Python FastAPI service implementing the REST API and wrapping the face recognition logic.  It stores data in a local SQLite database and saves portrait/unknown images to disk. |
| `client/`  | React single‑page application built with Create React App and Material UI.  It interacts with the backend via JSON over HTTP. |

### Backend

The backend exposes endpoints under `/api`:

| Endpoint              | Method | Purpose                                                    |
|-----------------------|-------|------------------------------------------------------------|
| `/api/people`         | GET   | List all enrolled people with IDs and names.              |
| `/api/enroll`         | POST  | Enrol a person given a name and a list of base64 images.  |
| `/api/recognize`      | POST  | Recognise faces in a single image (data URL).             |
| `/api/report/{date}`  | GET   | Return a summary of attendance for the given date in ISO format (YYYY‑MM‑DD). |

See `backend/main.py` for the implementation details.  The face
recognition logic lives in `backend/services/face_service.py` and
mirrors the functionality of the original CLI scripts.

#### Running the backend

Install the dependencies listed in `backend/requirements.txt` into a
virtual environment and start the server with Uvicorn:

```bash
cd backend
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt

# Start the API on port 8000
uvicorn main:app --reload --host 0.0.0.0 --port 8000
```

The API listens on `http://localhost:8000` by default.  CORS is
enabled to allow the frontend to communicate with it from another
origin during development.

#### Configuration

Backend configuration resides in `backend/config.yaml`.  You can
adjust camera settings, recognition thresholds and database path
according to your deployment environment.  The file is loaded at
startup.  Portraits and unknown faces are saved in `portraits/` and
`unknowns/` respectively.

### Frontend

The frontend is a React application using Material UI for a clean,
enterprise‑style design.  It features three pages accessible from the
navigation bar:

* **Dashboard:** shows the live webcam feed and displays the names and
  similarity scores of recognised faces in real time.  Frames are
  captured periodically and sent to the backend for inference.
* **Enroll:** allows an operator to capture multiple images of a
  person using the webcam and submit them along with a name to the
  backend for enrolment.
* **Reports:** fetches and displays a summary of attendance for a
  selected date in a tabular view.

#### Running the frontend

You need Node.js (v14 or newer) installed.  Inside the `client`
folder run:

```bash
cd client
npm install
npm start
```

The app starts on `http://localhost:3000` and expects the backend to
be available on `http://localhost:8000`.  During production you can
build the frontend (`npm run build`) and serve the static files from
the backend using `fastapi.staticfiles`.

## Deployment

For production you may want to run the backend behind a reverse proxy
such as Nginx and serve the React build directly from FastAPI.  The
database can be migrated to PostgreSQL or MySQL by replacing the
SQLite connection code.  InsightFace can utilise a GPU via ONNX
Runtime’s DirectML or CUDA providers to accelerate inference.

## Logo

A placeholder logo is included at `client/src/logo.png`.  Replace it
with your company’s logo to personalise the application.  The logo is
rendered in the navigation bar.