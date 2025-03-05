```markdown
# Movie App

Code repository for NIIT Capstone project - Movie App.

## Table of Contents  
- [About](#about)  
- [Features](#features)  
- [Technologies Used](#technologies-used)  
- [Architecture](#architecture)  
- [Installation](#installation)  
- [Usage](#usage)  
- [Contributing](#contributing)  
- [License](#license)  

---

## About  
The Movie App is designed to provide users with information about various movies, including details such as title, genre, release date, and ratings. This project serves as a capstone project for NIIT, demonstrating the application of various technologies and best practices in software development.

---

## Features  
- Browse a list of movies with detailed information.  
- Search for movies by title or genre.  
- View detailed information about each movie, including synopsis, cast, and ratings.  
- User authentication to manage personal watchlists or favorites.  

---

## Technologies Used  
### Frontend:  
- Angular  
- TypeScript  
- HTML5  
- CSS3  

### Backend:  
- Java  
- Spring Boot  
- RESTful APIs  

### Database:  
- MySQL  

### Version Control:  
- Git  

---

## Architecture  
The application follows a client-server architecture:  
- **Frontend**: Developed using Angular and TypeScript, providing a dynamic and responsive user interface.  
- **Backend**: Built with Java and Spring Boot, handling business logic and API endpoints.  
- **Database**: MySQL is used for data storage, managing movie details and user information.  

---

## Installation  
Follow these steps to set up the project locally:  

1. **Clone the repository**:  
   ```bash
   git clone https://github.com/jamesndr/movie-app.git
   ```

2. **Navigate to the project directory**:  
   ```bash
   cd movie-app
   ```

3. **Set up the backend**:  
   ```bash
   # Navigate to the backend directory
   cd backend

   # Install dependencies and build the project (Maven)
   mvn install

   # Start the Spring Boot application
   mvn spring-boot:run
   ```

4. **Set up the frontend**:  
   ```bash
   # Navigate to the frontend directory
   cd ../frontend

   # Install dependencies
   npm install

   # Start the Angular development server
   ng serve
   ```

5. **Database Setup**:  
   - Ensure MySQL is installed and running.  
   - Create a database named `movie_app`.  
   - Update the database configurations in `backend/src/main/resources/application.properties`.  
   - Run the Spring Boot application to auto-generate tables via Hibernate.  

---

## Usage  
1. Access the frontend at `http://localhost:4200` (Angular default port).  
2. Browse movies or use the search bar to find specific titles/genres.  
3. Click on a movie to view details like synopsis, cast, and ratings.  
4. Register or log in to save movies to your watchlist/favorites.  

---

## Contributing  
Contributions are welcome! Follow these steps:  
1. Fork the repository.  
2. Create a feature branch:  
   ```bash
   git checkout -b feature/your-feature-name
   ```  
3. Commit your changes:  
   ```bash
   git commit -m "Add your commit message"
   ```  
4. Push to the branch:  
   ```bash
   git push origin feature/your-feature-name
   ```  
5. Open a pull request on GitHub.  

---

