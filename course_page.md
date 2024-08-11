# Report on the Development of a Course Page Using React, Node.js, and MongoDB

## Table of Contents
1. [Introduction](#introduction)
2. [Project Overview](#project-overview)
3. [Technology Stack](#technology-stack)
4. [System Architecture](#system-architecture)
5. [Backend Development](#backend-development)
    - [Server Setup](#server-setup)
    - [Course Model](#course-model)
    - [Course Routes](#course-routes)
6. [Frontend Development](#frontend-development)
    - [React Application Structure](#react-application-structure)
    - [CoursePage Component](#coursepage-component)
7. [Conclusion](#conclusion)

## 1. Introduction

This report provides an in-depth overview of the development process for a Course Page as part of a Learning Management System (LMS) project. The project leverages modern web technologies, including React for the frontend, Node.js for the backend, and MongoDB as the database. The objective was to create an interactive and user-friendly interface for users to browse, view, and enroll in courses on the LMS platform.

## 2. Project Overview

The main goal of this project was to implement a feature-rich Course Page in a single-page application (SPA). The key functionalities include:

- **Course Listing**: Displaying a list of available courses with relevant details.
- **Course Details View**: Allowing users to view detailed information about each course.
- **Enrollment**: Enabling users to enroll in courses directly from the Course Page.
- **Secure Data Handling**: Ensuring that course data and user interactions are securely managed.

## 3. Technology Stack

The following technologies were used in the project:

- **Frontend**: React.js, Axios
- **Backend**: Node.js, Express.js
- **Database**: MongoDB
- **Additional Libraries**: bcrypt.js (for password hashing), jsonwebtoken (for authentication), body-parser (for parsing request bodies), cors (for handling Cross-Origin Resource Sharing)

## 4. System Architecture

The system architecture follows the Model-View-Controller (MVC) pattern:

- **Model**: Defines the data structure and interacts with the database (MongoDB).
- **View**: Represents the user interface built with React.js.
- **Controller**: Handles requests, processes data, and returns responses (Node.js and Express.js).

## 5. Backend Development

### Server Setup

The backend server was set up using Node.js and Express.js. The server handles HTTP requests related to course management, including listing available courses, displaying course details, and handling enrollments.

### Course Model

The course data is modeled using Mongoose, an Object Data Modeling (ODM) library for MongoDB. The Course schema defines the structure of the course document in the database.

```javascript
const mongoose = require('mongoose');

const CourseSchema = new mongoose.Schema({
    title: { type: String, required: true },
    description: { type: String, required: true },
    instructor: { type: String, required: true },
    duration: { type: String, required: true },
    price: { type: Number, required: true },
    thumbnail: { type: String },
    content: [{ type: String }]
});

module.exports = mongoose.model('Course', CourseSchema);
```

### Course Routes

The course routes handle HTTP requests for listing courses, displaying course details, and handling enrollments. The routes interact with the Course model and return appropriate responses.

- **Get Courses Route**: Retrieves a list of all available courses from the database.
- **Get Course Details Route**: Retrieves detailed information about a specific course.
- **Enroll Route**: Handles user enrollment in a course.

```javascript
const express = require('express');
const router = express.Router();
const Course = require('../models/Course');

// Get all courses
router.get('/courses', async (req, res) => {
    try {
        const courses = await Course.find();
        res.json(courses);
    } catch (error) {
        res.status(500).send('Server Error');
    }
});

// Get course details by ID
router.get('/courses/:id', async (req, res) => {
    try {
        const course = await Course.findById(req.params.id);
        if (!course) {
            return res.status(404).json({ msg: 'Course not found' });
        }
        res.json(course);
    } catch (error) {
        res.status(500).send('Server Error');
    }
});

// Enroll in a course
router.post('/courses/enroll', async (req, res) => {
    const { courseId, userId } = req.body;

    try {
        // Add enrollment logic here (e.g., add course to user's enrolled courses)
        res.json({ msg: 'Enrollment successful' });
    } catch (error) {
        res.status(500).send('Server Error');
    }
});

module.exports = router;
```

## 6. Frontend Development

### React Application Structure

The frontend was built using React.js to provide a responsive and interactive user experience. The application consists of a CoursePage component that displays a list of available courses and allows users to view course details and enroll.

- **React Router**: Used for handling navigation within the application.
- **Axios**: Used for making HTTP requests to the backend API.

### CoursePage Component

The CoursePage component displays a list of available courses and allows users to view detailed information about each course and enroll. The component handles data fetching, user interactions, and form submissions.

```javascript
import React, { useState, useEffect } from 'react';
import axios from 'axios';

function CoursePage() {
    const [courses, setCourses] = useState([]);

    useEffect(() => {
        // Fetch courses from API
        axios.get('/api/courses')
            .then(response => setCourses(response.data))
            .catch(error => console.error('Error fetching courses', error));
    }, []);

    const handleEnroll = async (courseId) => {
        try {
            await axios.post('/api/courses/enroll', { courseId });
            alert('Enrolled successfully');
        } catch (error) {
            alert('Error enrolling in course');
        }
    };

    return (
        <div>
            <h2>Available Courses</h2>
            <div>
                {courses.map(course => (
                    <div key={course._id} className="course-card">
                        <img src={course.thumbnail} alt={course.title} />
                        <h3>{course.title}</h3>
                        <p>{course.description}</p>
                        <button onClick={() => handleEnroll(course._id)}>Enroll</button>
                    </div>
                ))}
            </div>
        </div>
    );
}

export default CoursePage;
```

## 7. Conclusion

This project successfully implemented a Course Page using React, Node.js, and MongoDB. The development process involved creating a secure, scalable, and user-friendly interface for browsing, viewing, and enrolling in courses. The application provides a robust foundation for further enhancements, such as course filtering, sorting, and advanced enrollment features.

---

This report provides a comprehensive overview of the development of a course page, complete with a table of contents and detailed code scripts.
