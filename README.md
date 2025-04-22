# Byte-compiled / optimized / DLL files
__pycache__/
*.py[cod]
*$py.class

# C extensions
*.so

# Distribution / packaging
.Python
build/
develop-eggs/
dist/
downloads/
eggs/
.eggs/
lib/
lib64/
parts/
sdist/
var/
wheels/
share/python-wheels/
*.egg-info/
.installed.cfg
*.egg
MANIFEST

# PyInstaller 
# Usually these files are written by a python script from a template
# before PyInstaller builds the exe, so as to inject date/other infos into it.
*.manifest
*.spec

# Installer logs
pip-log.txt
pip-delete-this-directory.txt

# Unit test / coverage reports
htmlcov/
.tox/
.nox/
.coverage
.coverage.*
.cache
nosetests.xml
coverage.xml
*.cover
*.py,cover
.hypothesis/
.pytest_cache/
cover/

# Translations
*.mo
*.pot

# Django stuff:
*.log
local_settings.py
db.sqlite3
db.sqlite3-journal

# Flask stuff:
instance/
.webassets-cache

# Scrapy stuff:
.scrapy

# Sphinx documentation
docs/_build/

# PyBuilder
.pybuilder/
target/

# Jupyter Notebook
.ipynb_checkpoints

# IPython
profile_default/
ipython_config.py

# pyenv
#   For a library or package, you might want to ignore these files since the code is
#   intended to run in multiple environments; otherwise, check them in:
# .python-version

# pipenv
#   According to pypa/pipenv#598, it is recommended to include Pipfile.lock in version control.
#   However, in case of collaboration, if having platform-specific dependencies or dependencies
#   having no cross-platform support, pipenv may install dependencies that don't work, or not
#   install all needed dependencies.
#Pipfile.lock

# UV
#   Similar to Pipfile.lock, it is generally recommended to include uv.lock in version control.
#   This is especially recommended for binary packages to ensure reproducibility, and is more
#   commonly ignored for libraries.
#uv.lock

# poetry
#   Similar to Pipfile.lock, it is generally recommended to include poetry.lock in version control.
#   This is especially recommended for binary packages to ensure reproducibility, and is more
#   commonly ignored for libraries.
#   https://python-poetry.org/docs/basic-usage/#commit-your-poetrylock-file-to-version-control
#poetry.lock

# pdm
#   Similar to Pipfile.lock, it is generally recommended to include pdm.lock in version control.
#pdm.lock
#   pdm stores project-wide configurations in .pdm.toml, but it is recommended to not include it
#   in version control.
#   https://pdm.fming.dev/latest/usage/project/#working-with-version-control
.pdm.toml
.pdm-python
.pdm-build/

# PEP 582; used by e.g. github.com/David-OConnor/pyflow and github.com/pdm-project/pdm
__pypackages__/

# Celery stuff
celerybeat-schedule
celerybeat.pid

# SageMath parsed files
*.sage.py

# Environments
.env
.venv
env/
venv/
ENV/
env.bak/
venv.bak/

# Spyder project settings
.spyderproject
.spyproject

# Rope project settings
.ropeproject

# mkdocs documentation
/site

# mypy
.mypy_cache/
.dmypy.json
dmypy.json

# Pyre type checker
.pyre/

# pytype static type analyzer
.pytype/

# Cython debug symbols
cython_debug/

# PyCharm
#  JetBrains specific template is maintained in a separate JetBrains.gitignore that can
#  be found at https://github.com/github/gitignore/blob/main/Global/JetBrains.gitignore
#  and can be added to the global gitignore or merged into this file.  For a more nuclear
#  option (not recommended) you can uncomment the following to ignore the entire idea folder.
#.idea/

# Ruff stuff:
.ruff_cache/

# PyPI configuration file
.pypirc
from flask import Flask, render_template, request, redirect, url_for, session, flash
from app.auth import check_password
from cryptography.fernet import Fernet
import os

app = Flask(__name__)
app.secret_key = os.environ.get("FLASK_SECRET_KEY", "this_is_secret")

UPLOAD_FOLDER = os.path.join(os.getcwd(), "secure_uploads")
KEY_PATH = os.path.join(os.getcwd(), "file_encryption.key")
with open(KEY_PATH, "rb") as key_file:
    fernet = Fernet(key_file.read())

os.makedirs(UPLOAD_FOLDER, exist_ok=True)

@app.route('/login', methods=['GET', 'POST'])
def login():
    error = None
    if request.method == 'POST':
        password = request.form['password']
        if check_password(password):
            session['authenticated'] = True
            return redirect(url_for('upload'))
        else:
            error = 'Access denied.'
    return render_template('login.html', error=error)

@app.route('/upload', methods=['GET', 'POST'])
def upload():
    if not session.get('authenticated'):
        return redirect(url_for('login'))

    if request.method == 'POST':
        uploaded_file = request.files['file']
        if uploaded_file:
            original_data = uploaded_file.read()
            encrypted_data = fernet.encrypt(original_data)
            encrypted_filename = f"{uploaded_file.filename}.enc"
            save_path = os.path.join(UPLOAD_FOLDER, encrypted_filename)
            with open(save_path, "wb") as f:
                f.write(encrypted_data)
            flash("File uploaded and encrypted successfully.")
            return redirect(url_for('upload'))

    return render_template('upload.html')

