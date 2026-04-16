

* Authentication (Admin + Employee)
* Employee management (add/edit)
* Self-service profile editing
* ⭐ 5-star performance rating
* Backend API + database
* Ready to expand into full HRMS

You can build everything else (payroll, leave, etc.) on top of this.

---

# ✅ **TECH STACK (Simple & Practical)**

* Frontend: React
* Backend: Node.js + Express
* Database: MongoDB
* Auth: JWT

---

# 📁 **PROJECT STRUCTURE**

```
hrms-portal/
├── backend/
│   ├── server.js
│   ├── models/
│   ├── routes/
│   ├── middleware/
│
├── frontend/
│   ├── src/
│   ├── components/
│   ├── pages/
```

---

# ⚙️ **BACKEND CODE (Node + Express)**

## 1. Install dependencies

```bash
npm init -y
npm install express mongoose cors jsonwebtoken bcryptjs
```

---

## 2. `server.js`

```javascript
const express = require("express");
const mongoose = require("mongoose");
const cors = require("cors");

const authRoutes = require("./routes/auth");
const employeeRoutes = require("./routes/employees");

const app = express();
app.use(cors());
app.use(express.json());

mongoose.connect("mongodb://127.0.0.1:27017/hrms");

app.use("/api/auth", authRoutes);
app.use("/api/employees", employeeRoutes);

app.listen(5000, () => console.log("Server running on port 5000"));
```

---

## 3. Employee Model (`models/Employee.js`)

```javascript
const mongoose = require("mongoose");

const employeeSchema = new mongoose.Schema({
  name: String,
  email: String,
  password: String,
  role: { type: String, default: "employee" },

  position: String,
  department: String,

  personalDetails: {
    phone: String,
    address: String
  },

  education: String,
  familyDetails: String,

  performance: {
    rating: { type: Number, default: 0 }, // 1–5 stars
    taskScore: Number,
    attendance: Number,
    training: Number
  }
});

module.exports = mongoose.model("Employee", employeeSchema);
```

---

## 4. Auth Routes (`routes/auth.js`)

```javascript
const router = require("express").Router();
const Employee = require("../models/Employee");
const bcrypt = require("bcryptjs");
const jwt = require("jsonwebtoken");

const SECRET = "secretkey";

// Register
router.post("/register", async (req, res) => {
  const hashed = await bcrypt.hash(req.body.password, 10);
  const user = await Employee.create({ ...req.body, password: hashed });
  res.json(user);
});

// Login
router.post("/login", async (req, res) => {
  const user = await Employee.findOne({ email: req.body.email });
  if (!user) return res.status(400).send("User not found");

  const valid = await bcrypt.compare(req.body.password, user.password);
  if (!valid) return res.status(400).send("Invalid password");

  const token = jwt.sign({ id: user._id }, SECRET);
  res.json({ token, user });
});

module.exports = router;
```

---

## 5. Employee Routes (`routes/employees.js`)

```javascript
const router = require("express").Router();
const Employee = require("../models/Employee");

// Get all employees
router.get("/", async (req, res) => {
  const employees = await Employee.find();
  res.json(employees);
});

// Add employee (Admin)
router.post("/", async (req, res) => {
  const emp = await Employee.create(req.body);
  res.json(emp);
});

// Update employee (Self-edit or Admin)
router.put("/:id", async (req, res) => {
  const emp = await Employee.findByIdAndUpdate(req.params.id, req.body, { new: true });
  res.json(emp);
});

// ⭐ Rate employee
router.put("/:id/rate", async (req, res) => {
  const { rating } = req.body; // 1–5
  const emp = await Employee.findByIdAndUpdate(
    req.params.id,
    { "performance.rating": rating },
    { new: true }
  );
  res.json(emp);
});

module.exports = router;
```

---

# 🎨 **FRONTEND (React)**

## 1. Install

```bash
npx create-react-app frontend
cd frontend
npm install axios react-router-dom
```

---

## 2. API Setup (`src/api.js`)

```javascript
import axios from "axios";

export default axios.create({
  baseURL: "http://localhost:5000/api"
});
```

---

## 3. Login Page (`pages/Login.js`)

```javascript
import { useState } from "react";
import API from "../api";

export default function Login() {
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");

  const login = async () => {
    const res = await API.post("/auth/login", { email, password });
    localStorage.setItem("user", JSON.stringify(res.data.user));
    window.location.href = "/dashboard";
  };

  return (
    <div>
      <h2>Login</h2>
      <input onChange={e => setEmail(e.target.value)} placeholder="Email" />
      <input type="password" onChange={e => setPassword(e.target.value)} />
      <button onClick={login}>Login</button>
    </div>
  );
}
```

---

## 4. Employee Dashboard (`pages/Dashboard.js`)

```javascript
import { useEffect, useState } from "react";
import API from "../api";

export default function Dashboard() {
  const [employees, setEmployees] = useState([]);

  useEffect(() => {
    API.get("/employees").then(res => setEmployees(res.data));
  }, []);

  return (
    <div>
      <h2>Employees</h2>

      {employees.map(emp => (
        <div key={emp._id} style={{ border: "1px solid #ccc", margin: 10 }}>
          <h3>{emp.name}</h3>
          <p>{emp.position}</p>

          ⭐ Rating: {emp.performance?.rating || 0}

          <StarRating empId={emp._id} />
        </div>
      ))}
    </div>
  );
}
```

---

## 5. ⭐ Star Rating Component

```javascript
import API from "../api";

export default function StarRating({ empId }) {
  const rate = async (value) => {
    await API.put(`/employees/${empId}/rate`, { rating: value });
    window.location.reload();
  };

  return (
    <div>
      {[1,2,3,4,5].map(star => (
        <span key={star} onClick={() => rate(star)} style={{ cursor: "pointer" }}>
          ⭐
        </span>
      ))}
    </div>
  );
}
```

--
