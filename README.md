# PROJECT
Project Structure
1. Backend (Express.js)
Directory Structure
Copy
/backend
  ├── /config
  ├── /controllers
  ├── /models
  ├── /routes
  ├── /middleware
  ├── /utils
  ├── server.js
  └── package.json
Sample Code
1.1. server.js

Copy
const express = require('express');
const bodyParser = require('body-parser');
const cors = require('cors');
const app = express();
const PORT = process.env.PORT || 5000;

app.use(cors());
app.use(bodyParser.json());

// Import routes
const userRoutes = require('./routes/userRoutes');
const storeRoutes = require('./routes/storeRoutes');
const ratingRoutes = require('./routes/ratingRoutes');

// Use routes
app.use('/api/users', userRoutes);
app.use('/api/stores', storeRoutes);
app.use('/api/ratings', ratingRoutes);

app.listen(PORT, () => {
  console.log(`Server is running on port ${PORT}`);
});
1.2. routes/userRoutes.js

Copy
const express = require('express');
const router = express.Router();
const { signup, login, updatePassword } = require('../controllers/userController');

// User routes
router.post('/signup', signup);
router.post('/login', login);
router.put('/:id', updatePassword);

module.exports = router;
1.3. controllers/userController.js

Copy
const User = require('../models/User');
const bcrypt = require('bcrypt');
const jwt = require('jsonwebtoken');

// Signup
exports.signup = async (req, res) => {
  const { name, email, password, address } = req.body;
  const hashedPassword = await bcrypt.hash(password, 10);
  const newUser = new User({ name, email, password: hashedPassword, address });

  try {
    await newUser.save();
    res.status(201).json({ message: 'User created successfully' });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
};

// Login
exports.login = async (req, res) => {
  const { email, password } = req.body;
  const user = await User.findOne({ email });

  if (user && await bcrypt.compare(password, user.password)) {
    const token = jwt.sign({ id: user._id, role: user.role }, 'secret_key');
    res.json({ token });
  } else {
    res.status(401).json({ message: 'Invalid credentials' });
  }
};

// Update Password
exports.updatePassword = async (req, res) => {
  const { password } = req.body;
  const hashedPassword = await bcrypt.hash(password, 10);
  
  try {
    await User.findByIdAndUpdate(req.params.id, { password: hashedPassword });
    res.json({ message: 'Password updated successfully' });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
};
1.4. models/User.js

Copy
const mongoose = require('mongoose');

const userSchema = new mongoose.Schema({
  name: { type: String, required: true, minlength: 20, maxlength: 60 },
  email: { type: String, required: true, unique: true },
  password: { type: String, required: true },
  address: { type: String, maxlength: 400 },
  role: { type: String, enum: ['admin', 'normal', 'store_owner'], required: true }
});

module.exports = mongoose.model('User', userSchema);
2. Frontend (React.js)
Directory Structure
Copy
/frontend
  ├── /src
  │   ├── /components
  │   ├── /pages
  │   ├── /services
  │   ├── App.js
  │   ├── index.js
  │   └── App.css
  └── package.json
Sample Code
2.1. App.js

Copy
import React from 'react';
import { BrowserRouter as Router, Route, Switch } from 'react-router-dom';
import Signup from './pages/Signup';
import Login from './pages/Login';
import Dashboard from './pages/Dashboard';

function App() {
  return (
    <Router>
      <Switch>
        <Route path="/signup" component={Signup} />
        <Route path="/login" component={Login} />
        <Route path="/dashboard" component={Dashboard} />
      </Switch>
    </Router>
  );
}

export default App;
2.2. pages/Signup.js

Copy
import React, { useState } from 'react';
import axios from 'axios';

const Signup = () => {
  const [formData, setFormData] = useState({ name: '', email: '', password: '', address: '' });

  const handleChange = (e) => {
    setFormData({ ...formData, [e.target.name]: e.target.value });
  };

  const handleSubmit = async (e) => {
    e.preventDefault();
    try {
      await axios.post('http://localhost:5000/api/users/signup', formData);
      alert('User registered successfully');
    } catch (error) {
      alert('Error registering user');
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <input name="name" onChange={handleChange} placeholder="Name" required />
      <input name="email" onChange={handleChange} placeholder="Email" required />
      <input name="password" type="password" onChange={handleChange} placeholder="Password" required />
      <input name="address" onChange={handleChange} placeholder="Address" />
      <button type="submit">Sign Up</button>
    </form>
  );
};

export default Signup;
3. Setting Up the Environment
Backend Setup:

Navigate to the /backend directory.
Run npm init -y to create a package.json.
Install dependencies: npm install express mongoose bcrypt jsonwebtoken cors body-parser.
Frontend Setup:

Navigate to the /frontend directory.
Create a new React app: npx create-react-app ..
Install Axios: npm install axios react-router-dom.
4. Running the Project
Backend:
Start the server: node server.js.
Frontend:
Start the React app: npm start.
Conclusion
This project structure and sample code provide a foundational setup for your web application. You can expand upon it by adding more features, enhancing the user interface, and implementing additional functionalities as per the requirements. If you have further questions or need specific implementations, feel free to ask!