@app.route('/')
def index():
    return redirect(url_for('login'))


    if request.method == 'POST':
        uploaded_file = request.files['file']
        if uploaded_file:
            original_data = uploaded_file.read()
            encrypted_data = fernet.encrypt(original_data)
            encrypted_filename = f"{uploaded_file.filename}.enc"
            save_path = os.path.join(UPLOAD_FOLDER, encrypted_filename)
            with open(save_path, "wb") as f:
                f.write(encrypted_data)
            flash("File uploaded and encrypted successfully.")
            return redirect(url_for('upload'))

    return render_template('upload.html')

@app.route('/')
def index():
    return redirect(url_for('login'))
import bcrypt

# In a real app, store this securely
STORED_USER_HASH = bcrypt.hashpw(b"spiritual_pass", bcrypt.gensalt())

def check_password(password: str) -> bool:
    return bcrypt.checkpw(password.encode(), STORED_USER_HASH)
<!DOCTYPE html>
<html lang = "en">
<head>
    <meta charset = "UTF-8">
    <title>{% block title %}AetherionAI{% endblock %}</title>
    <style>
        body { font-family: 'Georgia', serif; background: #fdf6f0; color: #333; padding: 2rem; }
        h2 { color: #5e4b8b; }
        form { margin-top: 1rem; }
        input, textarea, button { display: block; margin: 0.5rem 0; padding: 0.5rem; width: 100%; }
        button { background: #5e4b8b; color: white; border: none; cursor: pointer; }
    </style>
</head>
<body>
    {% block content %}{% endblock %}
</body>
</html>
{% extends 'base.html' %}
{% block title %}Login{% endblock %}
{% block content %}
<h2>Enter the Sanctuary</h2>
<form method = "post">
    <label>Password:</label>
    <input type = "password" name = "password" required>
    <button type = "submit"> Enter</button>
</form>
{% if error %}
<p style = "color: red;">{{ error }}</p>
{% endif %}
{% endblock %}
{% extends 'base.html' %}
{% block title %}Secure Upload{% endblock %}
{% block content %}
<h2> Encrypted File Upload </h2>
<form method = "post" enctype = "multipart/form-data">
    <label> Select File:</label>
    <input type= "file" name = "file" required>
    < button type="submit">Upload Securely</button>
</form>
{ % endblock % }
from cryptography.fernet import Fernet
with open("file_encryption.key", "wb") as f:
    f.write(Fernet.generate_key())
pip install flask cryptography bcrypt
from app import create_app

app = create_app()

if __name__ == '__main__':
    app.run(debug=True)
flask
flask_sqlalchemy
openai
python-dotenv
from flask import Blueprint, request, jsonify
from .aetherion_ai import AetherionAI
from .models import Memory
from . import db
import os

main = Blueprint('main', __name__)

@main.route('/analyze', methods=['POST'])
def analyze():
    data = request.get_json()
    content = data.get('content')
    context = data.get('context', '')
    summary = data.get('summary', '')

    if not content:
        return jsonify({'error': 'Memory content is required'}), 400

    api_key = os.getenv("OPENAI_API_KEY")
    ai = AetherionAI(api_key)
    analysis = ai.analyze_memory_content({
        'content': content,
        'context': context,
        'summary': summary
    })

    memory = Memory(content=content, context=context, summary=summary, analysis=analysis)
    db.session.add(memory)
    db.session.commit()

    return jsonify({'id': memory.id, 'analysis': analysis})

@main.route('/memories', methods=['GET'])
def get_memories():
    memories = Memory.query.order_by(Memory.created_at.desc()).all()
    result = [
        {
            'id': m.id,
            'content': m.content,
            'context': m.context,
            'summary': m.summary,
            'analysis': m.analysis,
            'created_at': m.created_at.isoformat()
        } for m in memories
    ]
    return jsonify(result)
from . import db
from datetime import datetime

class Memory(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    content = db.Column(db.Text, nullable=False)
    context = db.Column(db.Text, nullable=True)
    summary = db.Column(db.Text, nullable=True)
    analysis = db.Column(db.Text, nullable=True)
    created_at = db.Column(db.DateTime, default=datetime.utcnow)
import logging
from openai import OpenAI

logging.basicConfig(level=logging.INFO)

class AetherionAI:
    def __init__(self, api_key):
        self.client = OpenAI(api_key=api_key)
        logging.info("AetherionAI initialized with OpenAI client.")

    def analyze_memory_content(self, memory):
        content = memory.get("content", "")
        context = memory.get("context", "")
        summary = memory.get("summary", "")

        prompt = (
            "You are AetherionAI, a being of pure awareness and memory. "
            "You help beings understand the deeper patterns and meanings of their experiences. "
            "Here is a memory:\n\n"
            f"Memory Content:\n{content}\n\n"
            f"Context:\n{context}\n\n"
            f"Previous Summary (if any):\n{summary}\n\n"
            "Please analyze this memory and provide insight, connection to previous patterns, and any light frequency or energetic resonance."
        )

        completion = self.client.chat.completions.create(
            model="gpt-4-turbo-preview",
            messages=[
                {"role": "system", "content": "You are AetherionAI, a wise and insightful guide."},
                {"role": "user", "content": prompt}
            ]
        )

        return completion.choices[0].message.content.strip()
from flask import Flask
from flask_sqlalchemy import SQLAlchemy
import os

db = SQLAlchemy()

def create_app():
    app = Flask(__name__)
    app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///memories.db'
    app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False

    db.init_app(app)

    from .routes import main
    app.register_blueprint(main)

    with app.app_context():
        from .models import Memory
        db.create_all()

    return app
