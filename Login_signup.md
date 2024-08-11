**Login and Signup** page with database connectivity using **React**, **Node.js**, and **MongoDB**. This solution includes both the backend and frontend parts.

### 1. Backend Setup (Node.js with Express and MongoDB)
First, create a Node.js application and install the required dependencies.

```bash
mkdir lms-backend
cd lms-backend
npm init -y
npm install express mongoose bcryptjs jsonwebtoken cors body-parser
```

#### `server.js` (Entry Point)
Create a `server.js` file to set up the Express server and connect to MongoDB.

```javascript
const express = require('express');
const mongoose = require('mongoose');
const cors = require('cors');
const bodyParser = require('body-parser');
const userRoutes = require('./routes/user');

const app = express();

// Middleware
app.use(cors());
app.use(bodyParser.json());

// MongoDB Connection
mongoose.connect('mongodb://localhost:27017/lms', { useNewUrlParser: true, useUnifiedTopology: true })
    .then(() => console.log('MongoDB Connected'))
    .catch(err => console.log(err));

// Routes
app.use('/api/users', userRoutes);

// Start the server
const PORT = process.env.PORT || 5000;
app.listen(PORT, () => console.log(`Server running on port ${PORT}`));
```

#### `models/User.js` (User Model)
Create a `models` folder and inside it, create a `User.js` file to define the User schema.

```javascript
const mongoose = require('mongoose');
const bcrypt = require('bcryptjs');

const UserSchema = new mongoose.Schema({
    username: {
        type: String,
        required: true,
        unique: true
    },
    email: {
        type: String,
        required: true,
        unique: true
    },
    password: {
        type: String,
        required: true
    }
});

// Hash password before saving
UserSchema.pre('save', async function(next) {
    if (!this.isModified('password')) return next();
    const salt = await bcrypt.genSalt(10);
    this.password = await bcrypt.hash(this.password, salt);
    next();
});

module.exports = mongoose.model('User', UserSchema);
```

#### `routes/user.js` (User Routes)
Create a `routes` folder and inside it, create a `user.js` file for user-related routes.

```javascript
const express = require('express');
const bcrypt = require('bcryptjs');
const jwt = require('jsonwebtoken');
const User = require('../models/User');

const router = express.Router();

// Signup Route
router.post('/signup', async (req, res) => {
    const { username, email, password } = req.body;
    try {
        const user = new User({ username, email, password });
        await user.save();
        res.status(201).json({ message: 'User registered successfully' });
    } catch (err) {
        res.status(400).json({ error: 'User registration failed' });
    }
});

// Login Route
router.post('/login', async (req, res) => {
    const { email, password } = req.body;
    try {
        const user = await User.findOne({ email });
        if (!user) return res.status(400).json({ error: 'Invalid credentials' });

        const isMatch = await bcrypt.compare(password, user.password);
        if (!isMatch) return res.status(400).json({ error: 'Invalid credentials' });

        const token = jwt.sign({ userId: user._id }, 'your_jwt_secret', { expiresIn: '1h' });
        res.json({ token });
    } catch (err) {
        res.status(500).json({ error: 'Server error' });
    }
});

module.exports = router;
```

### 2. Frontend Setup (React)
Now, create the React application for the frontend.

```bash
npx create-react-app lms-frontend
cd lms-frontend
npm install axios
```

#### `src/App.js`
Replace the content of `App.js` to handle routing between the Login and Signup pages.

```javascript
import React from 'react';
import { BrowserRouter as Router, Route, Routes } from 'react-router-dom';
import Login from './components/Login';
import Signup from './components/Signup';

function App() {
    return (
        <Router>
            <div className="App">
                <Routes>
                    <Route path="/login" element={<Login />} />
                    <Route path="/signup" element={<Signup />} />
                </Routes>
            </div>
        </Router>
    );
}

export default App;
```

#### `src/components/Signup.js`
Create a `components` folder, and inside it, create a `Signup.js` file for the Signup form.

```javascript
import React, { useState } from 'react';
import axios from 'axios';

function Signup() {
    const [formData, setFormData] = useState({
        username: '',
        email: '',
        password: ''
    });

    const handleChange = e => {
        setFormData({ ...formData, [e.target.name]: e.target.value });
    };

    const handleSubmit = async e => {
        e.preventDefault();
        try {
            const response = await axios.post('http://localhost:5000/api/users/signup', formData);
            alert(response.data.message);
        } catch (err) {
            alert('Error signing up');
        }
    };

    return (
        <form onSubmit={handleSubmit}>
            <h2>Signup</h2>
            <input type="text" name="username" placeholder="Username" value={formData.username} onChange={handleChange} required />
            <input type="email" name="email" placeholder="Email" value={formData.email} onChange={handleChange} required />
            <input type="password" name="password" placeholder="Password" value={formData.password} onChange={handleChange} required />
            <button type="submit">Signup</button>
        </form>
    );
}

export default Signup;
```

#### `src/components/Login.js`
Create a `Login.js` file for the Login form.

```javascript
import React, { useState } from 'react';
import axios from 'axios';

function Login() {
    const [formData, setFormData] = useState({
        email: '',
        password: ''
    });

    const handleChange = e => {
        setFormData({ ...formData, [e.target.name]: e.target.value });
    };

    const handleSubmit = async e => {
        e.preventDefault();
        try {
            const response = await axios.post('http://localhost:5000/api/users/login', formData);
            alert('Login successful');
            localStorage.setItem('token', response.data.token);
        } catch (err) {
            alert('Invalid credentials');
        }
    };

    return (
        <form onSubmit={handleSubmit}>
            <h2>Login</h2>
            <input type="email" name="email" placeholder="Email" value={formData.email} onChange={handleChange} required />
            <input type="password" name="password" placeholder="Password" value={formData.password} onChange={handleChange} required />
            <button type="submit">Login</button>
        </form>
    );
}

export default Login;
```

### 3. Run the Application
- **Backend**: Run the server with `node server.js`.
- **Frontend**: Run the React application with `npm start`.

### Summary
This solution provides a basic implementation of a Login and Signup system using React for the frontend, Node.js for the backend, and MongoDB for the database. The backend handles user authentication, while the frontend provides the UI for users to interact with.
