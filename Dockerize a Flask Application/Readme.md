![image](https://user-images.githubusercontent.com/44756128/114846775-9f6e4980-9da2-11eb-8729-c7bc8dd7b4ca.png)

# Introduction
Migrating static content into containers was a great way to learn the basics of Docker, but real-world uses are usually dynamic, including full applications.

This will show the process to dockerize a Flask application. Flask is a lightweight Python WSGI micro web framework.

Log in to the server using SSH:
```sh
ssh username@<PUBLIC_IP_ADDRESS>
```

# Create the Build Files
Change to the notes directory:
```sh
cd notes
```

List the files in the directory:
```sh
ls -la
```

Inspect the config.py file:
```sh
cat config.py
```

Crate the .dockerignore file:
```sh
vim .dockerignore
```

In the file, paste the following:
```sh
.dockerignore
Dockerfile
.gitignore
Pipfile.lock
migrations/
```

Save the file:
```sh
ESC
:wq
```

Create the Dockerfile:
```sh
vim Dockerfile
```

In the file, paste the following:
```py
FROM python:3
ENV PYBASE /pybase
ENV PYTHONUSERBASE $PYBASE
ENV PATH $PYBASE/bin:$PATH
RUN pip install pipenv

WORKDIR /tmp
COPY Pipfile .
RUN pipenv lock
RUN PIP_USER=1 PIP_IGNORE_INSTALLED=1 pipenv install -d --system --ignore-pipfile

COPY . /app/notes
WORKDIR /app/notes
EXPOSE 80
CMD ["flask", "run", "--port=80", "--host=0.0.0.0"]
```

![image](https://user-images.githubusercontent.com/44756128/114847670-8f0a9e80-9da3-11eb-8acc-4b2861c3b5b1.png)

# Build and Setup Environment
Build the notesapp image:
```sh
docker build -t notesapp:0.1 .
```

![image](https://user-images.githubusercontent.com/44756128/114847887-cb3dff00-9da3-11eb-8698-ce7ed978f525.png)

Check the status of the image:
```sh
docker images
```

![image](https://user-images.githubusercontent.com/44756128/114847925-d4c76700-9da3-11eb-8473-b7e3dff5fe1b.png)

List the containers:
```sh
docker ps -a
```

List the docker networks:
```sh
docker network ls
```

Run a container using the notesapp image and mount the migrations directory:
```sh
docker run --rm -it --network notes -v /home/cloud_user/notes/migrations:/app/notes/migrations notesapp:0.1 bash
```

Once connected to the container, enable SQLAlchemy:
```sh
flask db init
```

Check the migrations folder:
```sh
ls -l migrations
```

Create the files needed to configure the database:
```sh
flask db migrate
```

Apply the files:
```sh
flask db upgrade
```

![image](https://user-images.githubusercontent.com/44756128/114848193-10623100-9da4-11eb-8c5e-a8756ef4c626.png)

# Run, Evaluate, and Upgrade
Run a container using the notesapp:0.1 image:
```sh
docker run --rm -it --network notes -p 80:80 notesapp:0.1
```

![image](https://user-images.githubusercontent.com/44756128/114848319-31c31d00-9da4-11eb-8d79-8ddb965ed2ba.png)

Using a web browser, navigate to the public IP address for the server.

![image](https://user-images.githubusercontent.com/44756128/114848387-42739300-9da4-11eb-9ab5-b8c866051b4d.png)

Sign up for a new account using an email address and password.

Once you are signed up, log in to your account.

![image](https://user-images.githubusercontent.com/44756128/114848547-6afb8d00-9da4-11eb-8052-2c86debf29fe.png)

Create your first note.

![image](https://user-images.githubusercontent.com/44756128/114848675-8d8da600-9da4-11eb-8df0-cb5aeb438b91.png)

Verify that you can edit the note.

![image](https://user-images.githubusercontent.com/44756128/114848743-a433fd00-9da4-11eb-8fa1-bfab9df17efa.png)

![image](https://user-images.githubusercontent.com/44756128/114849444-5e2b6900-9da5-11eb-92fb-7fb2a7d58032.png)

Back in the terminal, disable Debug mode by editing the .env file:
```sh
vim .env
```

Remove the export FLASK_ENV='development' line.

Save the file:
```sh
ESC
:wq
```

![image](https://user-images.githubusercontent.com/44756128/114849081-05f46700-9da5-11eb-8429-648d0e0772b3.png)

Build the image again:
```sh
docker build -t notesapp:0.2 .
```

Run a container using the updated image:
```sh
docker run --rm -it --network notes -p 80:80 notesapp:0.2
```
![image](https://user-images.githubusercontent.com/44756128/114849158-1c9abe00-9da5-11eb-8116-27888d01acea.png)

 In a web browser, navigate to the public IP address for the server, and log in to your account.
 
 Verify that you can add a second note.

![image](https://user-images.githubusercontent.com/44756128/114849337-4227c780-9da5-11eb-91aa-977371859f03.png)

![image](https://user-images.githubusercontent.com/44756128/114849586-85823600-9da5-11eb-90e9-23aefc721e69.png)

In the terminal, stop the container:
```sh
CTRL+C
```

# Upgrade to Gunicorn
Check the Pipfile:
```sh
cat Pipfile
```

Run a container and modify the pip file:
```sh
docker run --rm -it -v /home/cloud_user/notes/Pipfile:/tmp/Pipfile notesapp:0.2 bash
```

Once connected, change to the /tmp directory:
```sh
cd /tmp
```

Add gunicorn to the list of dependencies:
```sh
pipenv install gunicorn
```

Exit the container:
```sh
exit
```

Verify that gunicorn was added under [packages]:
```sh
cat Pipfile
```

Modify the init.py script:
```sh
vim __init__.py
```

![image](https://user-images.githubusercontent.com/44756128/114850339-46081980-9da6-11eb-97ff-93fad25effce.png)

![image](https://user-images.githubusercontent.com/44756128/114850388-54563580-9da6-11eb-91d7-768a3a209e2f.png)

Beneath the import section, add the following:
```py
from dotenv import load_dotenv, find_dotenv
load_dotenv(find_dotenv())
```

```py
import os
import functools
from dotenv import load_dotenv, find_dotenv
load_dotenv(find_dotenv())

from flask import Flask, render_template, redirect, url_for, request, session, flash, g
from flask_migrate import Migrate
from werkzeug.security import generate_password_hash, check_password_hash

def create_app(test_config=None):
    app = Flask(__name__)
    app.config.from_mapping(
        SECRET_KEY=os.environ.get('SECRET_KEY', default='dev'),
    )

    if test_config is None:
        app.config.from_pyfile('config.py', silent=True)
    else:
        app.config.from_mapping(test_config)

    from .models import db, User, Note

    db.init_app(app)
    migrate = Migrate(app, db)

    def require_login(view):
        @functools.wraps(view)
        def wrapped_view(**kwargs):
            if not g.user:
                return redirect(url_for('log_in'))
            return view(**kwargs)
        return wrapped_view

    @app.errorhandler(404)
    def page_not_found(e):
        return render_template('404.html'), 404

    @app.before_request
    def load_user():
        user_id = session.get('user_id')
        if user_id:
            g.user = User.query.get(user_id)
        else:
            g.user = None

    @app.route('/sign_up', methods=('GET', 'POST'))
    def sign_up():
        if request.method == 'POST':
            username = request.form['username']
            password = request.form['password']
            error = None

            if not username:
                error = 'Username is required.'
            elif not password:
                error = 'Password is required.'
            elif User.query.filter_by(username=username).first():
                error = 'Username is already taken.'

            if error is None:
                user = User(username=username, password=generate_password_hash(password))
                db.session.add(user)
                db.session.commit()
                flash("Successfully signed up! Please log in.", 'success')
                return redirect(url_for('log_in'))

            flash(error, category='error')

        return render_template('sign_up.html')

    @app.route('/log_in', methods=('GET', 'POST'))
    def log_in():
        if request.method == 'POST':
            username = request.form['username']
            password = request.form['password']
            error = None

            user = User.query.filter_by(username=username).first()

            if not user or not check_password_hash(user.password, password):
                error = 'Username or password are incorrect'

            if error is None:
                session.clear()
                session['user_id'] = user.id
                return redirect(url_for('index'))

            flash(error, category='error')
        return render_template('log_in.html')

    @app.route('/')
    def index():
        return redirect(url_for('note_index'))

    @app.route('/log_out', methods=('GET', 'DELETE'))
    def log_out():
        session.clear()
        flash('Successfully logged out.', 'success')
        return redirect(url_for('log_in'))

    @app.route('/notes')
    @require_login
    def note_index():
        return render_template('note_index.html', notes=g.user.notes)

    @app.route('/notes/new', methods=('GET', 'POST'))
    @require_login
    def note_create():
        if request.method == 'POST':
            title = request.form['title']
            body = request.form['body']
            error = None

            if not title:
                error = 'Title is required.'

            if not error:
                note = Note(author=g.user, title=title, body=body)
                db.session.add(note)
                db.session.commit()
                flash(f"Successfully created note: '{title}'", 'success')
                return redirect(url_for('note_index'))

            flash(error, 'error')

        return render_template('note_create.html')

    @app.route('/notes/<note_id>/edit', methods=('GET', 'POST', 'PATCH'))
    @require_login
    def note_update(note_id):
        note = Note.query.filter_by(user_id=g.user.id, id=note_id).first_or_404()
        if request.method in ['POST', 'PATCH']:
            title = request.form['title']
            body = request.form['body']
            error = None

            if not title:
                error = 'Title is required.'

            if not error:
                note.title = title
                note.body = body
                db.session.add(note)
                db.session.commit()
                flash(f"Successfully updated note: '{title}'", 'success')
                return redirect(url_for('note_index'))

            flash(error, 'error')

        return render_template('note_update.html', note=note)

    @app.route('/notes/<note_id>/delete', methods=('GET', 'DELETE'))
    @require_login
    def note_delete(note_id):
        note = Note.query.filter_by(user_id=g.user.id, id=note_id).first_or_404()
        db.session.delete(note)
        db.session.commit()
        flash(f"Successfully deleted note: '{note.title}'", 'success')
        return redirect(url_for('note_index'))

    return app
```

Save the file:
```sh
ESC
:wq
```

Open the Dockerfile:
```sh
vim Dockerfile
```

To the bottom of the file, make the following changes:
```sh
COPY . /app/notes
WORKDIR /app
EXPOSE 80
CMD ["gunicorn", "-b 0.0.0.0:80", "notes:create_app()"]
```

Save the file:
```sh
ESC
:wq
```

![image](https://user-images.githubusercontent.com/44756128/114850711-adbe6480-9da6-11eb-9abe-1a1b0e94edec.png)

# Build a Production Image
Build the updated notesapp image:
```sh
docker build -t notesapp:0.3 .
```

Run a detached container using the updated image:
```sh
docker run -d --name notesapp --network notes -p 80:80 notesapp:0.3
```

![image](https://user-images.githubusercontent.com/44756128/114850975-f37b2d00-9da6-11eb-9734-93693748c126.png)

Check the status of the container:
```sh
docker ps -a
```

![image](https://user-images.githubusercontent.com/44756128/114851008-fbd36800-9da6-11eb-85e6-af26c45c2242.png)

In a web browser, navigate to the public IP address for the server, and log in to your account.

Verify that you can create a new note.

![image](https://user-images.githubusercontent.com/44756128/114851131-1c032700-9da7-11eb-874e-d7cc3740c34d.png)
