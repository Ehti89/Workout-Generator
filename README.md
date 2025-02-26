# Workout-Generator
from flask import Flask, redirect, url_for, render_template, request, session, flash
from datetime import timedelta
from flask_sqlalchemy import SQLAlchemy
from flask_session import Session
from flask_migrate import Migrate

# Create an instance of the webapp
app = Flask(__name__)

# Set a secret key for session management
app.secret_key = "1234567890987654321qwertyuiopasdfghjklzxcvbnm"

# Configure the SQLAlchemy database URI and track modifications setting
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///UserInformation.sqlite3'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False

# Set the session type and session lifetime
app.config["SESSION_TYPE"] = "filesystem"
app.permanent_session_lifetime = timedelta(minutes=30)

# Initialize the SQLAlchemy object
db = SQLAlchemy(app)

# Initialize the Session extension
Session(app)

# Initialize Flask-Migrate
migrate = Migrate(app, db)

# Define the UserInformation model
class UserInformation(db.Model):
    _id = db.Column("id", db.Integer, primary_key=True)
    email = db.Column(db.String(100))
    password = db.Column(db.String(100))
    content = db.Column(db.String(1000))
    content = db.Column(db.String(1000))
    
    def __init__(self, email, password, content,workouts):
        self.email = email
        self.password = password
        self.content = content
        self.workouts = workouts

# Route for the landing page
@app.route("/")
def landingpage():
    return render_template("landingpage.html")

# Route for the login page
@app.route("/login", methods=["POST", "GET"])
def login():
    session.permanent = True
    if request.method == "POST":
        email = request.form["email"]
        session["email"] = email

        password = request.form["password"]
        session["password"] = password

        # Check for empty password
        if len(password) == 0:
            flash("Enter password!", "info")
            return redirect(url_for("login"))

        existing_user = UserInformation.query.filter_by(email=email).first()

        # Check for existing user and verify password
        if existing_user:
            if existing_user.password == password:
                flash("Successfully logged in", "info")
                return redirect(url_for("homepage"))
            else:
                flash("Incorrect password. Try again.", "danger")
        else:
            flash("User not found. Please sign up.", "danger")
            return redirect(url_for("signup"))
    return render_template("login.html")

# Route for the signup page
@app.route("/signup", methods=["POST", "GET"])
def signup():
    if request.method == "POST":
        email_signup = request.form["email_signup"]
        session["email_signup"] = email_signup

        password_signup = request.form["password_signup"]
        session["password_signup"] = password_signup

        password_verification = request.form["password_verification"]
        session["password_verification"] = password_verification

        if not email_signup:
            flash("Enter email!", "info")
            return redirect(url_for("signup"))

        existing_user = UserInformation.query.filter_by(email=email_signup).first()
        if existing_user:
            flash("User already exists, please log in.", "danger")
            return redirect(url_for("login"))

        if not password_signup:
            flash("Enter password!", "info")
            return redirect(url_for("signup"))

        if not password_verification:
            flash("Verify password!", "info")
            return redirect(url_for("signup"))

        if password_signup != password_verification:
            flash("Passwords do not match. Please try again.", "danger")
            return redirect(url_for("signup"))

        new_user = UserInformation(email=email_signup, password=password_signup, content="",workouts="")
        db.session.add(new_user)
        db.session.commit()
        flash("Successfully signed up", "info")
        return redirect(url_for("login"))

    return render_template("signup.html")

# Route for user information page
@app.route("/UserInformation", methods=["POST", "GET"])
def user_info():
    if "email" in session:
        email = session["email"]

        if request.method == "POST":
            content = request.form.get('workouts')
            workouts = request.form.get('content')
            print("Received POST request with content:", content)
            print("Received POST request with workouts:", workouts)
            if content:
                new_user_info = UserInformation(email=email, password="", content=content,workouts=workouts)
                db.sessionadd(new_user_info)
                db.session.commit()
                print("Workout saved to database")

        user_workouts = UserInformation.query.filter_by(email=email).all()
        for workout in user_workouts:
            print("User workout from database:", workout.email, workout.content)
        return render_template("user.html", email=email, workouts=user_workouts)

    return redirect(url_for("login"))

# Route for the homepage
@app.route("/homepage")
def homepage():
    if "email" in session and "password" in session:
        email = session["email"]
        password = session["password"]
        flash("Successfully redirected", "info")
        return render_template("homepage.html", email=email, password=password)
    else:
        return redirect(url_for("login"))

@app.route("/feedback", methods=["POST", "GET"])
def feedback():
    return render_template("feedback.html")

# Route for the pre-generated workout page
@app.route("/generated", methods=["POST", "GET"])
def generated():
    return render_template("pregenerated.html")

# Route for the custom workout page
@app.route("/custom", methods=["POST", "GET"])
def custom():
    return render_template("custom.html")

# Route for logging out
@app.route("/logout")
def logout():
    session.pop("email", None)
    session.pop("password", None)
    session.pop("email_signup", None)
    session.pop("password_signup", None)
    session.pop("password_verification", None)
    flash("Successfully logged out", "info")
    return redirect(url_for("landingpage"))

# Run the webapp
if __name__ == '__main__':
    with app.app_context():
        db.create_all()  # Create database tables
    app.run(debug=True)
