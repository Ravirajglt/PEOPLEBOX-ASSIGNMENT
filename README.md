# PEOPLEBOX-ASSIGNMENT
from flask import Flask, request, jsonify
from flask_cors import CORS
from pymongo import MongoClient
from dotenv import load_dotenv
import os

# Load environment variables from .env file
load_dotenv()

# MongoDB connection string
MONGO_URI = os.getenv('MONGO_URI')

# MongoDB client setup
client = MongoClient(MONGO_URI)
db = client['job_recommendation']

# Collections
users_collection = db['users']
jobs_collection = db['jobs']

# Weights for matching algorithm
WEIGHTS = {
    'skills': 0.7,
    'experience_level': 0.2,
    'preferences': 0.1
}

# Calculate the match score based on user profile and job requirements
def calculate_match_score(user, job):
    score = 0

    # Match skills
    matched_skills = set(user['skills']).intersection(set(job['required_skills']))
    skill_match_percentage = len(matched_skills) / len(job['required_skills'])
    score += WEIGHTS['skills'] * skill_match_percentage

    # Match experience level
    if user['experience_level'] == job['experience_level']:
        score += WEIGHTS['experience_level']

    # Match preferences (location and job type)
    preferences = user['preferences']
    if job['location'] in preferences['locations'] and job['job_type'] == preferences['job_type']:
        score += WEIGHTS['preferences']

    return score

# Fetch job recommendations based on user profile
def get_job_recommendations(user):
    # Fetch all job postings from the database
    jobs = jobs_collection.find()

    # Calculate match score for each job
    scored_jobs = []
    for job in jobs:
        match_score = calculate_match_score(user, job)
        if match_score > 0:  # Only recommend jobs with a positive score
            job_data = {
                'job_title': job['job_title'],
                'company': job['company'],
                'location': job['location'],
                'job_type': job['job_type'],
                'required_skills': job['required_skills'],
                'experience_level': job['experience_level'],
                'match_score': match_score
            }
            scored_jobs.append(job_data)

    # Sort jobs by match score in descending order
    scored_jobs.sort(key=lambda x: x['match_score'], reverse=True)
    
    return scored_jobs

# Flask application setup
app = Flask(__name__)
CORS(app)

# Job recommendation API
@app.route('/recommendations', methods=['POST'])
def recommend_jobs():
    try:
        user_data = request.json
        recommendations = get_job_recommendations(user_data)
        return jsonify(recommendations), 200
    except Exception as e:
        return jsonify({'error': str(e)}), 500

if __name__ == '__main__':
    app.run(debug=True)

# Sample data insertion into MongoDB (for testing)
def insert_sample_jobs():
    jobs_collection.insert_many([
        {
            "job_title": "Backend Developer",
            "company": "Tech Solutions Inc.",
            "required_skills": ["Python", "Django", "REST APIs"],
            "location": "Remote",
            "job_type": "Full-Time",
            "experience_level": "Intermediate"
        },
        {
            "job_title": "Software Engineer",
            "company": "Innovative Tech Corp.",
            "required_skills": ["Python", "Microservices", "SQL"],
            "location": "New York",
            "job_type": "Full-Time",
            "experience_level": "Intermediate"
        },
        {
            "job_title": "Frontend Developer",
            "company": "Creative Designs LLC",
            "required_skills": ["HTML", "CSS", "JavaScript", "Vue.js"],
            "location": "New York",
            "job_type": "Part-Time",
            "experience_level": "Junior"
        },
        {
            "job_title": "Data Scientist",
            "company": "Data Analytics Corp.",
            "required_skills": ["Python", "Data Analysis", "Machine Learning"],
            "location": "Remote",
            "job_type": "Full-Time",
            "experience_level": "Intermediate"
        }
    ])

# Uncomment the line below to insert the sample jobs into MongoDB
# insert_sample_jobs()

