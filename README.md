<header>

<!--// تغییر مدل User (mongoose)
const UserSchema = new mongoose.Schema({
  email: String,
  password: String,
  role: { type: String, enum: ['user', 'admin'], default: 'user' },
  balance: {
    USDT: { type: Number, default: 1000 },
    BTC: { type: Number, default: 0 },
    ETH: { type: Number, default: 0 },
  }
});// Middleware احراز هویت ادمین
const authAdmin = async (req, res, next) => {
  const token = req.headers.authorization;
  if (!token) return res.status(401).json({ error: 'No token' });
  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    const user = await User.findById(decoded.id);
    if (!user || user.role !== 'admin') return res.status(403).json({ error: 'Access denied' });
    req.user = user;
    next();
  } catch (err) {
    res.status(401).json({ error: 'Invalid token' });
  }
};// مشاهده همه کاربران
app.get('/api/admin/users', authAdmin, async (req, res) => {
  const users = await User.find().select('-password');
  res.json(users);
});

// تغییر نقش یا مسدود کردن کاربر
app.put('/api/admin/users/:id', authAdmin, async (req, res) => {
  const { role } = req.body; // مثلا role: 'admin' یا 'user'
  const user = await User.findById(req.params.id);
  if (!user) return res.status(404).json({ error: 'User not found' });
  if (role) user.role = role;
  await user.save();
  res.json({ message: 'User updated' });
});

// مشاهده همه سفارش‌ها
app.get('/api/admin/orders', authAdmin, async (req, res) => {
  const orders = await Order.find().sort({ createdAt: -1 });
  res.json(orders);
});

// لغو سفارش
app.put('/api/admin/orders/:id/cancel', authAdmin, async (req, res) => {
  const order = await Order.findById(req.params.id);
  if (!order) return res.status(404).json({ error: 'Order not found' });
  order.status = 'canceled';
  await order.save();
  res.json({ message: 'Order canceled' });
});

// مشاهده درخواست‌های برداشت
app.get('/api/admin/withdrawals', authAdmin, async (req, res) => {
  const withdrawals = await Withdrawal.find().sort({ createdAt: -1 });
  res.json(withdrawals);
});

// تایید یا رد برداشت
app.put('/api/admin/withdrawals/:id', authAdmin, async (req, res) => {
  const { status } = req.body; // 'approved' یا 'rejected'
  const withdrawal = await Withdrawal.findById(req.params.id);
  if (!withdrawal) return res.status(404).json({ error: 'Withdrawal not found' });
  withdrawal.status = status;
  await withdrawal.save();

  if (status === 'rejected') {
    // در صورت رد برداشت، موجودی کاربر باید بازگردانده شود
    const user = await User.findById(withdrawal.userId);
    if (user) {
      user.balance[withdrawal.asset] += withdrawal.amount;
      await user.save();
    }
  }db.users.insertOne({
  email: "admin@example.com",
  password: "<hashed_password>", // حتما پسورد رو هش کن
  role: "admin",
  balance: { USDT: 0, BTC: 0, ETH: 0 }
});const bcrypt = require('bcryptjs');
const hashedPassword = await bcrypt.hash('yourAdminPassword', 10);
console.log(hashedPassword);
PUT /api/admin/users/{userId}
Authorization: Bearer <admin_token>
Content-Type: application/json

{
  "role": "admin"
}
  res.json({ message: `Withdrawal ${status}` });
});
require('dotenv').config();
const mongoose = require('mongoose');
const bcrypt = require('bcryptjs');
const User = require('./models/User'); // مسیر به مدل User توی پروژه

async function createAdmin() {
  await mongoose.connect(process.env.MONGO_URI);
  const email = 'admin@example.com';
  const password = 'StrongPassword123';
  const hashed = await bcrypt.hash(password, 10);
  
  const exists = await User.findOne({ email });
  if (exists) {
    console.log('Admin already exists');
    process.exit();
  }

  const admin = new User({ email, password: hashed, role: 'admin' });
  await admin.save();
  console.log('Admin created:', email);
  process.exit();
}
admin-panel/
├── src/
│   ├── components/
│   │   ├── Login.jsx
│   │   ├── UsersList.jsx
│   │   ├── OrdersList.jsx
│   │   └── WithdrawalsList.jsx
│   ├── App.jsx
│   ├── api.js
│   └── index.js
├── package.json
└── ...
import axios from 'axios';

const API = axios.create({
  baseURL: 'http://localhost:5000/api/admin',
});

export const setToken = (token) => {
  if (token) {
    API.defaults.headers.common['Authorization'] = `Bearer ${token}`;
  } else {
    delete API.defaults.headers.common['Authorization'];
  }
};

export const loginAdmin = (email, password) =>
  axios.post('http://localhost:5000/api/login', { email, password });

export const fetchUsers = () => API.get('/users');
export const updateUserRole = (id, role) => API.put(`/users/${id}`, { role });

