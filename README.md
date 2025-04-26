# Technical Lesson: Querying a Database in a Flask Application

You’ve built the structure of your Flask app, and your database is seeded 
with sample data — but how do you actually make that data accessible to users?

## Scenario

At your first junior developer job at Pawfect Adoption Center, the team has 
a growing database of adoptable pets. They need a way to query that database 
and display information dynamically based on user interactions - like searching 
for pets by ID, by species, or eventually by name or availability.

In this lesson, you'll learn how to:
* Query the database directly through Flask-SQLAlchemy.
* Serve that data through Flask views as HTML responses.
* Handle situations where a user searches for a record that doesn’t exist.

This is a major milestone - connecting your backend database logic to your 
frontend user interface!

## Tools & Resources

- [GitHub Repo](http://github.com/learn-co-curriculum/flask-sqlalchemy-querying-technical-lesson)
- [Quickstart - Flask-SQLAlchemy](https://flask-sqlalchemy.palletsprojects.com/en/2.x/quickstart/)
- [Flask-Migrate](https://flask-migrate.readthedocs.io/en/latest/)
- [Flask Extensions, Plug-ins, and Related Libraries - Full Stack Python](https://www.fullstackpython.com/flask-extensions-plug-ins-related-libraries.html)

## Setup

This lesson is a code-along, so fork and clone the repo.

Run `pipenv install` to install the dependencies and `pipenv shell` to enter
your virtual environment before running your code.

```console
$ pipenv install
$ pipenv shell
```

Change into the `server` directory and configure the `FLASK_APP` and
`FLASK_RUN_PORT` environment variables:

```console
$ cd server
$ export FLASK_APP=app.py
$ export FLASK_RUN_PORT=5555
```

Execute the command `tree` within the `server` directory.

```command
$ tree
```

```text
.
├── app.py
├── migrations
│   ├── README
│   ├── alembic.ini
│   ├── env.py
│   ├── script.py.mako
│   └── versions
│       └── 51b06098cc9e_initial_migration.py
├── models.py
└── seed.py
```

The commands `flask db init` and `flask db migrate` have already been run, so
the `server` directory contains the `migrations` directory, and the directory
`server/migrations/versions` contains an initial migration script.

## Instructions

### Task 1: Define the Problem

Having a populated database is not enough — web applications must retrieve 
and present specific pieces of information based on user requests. Whether 
a user wants to see a particular pet’s profile, all dogs available for adoption, 
or the total number of turtles, your application needs to query the database 
efficiently and serve the results back over the web.

Manually connecting the database to a web route or hardcoding information is 
not scalable. You need dynamic querying — flexible, efficient, and safe.

Your Challenge is to use Flask-SQLAlchemy’s query capabilities to:
* Retrieve individual records or lists of records based on URL parameters.
* Handle cases where no matching data is found.
* Dynamically construct responses using the results of your queries.

This pattern forms the foundation of how real-world web applications handle 
user-generated requests every day.

### Task 2: Determine the Design

The technical design for querying and displaying database information will include:

* Using SQLAlchemy ORM Queries:
    * Query your models using .query, .filter(), .filter_by(), .all(), and .first() methods to retrieve records based on user-supplied parameters.
* Building Views Dynamically:
    * In Flask, views (functions decorated with @app.route) will accept dynamic parameters like id or species, query the database based on those parameters, and return an HTML response containing the relevant data.
* Handling Missing Data:
    * When a query returns no results, serve a customized 404 error page or message rather than letting the app crash or show a server error.
* Using Flask's make_response():
    * Construct customized HTTP responses, setting appropriate status codes (e.g., 200 for success, 404 for not found).
* Scaling with Query Results:
    * Dynamically loop over query results (like a list of pets) to generate multiple parts of a response when needed (such as listing all cats).

This ensures the application can serve real-time data, handle edge cases safely, and scale gracefully as more user requests and different types of queries are introduced.

### Task 3: Develop, Test, and Refine the Code

#### Step 1: Initialize Database

Run the following command to create the `instance` directory with the database
and initialize the database from the existing migration script:

```console
$ flask db upgrade head
```

The `instance` folder should now appear along with the database file `app.db`
inside it:

```text
.
├── app.py
├── instance
│   └── app.db
├── migrations
│   ├── README
│   ├── alembic.ini
│   ├── env.py
│   ├── script.py.mako
│   └── versions
│       └── 51b06098cc9e_initial_migration.py
├── models.py
└── seed.py
```

Confirm the database contains an empty `pets` table, either by using the Flask
shell or by using a VS Code extension to view the table contents.

```console
$ flask shell
>>> Pet.query.all()
[]
>>>
```

![new pet table](/assets/pet_table.png)

#### Step 2: Seed the Database

Let's seed the table with sample data. Type the following within the `server`
directory:

```command
$ python seed.py
```

Use the Flask shell to confirm 10 random pets have been added to the database:

```command
$ flask shell
>>> Pet.query.all()
[<Pet 1, Robin, Hamster>, <Pet 2, Gwendolyn, Dog>, <Pet 3, Michael, Turtle>, <Pet 4, Austin, Cat>, <Pet 5, Jennifer, Dog>, <Pet 6, Jenna, Dog>, <Pet 7, Crystal, Chicken>, <Pet 8, Jacob, Cat>, <Pet 9, Nicole, Chicken>, <Pet 10, Trevor, Turtle>]
>>>
```

#### Step 3: Serve Database Records in Endpoints

Open `server/app.py` and modify it to add the `index()` view as shown below:

```py
# server/app.py
#!/usr/bin/env python3

from flask import Flask, make_response
from flask_migrate import Migrate

from models import db, Pet

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///app.db'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False

migrate = Migrate(app, db)

db.init_app(app)


@app.route('/')
def index():
    response = make_response(
        '<h1>Welcome to the pet directory!</h1>',
        200
    )
    return response


if __name__ == '__main__':
    app.run(port=5555, debug=True)

```

We've seen all of this code before in one way or another, but let's take a
moment to review:

- Our `app.config` is set up to point to our existing database, and
  `'SQLALCHEMY_TRACK_MODIFICATIONS'` is set to `False` to avoid building up too
  much unhelpful data in memory when our application is running.
- Our `migrate` instance configures the application and models for
  Flask-Migrate.
- `db.init_app` connects our database to our application before it runs.
- `@app.route` determines which resources are available at which URLs and saves
  them to the application's URL map.
- Responses are what we return to the client after a request and `make_response`
  helps us with that. It is a function that allows you to create an HTTP
  response object that you can customize before returning it to the client. It's
  a useful tool for building more complex responses, especially when you need to
  set custom headers, cookies, or other response attributes.The included
  response has a status code of 200, which means that the resource exists and is
  accessible at the provided URL.

Enter the `server/` directory if you're not there already and type:

```console
python app.py
```

In a browser, navigate to 127.0.0.1:5555. You should see this message in your
browser:

![h1 text "Welcome to the pet directory!" in Google Chrome](/assets/index_welcome_html.png)

#### Step 4: Create Views

Let's start working on displaying data in our Flask application.

We'll start by adding another view for pets, searched by ID. Add the new view
after the `index()` view:

```py
# server/app.py

from flask import Flask, make_response
from flask_migrate import Migrate

from models import db, Pet

# add this view after index()

@app.route('/pets/<int:id>')
def pet_by_id(id):
    pet = Pet.query.filter(Pet.id == id).first()
    response_body = f'<p>{pet.name} {pet.species}</p>'

    response = make_response(response_body, 200)

    return response

if __name__ == '__main__':
    app.run(port=5555, debug=True)

```

Run the application again and navigate to
[127.0.0.1:5555/pets/1](127.0.0.1:5555/pets/1). Because we generated data with
Faker, you will probably not see the same name and species, but your format
should match below:

![p text Information for pet with id 1](/assets/pet1.png)

Navigate now to 127.0.0.1:5555/pets/1000. You will see an error message that
suggests something went wrong on your server. We're not tracking 1000 pets right
now, but we still don't want our users to see error pages like this. Let's make
another small change to the `pet_by_id()` view to fix this:

```py
@app.route('/pets/<int:id>')
def pet_by_id(id):
    pet = Pet.query.filter(Pet.id == id).first()

    if pet:
        response_body = f'<p>{pet.name} {pet.species}</p>'
        response_status = 200
    else:
        response_body = f'<p>Pet {id} not found</p>'
        response_status = 404

    response = make_response(response_body, response_status)
    return response
```

404 is the status code for "Not Found". It is generally used for the case we see
here: an ID or username that went into the URL but does not exist. Our 404 page
is quite minimal, but applications like Twitter and Facebook format them with
descriptive messages and the same style and formatting of their website as a
whole.

![error message for pet with id 1000](/assets/pet1000.png)

Let's add another view to get all pets for a given species. We'll filter to get
all rows that match the `species` route parameter, then loop through the query
result to generate response information for each pet.

```py
@app.route('/species/<string:species>')
def pet_by_species(species):
    pets = Pet.query.filter_by(species=species).all()

    size = len(pets)  # all() returns a list so we can get length
    response_body = f'<h2>There are {size} {species}s</h2>'
    for pet in pets:
        response_body += f'<p>{pet.name}</p>'
    response = make_response(response_body, 200)
    return response
```

The expression `species=species` passed into the `filter_by` function may be a
bit confusing. The `species` before the equal sign refers to the table column,
while `species` after the equal sign refers to the route parameter.

Let's test this new route (your result will differ). Navigate to
127.0.0.1:5555/species/Dog.

![query by species](/assets/species.png)

#### Step 5: Verify your Code

Solution Code:

```py
# server/app.py
#!/usr/bin/env python3

from flask import Flask, make_response
from flask_migrate import Migrate

from models import db, Pet

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///app.db'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False

migrate = Migrate(app, db)

db.init_app(app)


@app.route('/')
def index():
    response = make_response(
        '<h1>Welcome to the pet directory!</h1>',
        200
    )
    return response


@app.route('/pets/<int:id>')
def pet_by_id(id):
    pet = Pet.query.filter(Pet.id == id).first()

    if pet:
        response_body = f'<p>{pet.name} {pet.species}</p>'
        response_status = 200
    else:
        response_body = f'<p>Pet {id} not found</p>'
        response_status = 404

    response = make_response(response_body, response_status)
    return response


@app.route('/species/<string:species>')
def pet_by_species(species):
    pets = Pet.query.filter_by(species=species).all()

    size = len(pets)  # all() returns a list so we can get length
    response_body = f'<h2>There are {size} {species}s</h2>'
    for pet in pets:
        response_body += f'<p>{pet.name}</p>'
    response = make_response(response_body, 200)
    return response


if __name__ == '__main__':
    app.run(port=5555, debug=True)
```

#### Step 6: Commit, Push, and Merge to GitHub

* Commit and push your code:

```bash
git add .
git commit -m "create pets table with some started data"
git push
```

* If you created a separate feature branch, remember to open a PR on main and merge.

### Task 4: Document and Maintain

Best Practice documentation steps:
* Add comments to the code to explain purpose and logic, clarifying intent and functionality of your code to other developers.
* Update README text to reflect the functionality of the application following https://makeareadme.com. 
  * Add screenshot of completed work included in Markdown in README.
* Delete any stale branches on GitHub
* Remove unnecessary/commented out code
* If needed, update git ignore to remove sensitive data

## Considerations

When designing queries in a Flask-SQLAlchemy application, consider the following:

* Efficiency Matters:
    * Avoid retrieving more data than necessary — for example, use `.first()` when you only need one record instead of .all().
* Security and Validation:
    * Be cautious when accepting parameters from URLs — always validate types (e.g., `<int:id>`) and handle missing data gracefully with 404 responses.
* Performance Scaling:
    * As your app grows, complex queries (joins, filters, ordering) will need to be optimized. Start building good habits now by being selective and thoughtful in your queries.
* User Experience:
    * Clear, friendly messages when a resource isn’t found (404s) help users trust your app more than generic or broken error pages.
* Data Consistency:
    * When querying based on strings like species names, remember that slight differences (capitalization, pluralization) could impact query results. You may eventually need case-insensitive searches or validations.
* Future API Design:
    * While you're returning simple HTML strings now, the same querying patterns form the basis for building REST APIs later — responses will be JSON instead of HTML.

Mastering database queries within Flask views is the critical bridge between having raw data and building dynamic, interactive, full-stack applications.

