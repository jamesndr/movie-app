Movie App

Code repository for NIIT Capstone project - Movie App.

Table of Contents

About

Features

Technologies Used

Architecture

Installation

Usage

Contributing

License

About

The Movie App is designed to provide users with information about various movies, including details such as title, genre, release date, and ratings. This project serves as a capstone project for NIIT, demonstrating the application of various technologies and best practices in software development.

Features

Browse a list of movies with detailed information.

Search for movies by title or genre.

View detailed information about each movie, including synopsis, cast, and ratings.

User authentication to manage personal watchlists or favorites.

Technologies Used

Frontend:

Angular

TypeScript

HTML5

CSS3

Backend:

Java

Spring Boot

RESTful APIs

Database:

MySQL

Version Control:

Git

Architecture

The application follows a client-server architecture:

Frontend: Developed using React.js and TypeScript, providing a dynamic and responsive user interface.

Backend: Built with Java and Spring Boot, handling business logic and API endpoints.

Database: MySQL is used for data storage, managing movie details and user information.

Installation

To set up the project locally, follow these steps:

Clone the repository:

git clone https://github.com/jamesndr/movie-app.git

Navigate to the project directory:

cd movie-app

Set up the backend:

Navigate to the backend directory:

cd backend

Install necessary dependencies and build the project.

Start the Spring Boot application.

Set up the frontend:

Navigate to the frontend directory:

cd ../frontend

Install dependencies:

npm install

Start the development server:

npm start

Database Setup:

Ensure MySQL is installed and running.

Create a database named movie_app.

Update the database configurations in the backend application properties file.

Run the necessary migrations or scripts to set up the database schema.

Usage

Once the application is set up and running:

Access the frontend at http://localhost:3000.

Browse the list of movies or use the search functionality to find specific titles.

Click on a movie to view detailed information.

Register or log in to manage your personal watchlist or favorite movies.

Contributing

Contributions are welcome! To contribute:

Fork the repository.

Create a new branch:

git checkout -b feature-name

Make your changes and commit them:

git commit -m 'Add new feature'

Push to the branch:

git push origin feature-name

Submit a pull request detailing your changes.