export const fetchOrders = () => API.get('/orders');
export const cancelOrder = (id) => API.put(`/orders/${id}/cancel`);

export const fetchWithdrawals = () => API.get('/withdrawals');
export const updateWithdrawalStatus = (id, status) =>
  API.put(`/withdrawals/${id}`, { status });
  import React, { useState } from 'react';
import { loginAdmin, setToken } from './api';

export default function Login({ onLogin }) {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [error, setError] = useState('');

  const handleSubmit = async (e) => {
    e.preventDefault();
    try {
      const res = await loginAdmin(email, password);
      const token = res.data.token;
      setToken(token);
      onLogin(token);
    } catch {
      setError('Login failed');
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="email" value={email} onChange={e => setEmail(e.target.value)}
        placeholder="Admin email" required
      />
      <input
        type="password" value={password} onChange={e => setPassword(e.target.value)}
        placeholder="Password" required
      />
      <button type="submit">Login</button>
      {error && <p>{error}</p>}
    </form>
  );
}
import React, { useEffect, useState } from 'react';
import { fetchUsers, updateUserRole } from './api';

export default function UsersList() {
  const [users, setUsers] = useState([]);

  useEffect(() => {
    fetchUsers().then(res => setUsers(res.data));
  }, []);

  const changeRole = (id, role) => {
    updateUserRole(id, role).then(() => {
      setUsers(users.map(u => u._id === id ? { ...u, role } : u));
    });
  };

  return (
    <div>
      <h2>Users</h2>
      <table>
        <thead><tr><th>Email</th><th>Role</th><th>Change Role</th></tr></thead>
        <tbody>
          {users.map(u => (
            <tr key={u._id}>
              <td>{u.email}</td>
              <td>{u.role}</td>
              <td>
                {u.role !== 'admin' && <button onClick={() => changeRole(u._id, 'admin')}>Make Admin</button>}
                {u.role !== 'user' && <button onClick={() => changeRole(u._id, 'user')}>Make User</button>}
              </td>
            </tr>
          ))}
        </tbody>
      </table>
    </div>
  );
}
npm install
npm start
این هم فایل زیپ کامل پروژه React پنل ادمین که خواستی:

admin-panel.zip


---

نکات نصب و اجرا:

1. از حالت فشرده خارجش کن


2. داخل پوشه admin-panel ترمینال باز کن


3. دستور بزن:



npm install
npm start

4. پروژه روی http://localhost:3000 اجرا می‌شه




---

createAdmin();
  <<< Author notes: Course header >>>
  Include a 1280×640 image, course title in sentence case, and a concise description in emphasis.
  In your repository settings: enable template repository, add your 1280×640 social image, auto delete head branches.
  Add your open source license, GitHub uses MIT license.
-->

# GitHub Pages

_Create a site or blog from your GitHub repositories with GitHub Pages._

</header>

<!--
  <<< Author notes: Step 1 >>>
  Choose 3-5 steps for your course.
  The first step is always the hardest, so pick something easy!
  Link to docs.github.com for further explanations.
  Encourage users to open new tabs for steps!
-->

## Step 1: Enable GitHub Pages

_Welcome to GitHub Pages and Jekyll :tada:!_

The first step is to enable GitHub Pages on this [repository](https://docs.github.com/en/get-started/quickstart/github-glossary#repository). When you enable GitHub Pages on a repository, GitHub takes the content that's on the main branch and publishes a website based on its contents.

### :keyboard: Activity: Enable GitHub Pages

1. Open a new browser tab, and work on the steps in your second tab while you read the instructions in this tab.
1. Under your repository name, click **Settings**.
1. Click **Pages** in the **Code and automation** section.
1. Ensure "Deploy from a branch" is selected from the **Source** drop-down menu, and then select `main` from the **Branch** drop-down menu.
1. Click the **Save** button.
1. Wait about _one minute_ then refresh this page (the one you're following instructions from). [GitHub Actions](https://docs.github.com/en/actions) will automatically update to the next step.
   > Turning on GitHub Pages creates a deployment of your repository. GitHub Actions may take up to a minute to respond while waiting for the deployment. Future steps will be about 20 seconds; this step is slower.
   > **Note**: In the **Pages** of **Settings**, the **Visit site** button will appear at the top. Click the button to see your GitHub Pages site.

<footer>

<!--
  <<< Author notes: Footer >>>
  Add a link to get support, GitHub status page, code of conduct, license link.
-->

---

Get help: [Post in our discussion board](https://github.com/orgs/skills/discussions/categories/github-pages) &bull; [Review the GitHub status page](https://www.githubstatus.com/)

&copy; 2023 GitHub &bull; [Code of Conduct](https://www.contributor-covenant.org/version/2/1/code_of_conduct/code_of_conduct.md) &bull; [MIT License](https://gh.io/mit)

</footer>
