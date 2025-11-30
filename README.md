# 🎓 Alumni Bridge  Connecting Students & Alumni

Alumni Bridge is a **full stack web platform** designed to connect **students, alumni, faculty, and professionals** from the same college/university.  
It helps users **build meaningful connections, chat in real time, share opportunities, host events, and network smartly** for career growth.



## 🚀 Tech Stack

### **🖥 Frontend (Web)**
- HTML, CSS, JavaScript (Vanilla JS)
- REST API Integration
- Responsive UI Design

### **⚙ Backend (Java + Spring Boot)**
- Java, Spring Boot
- Spring Security (JWT Authentication)
- WebSocket (Real time Chat)
- REST API Architecture
- MySQL Database using JPA & Hibernate

### **🛢 Database**
- MySQL (Relational DB)
- JPA/Hibernate ORM

### **🔧 Tools & IDE**
- IntelliJ IDEA / VS Code
- Postman (API Testing)
- Git & GitHub (Version Control)
- Maven (Dependency Management)

---

## 🧠 Problem Statement

Many colleges do **not have a proper system to connect alumni with current students**.  
Because of that:
- Students struggle to get guidance, internships & job referrals.
- Alumni lose connection with their college.
- Faculty has **no centralized platform** to manage networks.

**Alumni Bridge solves this by creating a secure platform for all  Student, Alumni, Faculty & Admin.**

---

## ⭐ Key Features

| Feature | Description |
|--------|-------------|
| 🔐 Authentication | Secure login, JWT based authorization |
| 💬 Real-Time Chat | WebSocket based live chat system |
| 🧑‍🎓 Student & Alumni Profiles | Education, skills, experience |
| 🔍 Search Users | Search by name, batch, profession, skills |
| 🤝 Connection Requests | Send & accept connection requests |
| 📅 Event Management | Create & join events/webinars |
| 📤 API Driven System | REST-based scalable architecture |
| 🏗 Scalable Backend | Modular & maintainable Java backend |



## 📂 Project Structure
alumni__bridge/
│── backend/ # Spring Boot Project
│ │── config/ # CORS & WebSocket configs
│ │── controller/ # REST Controllers
│ │── dto/ # Data Transfer Objects
│ │── entity/ # JPA Entities (Database Tables)
│ │── repository/ # JPA Repositories
│ │── security/ # JWT Security + User Details
│ │── service/ # Business Logic Layer
│ │── resources/ # application.properties + SQL
│
│── frontend/
│ │── css/ # UI Styling Files
│ │── js/ # API + Page Logic
│ │── html/ # UI Pages
│ └── index.html
│
└── README.md


---

## 🏃 How to Run Backend (Spring Boot)

```bash
cd alumni__bridge/backend
mvn spring boot:run

🌐 How to Run Frontend

Simply open:

alumni__bridge/frontend/index.html


(No npm or yarn required – pure HTML/CSS/JS)

🔌 API Endpoints (Examples)
Method	Endpoint	Description
POST	/api/auth/register	Register new user
POST	/api/auth/login	Login & get JWT token
GET	/api/users/all	Fetch all users
POST	/api/request/send	Send connection request
GET	/api/events/list	Get all events
🧪 API Testing (Postman)

Import your API endpoints in Postman

First call /api/auth/login

Copy JWT Token

Go to Authorization → Bearer Token

Paste token and test secured APIs

📌 Future Enhancements

AI-based Alumni Recommendation System

Resume Builder & Job Portal

LinkedIn style Feed + Posts

College wide Notification System

Mobile App with React Native

👨‍💻 Author

👤 Shivam Tripathi
B.Tech CSE | Full Stack  Java Developer
📧 Email: s.shivamtripathi13@gmail.com
🔗 LinkedIn: linkedin.com/in/shivam-tripathi-b14141238

💻 GitHub: github.com/Shivam123Tripathi

⭐ Support

If you like this project, support by ⭐ starring the repo!

git clone https://github.com/Shivam123Tripathi/alumni_bridge.git

🚀 “Connecting Students to Opportunities, Alumni to Legacy.”


