Video Progress Tracker
The Video Progress Tracker is a web application that enables users to track their video watching progress and resume playback from where they left off. It features a React-based frontend and a Node.js/Express backend, with MongoDB Atlas as the database for storing progress data.
Table of Contents

Project Overview
Features
Setup Instructions
Prerequisites
Backend Setup
Frontend Setup


Design Documentation
Tracking Watched Intervals
Merging Intervals for Unique Progress Calculation
Challenges and Solutions


Code Explanation
Backend
Frontend


Repository Structure
Submission Details

Project Overview
This application allows users to select and watch videos, track their progress, and view a dashboard of their watching history. Progress is calculated based on unique watched time, preventing inflation from skipping or re-watching segments. The backend stores data in MongoDB Atlas, and the frontend provides an intuitive interface for video playback and progress visualization.
Features

Video Selection: Choose from a list of sample videos.
Progress Tracking: Records watched intervals and calculates progress as a percentage.
Resume Playback: Automatically resumes videos from the last watched position.
Progress Dashboard: Displays progress for all watched videos, including intervals and timestamps in IST.
Timezone Support: Timestamps are stored in UTC but displayed in Indian Standard Time (IST, UTC+5:30).

Setup Instructions
Prerequisites

Node.js (v18 or later): Required for running the backend and frontend.
MongoDB Atlas Account: A free-tier account for the database.
Git: For cloning the repository.
Postman (optional): For testing API endpoints manually.

Backend Setup

Clone the Repository:
git clone https://github.com/your-username/video-progress-tracker.git
cd video-progress-tracker/backend


Install Dependencies:
npm install


Set Up Environment Variables:

Create a .env file in the backend directory with the following content:MONGODB_URI=mongodb+srv://<username>:<password>@<cluster>.mongodb.net/video-progress-tracker?retryWrites=true&w=majority
PORT=5000


Replace <username>, <password>, and <cluster> with your MongoDB Atlas credentials. See the MongoDB Atlas Setup section below for details on obtaining the connection string.


Run the Backend:
node server.js


If successful, you’ll see:MongoDB connected successfully
Server running on port 5000





MongoDB Atlas Setup

Sign up for a free MongoDB Atlas account at mongodb.com.
Create a Shared (M0) cluster:
Choose a cloud provider (e.g., AWS) and region.
Name the cluster (e.g., Cluster0).


Configure network access:
Go to "Network Access" and add your IP address, or use 0.0.0.0/0 (allow access from anywhere) for testing.


Create a database user:
Go to "Database Access," add a new user with a username and password, and grant "Read and Write to Any Database" permissions.


Get the connection string:
Go to "Clusters," click "Connect," then "Connect Your Application."
Select Node.js driver (version 3.6 or later) and copy the mongodb+srv URI.
Update the .env file with this URI, replacing <username>, <password>, and the database name (video-progress-tracker).



Frontend Setup

Navigate to the Frontend Directory:
cd ../frontend


Install Dependencies:
npm install


Set Up Environment Variables:

Create a .env file in the frontend directory with the following:VITE_API_URL=http://localhost:5000/api


If the backend is hosted elsewhere (e.g., on Render), update the VITE_API_URL to the backend URL.


Run the Frontend:
npm run dev


Open http://localhost:3000 in your browser to access the application.



Design Documentation
Tracking Watched Intervals

Frontend Implementation (VideoPlayer.jsx):
The VideoPlayer component listens to the HTML5 video element’s timeupdate event to track when the video is playing.
When playback starts, a new interval begins (start time). When the user pauses, seeks, or the video ends, the interval ends (end time).
Intervals are stored in state as an array (e.g., [{ start: 0, end: 20 }, { start: 30, end: 50 }]).
A debounced function saves intervals to the backend every 2 seconds to reduce API calls.


Backend Storage (model/progress.js and routes/progress.js):
The Progress model defines a schema with an intervals array of objects containing start and end times.
The POST /progress endpoint saves these intervals to MongoDB Atlas, updating the document if it exists or creating a new one.



Merging Intervals for Unique Progress Calculation

Backend Middleware (middleware/calculateProgress.js):
The calculateProgress middleware processes intervals before saving them:
Sort Intervals: Sorts intervals by start time to ensure chronological order.
Merge Overlaps: Iterates through sorted intervals, merging overlapping ones (e.g., [0, 20] and [10, 30] become [0, 30]).
Calculate Unique Time: Sums the duration of merged intervals (e.g., 30 seconds for [0, 30]).
Compute Progress: Divides unique watched time by the video duration and multiplies by 100 to get a percentage (e.g., if video duration is 100 seconds, progress = (30 / 100) * 100 = 30%).


