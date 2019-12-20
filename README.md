# Documentation of quick tips to build a Flask app

## Project structure

```
mkdir app app/templates app/static app/static/js app/static/css app/blueprints app/blueprints/home app/middlewares app/models
touch app/main.py app/app.yaml app/templates/base.html app/templates/home.html app/static/js/index.js app/static/css/style.css
touch app/blueprints/__init__.py
export NEW_BLUEPRINT=home # Change 'home' to make a new blueprint
mkdir app/blueprints/$NEW_BLUEPRINT
touch app/blueprints/$NEW_BLUEPRINT/__init__.py app/blueprints/$NEW_BLUEPRINT/blueprint.py
touch app/requirements.txt
```

**app/template/base.html**
Extension: Bootstrap 4, Font awesome 4, Font Awesome 5 Free & Pro snippets
```
b4-$
```
Add javascript
```
<script src="static/index.js"></script>
```

**app/blueprints/__init__.py**
```
from .home import home_page
```
**app/blueprints/home/__init__.py**
```
from .blueprint import home_page
```
**app/blueprints/home/blueprint.py**
```
from flask import Blueprint, render_template

home_page = Blueprint('home_page', __name__)

@home_page.route('/')
def index():
    return render_template('home.html')
```
**app/main.py**
```
from blueprints import *

app.register_blueprint(home_page)
```