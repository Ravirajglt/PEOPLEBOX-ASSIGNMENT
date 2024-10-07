# PEOPLEBOX-ASSIGNMENT
Overview
This project is a backend service for an AI-powered talent acquisition platform that recommends relevant job postings to users based on their profiles. It uses a weighted matching algorithm to match users with jobs based on their skills, experience, and preferences.

The backend is developed using Flask for the RESTful API, MongoDB for data storage, and includes robust error handling and scalability features.

Features
User Profile Matching: Recommends jobs based on user skills, experience, and preferences.
Weighted Matching Algorithm: Prioritizes skills (70%), experience (20%), and preferences (10%) to generate recommendations.
RESTful API: A simple API to accept user profiles and return a list of job recommendations in JSON format.
MongoDB Database: Stores job postings and user profiles.
Error Handling: Handles invalid inputs and database connection errors gracefully.
Technology Stack
Backend Framework: Flask (Python)
Database: MongoDB
Libraries:
Flask: For API development
pymongo: For MongoDB integration
flask_cors: For enabling CORS
dotenv: For environment variable management