This prevents progress inflation from re-watching or skipping segments.


Storage:
The merged intervals and calculated progress are saved in the Progress document, ensuring accurate tracking.



Challenges and Solutions

Challenge 1: MongoDB Connection Errors:
Issue: Initially encountered ECONNREFUSED errors when trying to connect to a local MongoDB instance.
Solution: Switched to MongoDB Atlas, configured the MONGODB_URI in the .env file, and ensured proper IP whitelisting in Atlas.


Challenge 2: Overlapping Intervals Inflating Progress:
Issue: Overlapping intervals (e.g., [0, 20] and [10, 30]) resulted in double-counted time if not merged.
Solution: Implemented the calculateProgress middleware to merge overlapping intervals before calculating progress.


Challenge 3: Timezone Discrepancies:
Issue: Timestamps in MongoDB are stored in UTC, but users needed to see them in IST (UTC+5:30).
Solution: Added a toIST function in the backend to convert UTC timestamps to IST for API responses, and used toLocaleString with Asia/Kolkata timezone in the frontend to display dates (e.g., a timestamp saved at 07:10 AM UTC on May 25, 2025, is displayed as "May 25, 2025, 12:40 PM" in IST).



Code Explanation
Backend

server.js:
Initializes the Express server, sets up middleware (CORS, JSON parsing), and connects to MongoDB Atlas using Mongoose.
Mounts the /api routes and starts the server on port 5000 (or the port specified in .env).


model/progress.js:
Defines the Mongoose schema for the Progress model, including fields for userId, videoId, intervals, videoDuration, lastWatchedPosition, progress, and updatedAt.


middleware/calculateProgress.js:
Middleware that merges overlapping intervals and calculates the unique progress percentage, attaching the results to the request object.


routes/progress.js:
Defines four endpoints:
POST /progress: Saves or updates progress data, including intervals and calculated progress.
GET /progress: Retrieves progress for a specific user and video.
GET /user/:userId/progress: Fetches all progress entries for a user, with updatedAt timestamps converted to IST.
DELETE /progress: Deletes progress for a specific video (added for testing purposes).





Frontend

App.jsx:
The main component that renders the VideoSelector, VideoPlayer, and ProgressDashboard components within a VideoProvider context.


context/VideoContext.jsx:
Provides a React context for managing global state (userId, videoId) and API functions (saveProgress, loadProgress).


components/VideoSelector.jsx:
Renders a dropdown menu for selecting videos from a predefined list.


components/VideoPlayer.jsx:
Handles video playback, tracks watched intervals using video events (timeupdate, pause, seeked), and saves progress to the backend.


components/ProgressDashboard.jsx:
Displays a list of progress entries for the user, showing progress percentage, intervals, and timestamps in IST.


api.js:
Configures Axios for API requests, with functions to save progress, load progress, and fetch user progress.



Repository Structure
video-progress-tracker/
├── backend/
│   ├── middleware/
│   │   └── calculateProgress.js
│   ├── model/
│   │   └── progress.js
│   ├── routes/
│   │   └── progress.js
│   ├── .env
│   ├── package.json
│   ├── server.js
├── frontend/
│   ├── public/
│   ├── src/
│   │   ├── components/
│   │   │   ├── ProgressDashboard.jsx
│   │   │   ├── VideoPlayer.jsx
│   │   │   └── VideoSelector.jsx
│   │   ├── context/
│   │   │   └── VideoContext.jsx
│   │   ├── App.jsx
│   │   ├── api.js
│   │   └── index.css
│   ├── .env
│   ├── package.json
├── README.md
└── .gitignore

Submission Details

Repository: The project is hosted on GitHub at https://github.com/your-username/video-progress-tracker (replace with your actual repository URL).
Commits:
Used multiple small commits with meaningful messages:
feat(backend): add progress model and schema
feat(backend): implement progress routes and middleware
feat(frontend): create video selector and player components
feat(frontend): add progress dashboard component
docs: update README with setup and design documentation




Branching:
Used feature branches:
feature/backend-setup for backend development.
feature/frontend-components for frontend development.
Merged into main with descriptive merge messages.




Access:
The repository is public, or access has been granted to reviewers via GitHub.



