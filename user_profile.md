# Report on the Development of a User Profile Page Using React, Node.js, and MongoDB

## Table of Contents
1. [Introduction](#introduction)
2. [Project Overview](#project-overview)
3. [Technology Stack](#technology-stack)
4. [System Architecture](#system-architecture)
5. [Backend Development](#backend-development)
    - [Server Setup](#server-setup)
    - [User Model](#user-model)
    - [User Routes](#user-routes)
6. [Frontend Development](#frontend-development)
    - [React Application Structure](#react-application-structure)
    - [UserProfile Component](#userprofile-component)
7. [Conclusion](#conclusion)

## 1. Introduction

This report provides an in-depth overview of the development process for a User Profile page as part of a Learning Management System (LMS) project. The project leverages modern web technologies, including React for the frontend, Node.js for the backend, and MongoDB as the database. The objective was to create a user-friendly and secure interface for users to view and update their profile information on the LMS platform.

## 2. Project Overview

The main goal of this project was to implement a feature-rich User Profile page in a single-page application (SPA). The key functionalities include:

- **User Information Display**: Showing the user's details such as username, email, and profile picture.
- **Profile Update**: Allowing users to update their information, including changing their profile picture.
- **Secure Data Handling**: Ensuring that sensitive data such as email and passwords are securely handled and stored.

## 3. Technology Stack

The following technologies were used in the project:

- **Frontend**: React.js, Axios
- **Backend**: Node.js, Express.js
- **Database**: MongoDB
- **Additional Libraries**: bcrypt.js (for password hashing), multer (for file uploads), jsonwebtoken (for authentication), body-parser (for parsing request bodies), cors (for handling Cross-Origin Resource Sharing)

## 4. System Architecture

The system architecture follows the Model-View-Controller (MVC) pattern:

- **Model**: Defines the data structure and interacts with the database (MongoDB).
- **View**: Represents the user interface built with React.js.
- **Controller**: Handles requests, processes data, and returns responses (Node.js and Express.js).

## 5. Backend Development

### Server Setup

The backend server was set up using Node.js and Express.js. The server handles HTTP requests related to user profile management, including viewing and updating profile information.

### User Model

The user data is modeled using Mongoose, an Object Data Modeling (ODM) library for MongoDB. The User schema defines the structure of the user document in the database.

```javascript
const mongoose = require('mongoose');
const bcrypt = require('bcryptjs');

const UserSchema = new mongoose.Schema({
    username: { type: String, required: true },
    email: { type: String, required: true, unique: true },
    password: { type: String, required: true },
    profilePicture: { type: String }
});

UserSchema.pre('save', async function (next) {
    if (!this.isModified('password')) return next();
    this.password = await bcrypt.hash(this.password, 10);
    next();
});

module.exports = mongoose.model('User', UserSchema);
```

### User Routes

The user routes handle HTTP requests for viewing and updating the user profile. The routes interact with the User model and return appropriate responses.

- **Get Profile Route**: Retrieves the user's profile information from the database.
- **Update Profile Route**: Updates the user's profile information, including uploading a new profile picture.

```javascript
const express = require('express');
const router = express.Router();
const User = require('../models/User');
const multer = require('multer');

// Multer setup for file upload
const storage = multer.diskStorage({
    destination: function (req, file, cb) {
        cb(null, 'uploads/');
    },
    filename: function (req, file, cb) {
        cb(null, file.fieldname + '-' + Date.now());
    }
});
const upload = multer({ storage: storage });

// Get user profile
router.get('/profile', async (req, res) => {
    try {
        const user = await User.findById(req.user.id).select('-password');
        res.json(user);
    } catch (error) {
        res.status(500).send('Server Error');
    }
});

// Update user profile
router.post('/profile', upload.single('profilePicture'), async (req, res) => {
    const { username, email } = req.body;
    const profilePicture = req.file ? req.file.path : undefined;

    try {
        const user = await User.findById(req.user.id);

        if (username) user.username = username;
        if (email) user.email = email;
        if (profilePicture) user.profilePicture = profilePicture;

        await user.save();
        res.json(user);
    } catch (error) {
        res.status(500).send('Server Error');
    }
});

module.exports = router;
```

## 6. Frontend Development

### React Application Structure

The frontend was built using React.js to provide a responsive and interactive user experience. The application consists of a UserProfile component that displays and updates the user's information.

- **React Router**: Used for handling navigation within the application.
- **Axios**: Used for making HTTP requests to the backend API.

### UserProfile Component

The UserProfile component displays the user's profile information and allows the user to update their details. The component handles form submission, validates input data, and sends the data to the backend API.

```javascript
import React, { useState, useEffect } from 'react';
import axios from 'axios';

function UserProfile() {
    const [userData, setUserData] = useState({ username: '', email: '', profilePicture: '' });
    const [selectedFile, setSelectedFile] = useState(null);

    useEffect(() => {
        // Fetch user data from API
        axios.get('/api/users/profile')
            .then(response => setUserData(response.data))
            .catch(error => console.error('Error fetching user data', error));
    }, []);

    const handleChange = (e) => {
        setUserData({ ...userData, [e.target.name]: e.target.value });
    };

    const handleFileChange = (e) => {
        setSelectedFile(e.target.files[0]);
    };

    const handleSubmit = async (e) => {
        e.preventDefault();
        const formData = new FormData();
        formData.append('username', userData.username);
        formData.append('email', userData.email);
        if (selectedFile) {
            formData.append('profilePicture', selectedFile);
        }

        try {
            await axios.post('/api/users/profile', formData, {
                headers: {
                    'Content-Type': 'multipart/form-data'
                }
            });
            alert('Profile updated successfully');
        } catch (error) {
            alert('Error updating profile');
        }
    };

    return (
        <div>
            <h2>User Profile</h2>
            <form onSubmit={handleSubmit}>
                <input type="text" name="username" placeholder="Username" value={userData.username} onChange={handleChange} required />
                <input type="email" name="email" placeholder="Email" value={userData.email} onChange={handleChange} required />
                <input type="file" name="profilePicture" onChange={handleFileChange} />
                <button type="submit">Update Profile</button>
            </form>
        </div>
    );
}

export default UserProfile;
```

## 7. Conclusion

This project successfully implemented a User Profile page using React, Node.js, and MongoDB. The development process involved creating secure, scalable, and efficient code for handling user profile management, including viewing and updating profile information. The application provides a solid foundation that can be expanded to include additional features, such as social media integration, account deletion, and advanced profile customization options.

---

This report, with a table of contents, provides a comprehensive overview of the development of a user profile page, making it easier for readers to navigate and understand the project.
