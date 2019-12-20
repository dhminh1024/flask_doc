# Documentation of quick tips to build a Flask app

## Project structure
Run this in the project folder
```
mkdir app app/templates app/static app/static/js app/static/css app/blueprints app/middlewares app/models
touch app/main.py app/templates/base.html app/static/js/index.js app/static/css/style.css
touch app/blueprints/__init__.py
```

Using Canvas or jquery (optional)
```
cp docs/index.js app/static/js/index.js
cp docs/jquery-3.4.1.min.js app/static/js/
cp docs/images/coderschool-logo.png app/static/images/
```

**app/main.py**
VSCode Extension needed: flask-snippets
Start with `fapp`, remove the route

## Create a new blueprint

Run this in the project folder. Change 'home_page' to make a new blueprint
```
export NEW_BLUEPRINT=upload_api
mkdir app/blueprints/$NEW_BLUEPRINT
touch app/blueprints/$NEW_BLUEPRINT/__init__.py app/blueprints/$NEW_BLUEPRINT/blueprint.py
echo "from .$NEW_BLUEPRINT import $NEW_BLUEPRINT" >> app/blueprints/__init__.py
echo "from .blueprint import $NEW_BLUEPRINT" > app/blueprints/$NEW_BLUEPRINT/__init__.py
printf \
"from flask import Blueprint, render_template, request\n\
\n\
$NEW_BLUEPRINT = Blueprint('$NEW_BLUEPRINT', __name__)\
\n\
@$NEW_BLUEPRINT.route('/route_name')\n\
def route_name():\n\
    return render_template('$NEW_BLUEPRINT.html') \n\
" > app/blueprints/$NEW_BLUEPRINT/blueprint.py
cp docs/sample_page.html app/templates/$NEW_BLUEPRINT.html
```

In **app/main.py**:
```
from blueprints import *

app.register_blueprint(home_page)
```

## HTML Template

**app/template/base.html**
Extension: Bootstrap 4, Font awesome 4, Font Awesome 5 Free & Pro snippets
```
b4-$
```
Add javascript and block content to the body
```
<head>
    ...
    <link href="static/css/style.css" rel="stylesheet">
</head>
<body>
    {% block content%} {% endblock %}

    <script src="static/js/jquery-3.4.1.min.js"></script>
    <script src="static/js/index.js"></script>
    <!-- AJAX optional -->
    <script type="text/javascript">
        $("#myButton").click(function(){
            $('#result').text('  Predicting...');
            var $SCRIPT_ROOT = {{request.script_root|tojson|safe}};
            var canvasObj = document.getElementById("canvas");
            var img = canvasObj.toDataURL('image/jpeg');
            $.ajax({
                type: "POST",
                url: $SCRIPT_ROOT + "/upload/",
                data: img,
                success: function(data){
                    $('#result').text('Predictions ' + data);
                }
            });
        });
    </script>
</body>
```

**MNIST Example**: *app/templates/home.html*
```
<div>
    <img class="mb-4" src="static/coderschool-logo.png" alt="">
    <h1 class="h3 mb-3 font-weight-normal">Please draw a number</h1>

    <canvas id='canvas' width="400" height="400"></canvas>

    <h1 class="h3 mb-3 font-weight-normal" id="result">Predictions: </h1>

    <button id="myButton" class="btn btn-lg btn-primary btn-block" type="submit">Predict</button>
    <button id="clearButton" class="btn btn-lg btn-success btn-block" type="submit">Clear</button>

    <p class="mt-5 mb-3 text-muted">&copy; Mariana 2019</p>
</div>
```

## Image classification on Flask

**Load model**
```
model = tf.keras.models.load_model('models/mnist.h5')
```

**Handle Base64 encoded image from request.data()**
```
def parse_image(imgData):
    img_str = re.search(b"base64,(.*)", imgData).group(1)
    img_decode = base64.decodebytes(img_str)
    with open('output.png', "wb") as f:
        f.write(img_decode)
    return img_decode
```

**Preprocess image with TensorFlow**
```
image = parse_image(request.get_data())
image = tf.image.decode_jpeg(image, channels=1)
image = tf.image.resize(image, [28, 28])
# Change background to black and normalize data
image = (255 - image)/255.0
image = tf.reshape(image, (1, 28, 28))

prediction = model.predict(image)
prediction = np.argmax(prediction, axis=1)
return str(prediction)
```
