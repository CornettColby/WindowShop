# WindowShop
Appointment Scheduling Platform
// server.js
const express = require('express');
const app = express();
const mongoose = require('mongoose');
const bcrypt = require('bcrypt');
const jwt = require('jsonwebtoken');
const cors = require('cors');

// Middleware
app.use(cors());
app.use(express.json());

// Connect DB
mongoose.connect('mongodb://localhost/booking', { useNewUrlParser: true });

// User model
const User = mongoose.model('User', new mongoose.Schema({
  email: String,
  password: String,
  role: { type: String, default: 'user' }
}));

// Appointment model
const Appointment = mongoose.model('Appointment', new mongoose.Schema({
  time: String,
  bookedBy: { type: mongoose.Schema.Types.ObjectId, ref: 'User' }
}));

// Signup
app.post('/signup', async (req, res) => {
  const hashed = await bcrypt.hash(req.body.password, 10);
  const user = new User({ email: req.body.email, password: hashed });
  await user.save();
  res.sendStatus(201);
});

// Login
app.post('/login', async (req, res) => {
  const user = await User.findOne({ email: req.body.email });
  if (user && await bcrypt.compare(req.body.password, user.password)) {
    const token = jwt.sign({ userId: user._id, role: user.role }, 'secret');
    res.json({ token });
  } else {
    res.status(401).send('Unauthorized');
  }
});

// Get available slots
app.get('/appointments', async (req, res) => {
  const appointments = await Appointment.find({ bookedBy: null });
  res.json(appointments);
});

// Book a slot
app.post('/book', async (req, res) => {
  const { appointmentId, userId } = req.body;
  await Appointment.findByIdAndUpdate(appointmentId, { bookedBy: userId });
  res.sendStatus(200);
});

// Start
app.listen(3000, () => console.log('Server running on http://localhost:3000'));
