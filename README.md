# 🐾 PawMart

**PawMart** is a full-stack pet e-commerce platform to buy pets and pet products online. Customers browse pets, food, accessories and medicine, pay with Razorpay or cash on delivery, and book pet caretaking. Admins manage the catalog, orders, blogs and caretaking requests from a separate dashboard. A Gemini-powered chatbot answers pet-care questions.

Built with the **MERN stack** (MongoDB, Express, React, Node.js), plus a small Flask service for the chatbot.

---

## ✨ Features

### Customer app
- Browse **Pets, Foods, Accessories and Medicine** by category, with filters and search
- Product details, recommended products and an image slider on the home page
- **Cart** and quantity controls (Redux Toolkit + `react-use-cart`)
- **Checkout** with saved delivery addresses
- **Online payment with Razorpay** (server-side signature verification) or **Cash on Delivery**
- **Order history**, live order status and order cancellation
- **Pet caretaking requests**: submit a request with ID proof and track its approval status
- **Blogs** (video and article links) and a **photo gallery**
- Signup, login and profile management

### Admin dashboard
- Admin login with protected routes
- Dashboard with a **sales chart** (Chart.js)
- Add, update and delete **products** and **categories** (with image upload)
- View all **orders** and update their status
- View customer **addresses**
- Approve or reject **caretaking requests**
- Manage **blogs** and the **gallery**

### AI chatbot
- Flask service that uses **Google Gemini** to answer customer questions in a chat widget

---

## 🧱 Tech stack

| Layer | Technology |
|---|---|
| Customer frontend | React 18, Vite, Redux Toolkit, React Router, Axios, React-Toastify |
| Admin frontend | React 18, Vite, Redux Toolkit, PrimeReact, Chart.js, Axios |
| Backend API | Node.js, Express 4, Mongoose 8, Multer (uploads), bcryptjs, CORS |
| Database | MongoDB |
| Payments | Razorpay (Orders API + HMAC signature verification) |
| Chatbot | Python, Flask, Flask-CORS, Google Generative AI (Gemini) |

---

## 📁 Project structure

```
Pawmart/
├── Backend/            # Express + MongoDB REST API (port 5001)
│   ├── app.js
│   ├── routes/         # user, product, category, order, address, blog, gallery, admin
│   ├── controllers/    # request handlers
│   ├── models/         # Mongoose schemas
│   └── uploads/        # uploaded images (served statically)
├── user/               # Customer React app (Vite)
├── admin/              # Admin React app (Vite)
└── Chat-bot/           # Flask + Gemini chatbot (port 5000)
```

---

## 🚀 Getting started

### Prerequisites
- Node.js 18+ and npm
- MongoDB running locally (or a MongoDB Atlas connection string)
- Python 3.10+ (only for the chatbot)
- A Razorpay account (test mode is fine) and a Google Gemini API key

### 1. Clone
```bash
git clone https://github.com/11Aravind/Pawmart.git
cd Pawmart
```

### 2. Backend
```bash
cd Backend
npm install
```
Create `Backend/.env`:
```env
PORT=5001
RAZORPAY_KEY_ID=your_razorpay_key_id
RAZORPAY_SECRET=your_razorpay_key_secret
```
Start MongoDB, then run:
```bash
npm start
```
The API runs at `http://localhost:5001`. By default it connects to `mongodb://localhost:27017/petsworlds`.

### 3. Customer app
```bash
cd user
npm install
npm run dev
```
Open the URL Vite prints (usually `http://localhost:5173`).

### 4. Admin app
```bash
cd admin
npm install
npm run dev
```
Vite picks the next free port (usually `http://localhost:5174`).

> Both apps call the API at `http://localhost:5001/`. To change it, edit `src/API/api.js` in `user/` and `admin/`.

### 5. Chatbot (optional)
```bash
cd Chat-bot
python -m venv venv
# Windows: venv\Scripts\activate    macOS/Linux: source venv/bin/activate
pip install flask flask-cors python-dotenv google-generativeai
```
Create `Chat-bot/.env`:
```env
API_KEY=your_gemini_api_key
```
```bash
python app.py
```
The chatbot runs at `http://localhost:5000`.

