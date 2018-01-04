---
layout: post
title: "File Upload without saving to disk - Part 1"
date: 2017-12-23 16:18:01
categories: techie
keywords: "file upload, python, flask, unit test, bytesio"
image: http://www.freeiconspng.com/uploads/documents-upload-icon-25.jpeg
comments: true
---

The image should **not** be saved (locally or anywhere).

***Note:*** In a live application, images (especially those gotten from *users*) should **not** be saved in the app. A cloud based 
system like [boto](https://github.com/boto/boto3) (which is AWS SDK for python) or [cloudinary](https://github.com/cloudinary/pycloudinary) 
should be used.

```python
import os
from flask import Flask, request, redirect, url_for
from werkzeug.utils import secure_filename

UPLOAD_FOLDER = '/path/to/the/uploads'
ALLOWED_EXTENSIONS = set(['txt', 'pdf', 'png', 'jpg', 'jpeg', 'gif'])

app = Flask(__name__)
app.config['UPLOAD_FOLDER'] = UPLOAD_FOLDER

def allowed_file(filename):
    return '.' in filename and \
           filename.rsplit('.', 1)[1].lower() in ALLOWED_EXTENSIONS

@app.route('/', methods=['GET', 'POST'])
def upload_file():
    if request.method == 'POST':
        # check if the post request has the file part
        if 'file' not in request.files:
            flash('No file part')
            return redirect(request.url)
        file = request.files['file']
        # if user does not select file, browser also
        # submit a empty part without filename
        if file.filename == '':
            flash('No selected file')
            return redirect(request.url)
        if file and allowed_file(file.filename):
            filename = secure_filename(file.filename)
            file.save(os.path.join(app.config['UPLOAD_FOLDER'], filename))
            return redirect(url_for('uploaded_file',
                                    filename=filename))
    return '''
    <!doctype html>
    <title>Upload new File</title>
    <h1>Upload new File</h1>
    <form method=post enctype=multipart/form-data>
      <p><input type=file name=file>
         <input type=submit value=Upload>
    </form>
    '''
```

First, we have an `allowed_file` function that takes in a filename, checks if that filename is valid and returns a boolean; 
True if valid and False otherwise.

If a file is uploaded and the filename is valid, i.e.
```python
...
if file and allowed_file(file.name):
    # Save file in-memory
    file_obj = BytesIO()
    file.save(file_obj)

    # Head back to the beginning of the file
    file_obj.seek(0)

    return send_file(file_obj, mimetype=file.content_type)
```


- `file_obj = BytesIO()` - Create a [BytesIO](https://docs.python.org/3/library/io.html#io.BytesIO) object

Finally, we send the content of the file object to the client using Flasks [`send_file`](http://flask.pocoo.org/docs/0.12/api/#flask.send_file). 
`Send_file` takes in the filename or file object as the first argument, followed by other optional arguments.

```python
from flask import Flask, request, redirect, flash, send_file
from io import BytesIO

ALLOWED_EXTENSIONS = set(['txt', 'pdf', 'png', 'jpg', 'jpeg', 'gif'])

app = Flask(__name__)


def allowed_file(filename):
    return '.' in filename and \
           filename.rsplit('.', 1)[1].lower() in ALLOWED_EXTENSIONS


@app.route('/', methods=['GET', 'POST'])
def upload_file():
    if request.method == 'POST':
        print('Post method')
        # check if the post request has the file part
        if 'file' not in request.files:
            flash('No file part')
            return redirect(request.url)
        file = request.files['file']
        # if user does not select file, browser also
        # submit a empty part without filename
        if file.filename == '':
            flash('No selected file')
            return redirect(request.url)
        if file and allowed_file(file.filename):
            # Save file in-memory
            file_obj = BytesIO()
            file.save(file_obj)

            # Head back to the beginning of the file
            file_obj.seek(0)

            return send_file(file_obj, mimetype=file.content_type)
    return '''
    <!doctype html>
    <title>Upload new File</title>
    <h1>Upload new File</h1>
    <form method=post enctype=multipart/form-data>
      <p><input type=file name=file>
         <input type=submit value=Upload>
    </form>
    '''


if __name__ == "__main__":
    app.run()

```
In part 2, we would look at testing the application. That's all folks!

[//]: # (Current project structure:)

<!---
- pu-tutorial
  - image_upload
    - templates
      - index.html
    - \_\_init__.py
    - app.py
  - .gitignore
  - README.md
  - requirements.txt

In the applications root folder, create a folder named `image_upload`.
-->
