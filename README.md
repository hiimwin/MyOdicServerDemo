📌 MyOidcServerDemo

This project is a simple demo to understand how OpenID Connect (OIDC) works internally, combined with CI/CD automation using Jenkins and Docker.

⚠️ Important:
This is a learning project only.
It is intentionally not using Clean Architecture or full SOLID design.
The goal is to:

Understand OIDC flow step-by-step
Practice CI/CD automation
Learn Docker containerization
🎯 Objectives

This project helps you:

🔐 Understand OIDC Authentication Flow
🔄 Simulate login & token flow manually
🍪 Understand cookie-based session handling
⚙️ Practice:
Jenkins pipeline
Docker build & run
CI/CD automation
🏗️ Project Structure
MyOdicServer/
 ├── OidcServer/        # OIDC Provider (Auth Server)
 ├── OidcWebClient/     # Client (Web App)
 ├── MyOdicServer.slnx  # Solution file

 ├── Dockerfile.server
 ├── Dockerfile.client
 ├── docker-compose.yml
 ├── docker-compose.dev.yml

 ├── Jenkinsfile        # CI/CD pipeline
 ├── README.md
🔄 Application Behavior
🧑‍💻 Login Flow
User opens Web Client
User enters a username
System checks:
✅ If username matches hardcoded value → continue OIDC flow
❌ If not → return "User not found"
🔐 Authentication Process
If valid user:
Redirect to OIDC Server
Server authenticates user
Issue authentication result
Redirect back to client
👤 After Login
User is redirected to Profile page
System stores session using cookies
🍪 Cookie Behavior
If user does not logout → session persists
If user logs out:
Cookies are cleared
Session is removed
🚀 Run Locally
1. Clone project
git clone https://github.com/hiimwin/MyOidcServerDemo.git
cd MyOidcServerDemo
2. Run Server
cd MyOdicServer/OidcServer
dotnet run
3. Run Client
cd MyOdicServer/OidcWebClient
dotnet run
4. Test
Open client app
Enter username:
✅ Correct → login success
❌ Wrong → "User not found"
After login:
Access profile
Logout to clear cookies
🐳 Docker Setup

Build & run with Docker:

docker-compose -f docker-compose.dev.yml up --build

Includes:

OIDC Server container
Web Client container
⚙️ CI/CD with Jenkins

This project includes a Jenkins pipeline to automate:

🔁 Pipeline Flow
Pull source code from GitHub
Build .NET project
Build Docker images:
Server image
Client image
Run containers
Deploy updated version
📄 Jenkinsfile Purpose

The Jenkinsfile is used to:

Automate build process
Integrate with Docker
Enable continuous deployment
❗ Limitations

This project does NOT include:

Clean Architecture
Full SOLID principles
Database (uses hardcoded user)
Advanced OIDC features:
PKCE
Refresh Token
Role/Claim management
Production-level security
💡 Why This Project Exists

This project is designed to:

👉 Learn by building from scratch instead of using libraries

It helps you:

Understand OIDC flow deeply
See how authentication really works
Practice real-world DevOps workflow (CI/CD)
🚀 Future Improvements
Replace hardcoded user with database
Add JWT token handling
Implement PKCE
Apply Clean Architecture
Deploy to cloud (AWS / Azure)

⭐ Final Note

This project is best used as:

🧪 Learning OIDC playground
⚙️ CI/CD practice project
🐳 Docker hands-on demo