### 6. Create the first admin
There is no admin signup endpoint, so insert one directly. First generate a password hash:
```bash
cd Backend
node -e "console.log(require('bcryptjs').hashSync('YourStrongPassword'))"
```
Then in `mongosh`:
```js
use petsworlds
db.admins.insertOne({ email: "admin@example.com", password: "<paste the hash here>" })
```
Log in to the admin app with that email and password.

---

## 🔌 API overview

Base URL: `http://localhost:5001`

| Area | Method and path | Purpose |
|---|---|---|
| User | `POST /api/user/signup`, `POST /api/user/login` | Register and log in |
| User | `GET /api/user/` | Get user details |
| User | `POST /api/user/address`, `GET /api/user/getAddress`, `GET /api/user/getAllAddress` | Delivery addresses |
| Caretaking | `POST /api/user/caretaking`, `GET /api/user/getCaretaking`, `GET /api/user/getallCaretaking`, `POST /api/user/updateCaretakeStatus` | Caretaking requests |
| Product | `GET /api/product`, `POST /api/product/save`, `POST /api/product/update`, `DELETE /api/product/:id` | Catalog |
| Category | `GET /api/category`, `POST /api/category/add`, `POST /api/category/update`, `DELETE /api/category/:id` | Categories |
| Order | `GET /api/order`, `GET /api/order/all` | User orders and all orders (admin) |
| Order | `POST /api/order/checkout` | Create a Razorpay order |
| Order | `POST /api/order/validate` | Verify the Razorpay payment signature |
| Order | `POST /api/order/cod` | Place a cash-on-delivery order |
| Order | `POST /api/order/updateStatus`, `POST /api/order/cancelOrder/:orderId` | Order status |
| Address | `DELETE /api/address/:addressId` | Delete an address |
| Blog | `GET /api/blog`, `POST /api/blog/add`, `POST /api/blog/update`, `DELETE /api/blog/:id` | Blogs |
| Gallery | `GET /api/gallery`, `POST /api/gallery/savegallery`, `DELETE /api/gallery/:id` | Gallery |
| Admin | `POST /api/admin/login` | Admin login |
| Chatbot | `POST /chat` (Flask, port 5000) | Ask the AI assistant |

---

## 💳 Payment flow (Razorpay)

1. Customer clicks **Pay** at checkout.
2. The frontend calls `POST /api/order/checkout`. The backend creates a Razorpay order and saves the order with a pending status.
3. Razorpay Checkout opens in the browser and the customer pays.
4. The frontend sends `razorpay_order_id`, `razorpay_payment_id` and `razorpay_signature` to `POST /api/order/validate`.
5. The backend recomputes the HMAC-SHA256 signature with the secret key. If it matches, the order is marked as successful.

For **Cash on Delivery**, the frontend calls `POST /api/order/cod` and the order is saved directly.

-------

## 🗂️ Data models

| Model | Main fields |
|---|---|
| `User` | name, email, password (hashed) |
| `Admin` | email, password (hashed) |
| `Category` | mainCategory (for example Pet), category (for example Dog), subCategory, image |
| `Product` | name, image, description, oldPrice, newPrice, status, category_id |
| `Order` | userId, addressId, items, totelamount, razorpayOrderId, paymentMode, status, order_message, dateOfOrder |
| `Address` | userId, name, mobileNo, address |
| `Caretaker` | userId, type, owner_name, proof, phone_no, address, pickup, deliver, status |
| `Blog` | category, link, description |
| `Gallery` | image |

---

## 🛣️ Roadmap

- [ ] Token-based authentication (JWT) and role checks on admin APIs
- [ ] Move API URLs into environment variables (`VITE_API_URL`)
- [ ] Cloud image storage instead of local `uploads/`
- [ ] Pagination and server-side search
- [ ] Automated tests and CI
- [ ] Docker setup for one-command startup
- [ ] Deployment guide

---

## 🤝 Contributing

Issues and pull requests are welcome. Fork the repo, create a branch, and open a pull request.

## 👤 Author

**Aravind** — [@11Aravind](https://github.com/11Aravind)
