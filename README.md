# DB_web_app
Lab 4: Flask CRUD Web Application

Setup instructions
1.	Install Python 3, unzip the project, and open a terminal in the folder that contains app.py.
2.	Create a virtual environment: python -m venv venv
3.	Activate it: venv\Scripts\activate. If PowerShell blocks the script, run Set-ExecutionPolicy -Scope CurrentUser RemoteSigned once and try again.
4.	Install the dependencies: pip install -r requirements.txt
5.	Start the app: python app.py
6.	Open http://127.0.0.1:5000 in a browser. Stop the server with Ctrl+C.

The database file movies_lab2.db must be in the same folder as app.py. The file database.sql creates the tables and inserts five sample movies if the database needs to be rebuilt.
Application structure
  •	app.py: the Flask application, the database connection and the four routes
  •	templates/index.html: the page that lists the movies, with Edit and Delete controls
  •	templates/form.html: the form shared by the Add and Edit pages
  •	movies_lab2.db: the SQLite database from Lab 2
  •	database.sql: the table creation script and sample data
  •	requirements.txt: the packages the app needs
  •	screenshots/: the demonstration of the CRUD operations
  
Database connection
  The app uses Python’s built-in sqlite3 module. DB_PATH is built from the location of app.py, so the app works no matter where it is started from. The function get_db() first checks that the database file exists. If it is missing, it shows an error instead of letting SQLite silently create an empty database, which would cause problems later. So it warns us and checks that everything is in order before proceeding. It then opens the connection and sets row_factory so that templates can read columns by name. Each route opens the connection, runs its own SQL, commits if it changed data, and then closes the connection. The commit saves the change permanently to the data. The queries use a question mark instead of putting the user’s text into the SQL because of SQL injection. The question mark makes the database treat the user’s input as a plain value and never as part of the SQL command.
  
Routes and functionality
  •	/ shows the 50 newest movies. It joins movie to country and rating with a LEFT JOIN, so a movie with a missing country or rating still appears.
  •	/add shows an empty form with dropdowns built from the country and rating tables (GET). On submit (POST) it checks that a name was entered, inserts the movie with a parameterized INSERT, and returns to the list.
  •	/edit/<id> loads the movie and shows the form pre-filled (GET). On submit (POST) it runs an UPDATE with WHERE movie_id = ?. An ID that does not exist gives a 404 page.
  •	/delete/<id> accepts POST only, so a link alone can never delete data. It removes the movie’s rows from movie_person first and then the movie. The Delete button asks for confirmation.
  
Reflection
  The Add and Edit forms needed dropdowns for country and rating, because the movie table stores them as numbers and the names live in other tables. What I get in return for the extra work is fewer errors moving forward and more accurate data locating and navigating, and it also minimizes the risk of duplication.
The delete route runs two statements to keep the data clean when a specific movie is deleted. The first one deletes that movie’s rows from movie_person, and the second deletes the movie itself. If I deleted only the movie, its rows in movie_person would stay behind and point at a movie that no longer exists, so they would point at nothing. The WHERE movie_id = ? keeps the data safe because it makes sure the director’s rows for other movies are untouched. And since we are using sqlite3 in Python, foreign keys are off by default, so the database will not do this cleanup for us.
What went wrong: I learned that even if the names are similar, we should actually check whether there is any data in the file at all. Otherwise it is pointless to try over and over again with the same file, like I did, until I realized it was not the right file.
If I could change something, I would add ON DELETE CASCADE to the movie_person foreign key, so the database deletes the junction rows itself whenever a movie is deleted and the app’s first delete is no longer needed. I would also add a way to look up movies by the director’s name. My tables already make this possible, because movie_person links movies to people, and a join can find them.
The movie table stores only a number, the country_id, and the country’s name is in a different table. The query joins the movie’s country_id with the country table’s country_id, matches them and links them together. A LEFT JOIN is used so that a movie with no country still shows up, with a blank cell instead of disappearing.
 
