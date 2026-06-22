# Student Grade Management System for Teachers

A web-based grade management tool built with Python/Flask and MySQL. Teachers can record grades, calculate weighted averages, generate reports, and export data. Students can view their performance with anonymized class comparisons.

## Features

- **Three roles**: Admin, Teacher, Student with role-based access
- **Course management**: Admin creates courses with academic terms
- **Weighted grading**: Custom categories with percentage weights
- **Grade recording**: Enter/edit scores per student per grade item
- **Student view**: Personal grades, overall percentage, letter grade, class averages
- **Reporting**: Grade distribution histogram (Chart.js), failing students list, CSV export
- **Audit log**: Every grade change is recorded with who and when
- **Security**: bcrypt passwords, prepared statements, role middleware

## Tech Stack

- **Backend**: Python 3, Flask, SQLAlchemy
- **Database**: MySQL
- **Frontend**: Bootstrap 5, Chart.js, Font Awesome
- **Testing**: PyTest

## Setup

1. **Clone and create virtual environment:**
   ```bash
   git clone <repo-url>
   cd student-grade-management
   python -m venv venv
   .\venv\Scripts\activate    # Windows
   pip install -r requirements.txt
   ```

2. **Configure database:**
   - Create a MySQL database named `grade_management`
   - Edit `.env` with your MySQL credentials

3. **Initialize database:**
   ```bash
   python database/init_db.py
   ```
   Creates tables and an admin account: `admin@school.edu` / `admin123`

4. **Run the application:**
   ```bash
   python app.py
   ```

5. **Run tests:**
   ```bash
   pytest tests/ -v
   ```

## Project Structure

```
├── app.py                  # Application entry point
├── config.py               # Configuration
├── requirements.txt        # Dependencies
├── database/
│   ├── schema.sql          # MySQL schema
│   └── init_db.py          # Database initialization
├── src/
│   ├── models/             # SQLAlchemy models
│   ├── routes/             # Flask blueprints
│   ├── services/           # Business logic
│   ├── templates/          # Jinja2 templates
│   └── static/             # CSS, JS
└── tests/                  # PyTest test suite
```
