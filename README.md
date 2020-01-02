# Documentation of quick tips to build a Flask app

## Project structure
Run this in the project folder
```
mkdir app app/templates app/static app/static/js app/static/css app/blueprints app/middlewares app/models app/static/images app/uploads
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
export NEW_BLUEPRINT=predict_api
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

**VND Classifier Example**

*home_page.html*
```
<div class="container">
    <br><br>
    <img class="mb-4" src="static/images/coderschool-logo.png" alt="">
    <br><br>
    <div class="row">
        <div class="col-md-6">
            <div id="my-camera"></div>
            <br>
            <form id="form-capture-image">
                <a href="#" class="btn btn-lg btn-primary btn-capture-image">Predict Now</a>
            </form>
        </div>
        <div class="col-md-6">
            <div id="results">
                <div id="loading" class="hidden spinner-border text-warning" role="status">
                    <span class="sr-only">Loading...</span>
                </div>
                <h3 id='class-result'>Predictions:</h3>
                <img id="taken-photo" src="static/images/1567072883.195046.jpg" alt="" />
                <div id="results-prediction">
                    <p id="prediction"></p>
                    <p id="probs"></p>
                </div>
            </div>
        </div>
    </div>
</div>
```

*index.js*
```
CAPTURE_IMG_WIDTH = 640
CAPTURE_IMG_HEIGHT = 480

jQuery.ajaxSetup({
  beforeSend: function() {
     $('#loading').removeClass('hidden');
  },
  complete: function(){
     $('#loading').addClass('hidden');
  },
  success: function() {
    $('#loading').addClass('hidden');
  }
});

// HTML5 WEBCAM
Webcam.set({
  width: CAPTURE_IMG_WIDTH,
  height: CAPTURE_IMG_HEIGHT,
  image_format: 'jpeg',
  jpeg_quality: 90
});
Webcam.attach( '#my-camera' );

let form_capture = document.getElementById('form-capture-image')
$('.btn-capture-image').on('click', function(e) {
  e.preventDefault();

  Webcam.snap(function(data_uri) {
    // display results in page
    // readURL(data_uri, '#input-data-uri')
    let json_data = {'data-uri': data_uri }

    $.ajax({
      type: 'POST',
      url: '/predict/',
      processData: false,
      contentType: 'application/json; charset=utf-8',
      dataType: 'json',
      data: JSON.stringify(json_data),
      success: function(data) {
        console.log(data)
        html = '<ul>'
        for( let i = 0; i < data['probs'].length; i++) {
          data_splitted = data['probs'][i]

          html += '<li><span class="num">' + data_splitted[0] + '</span> <span class="prob">'+ data_splitted[1] + '</span></li>'
        }
        html += '</ul>'

        $('#probs').text('').append(html)
        $('#class-result').text('Predictions: ' + data['label']);

        // $('.box-main').css('height', $('.box-results').height());
      }
    });
  });
});
```

*predict_api/blueprint.py*
```
from flask import Blueprint, jsonify, request
import tensorflow as tf
import numpy as np
import re
import os
import base64
import uuid

predict_api = Blueprint('predict', __name__)

model = tf.keras.models.load_model("models/vnd_classifier.h5")
class_names = np.array(['1000', '2000', '5000', '10000', '20000', 
                        '50000', '100000', '200000', '500000'])

def parse_image(imgData):
    img_str = re.search(b"base64,(.*)", imgData).group(1)
    img_decode = base64.decodebytes(img_str)
    filename = "{}.jpg".format(uuid.uuid4().hex)
    with open('uploads/'+filename, "wb") as f:
        f.write(img_decode)
    return img_decode

def preprocess(image):
    image = tf.image.decode_jpeg(image, channels=3)
    image = tf.image.resize(image, [192, 192])
    # Use `convert_image_dtype` to convert to floats in the [0,1] range.
    image = tf.image.convert_image_dtype(image, tf.float32)
    image = (image*2) - 1  # normalize to [-1,1] range
    image = tf.image.per_image_standardization(image)
    return image

@predict_api.route('/predict/', methods=['POST'])
def predict():
    data = request.get_json()

    # Preprocess the upload image
    img_raw = data['data-uri'].encode()
    image = parse_image(img_raw)
    image = preprocess(image)
    image = tf.expand_dims(image, 0)

    probs = model.predict(image)
    label = np.argmax(probs, axis=1)
    label = class_names[label[0]]
    probs = probs[0].tolist()
    probs = [(probs[i], class_names[i]) for i in range(len(class_names))]

    return jsonify({'label': label, 'probs': probs})
```