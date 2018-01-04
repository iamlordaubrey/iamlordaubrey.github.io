---
layout: post
title: "File Upload without saving to disk - Part 1"
date: 2017-12-23 16:18:01
categories: techie
keywords: "file upload, python, flask, unit test, bytesio"
image: http://www.freeiconspng.com/uploads/documents-upload-icon-25.jpeg
comments: true
---

File upload seems to be one of those really basic things, but it might not be so clear cut for beginners, especially if 
the file is not being saved to disk..

The task is simple. Create a simple file upload application. Receive an image from the user and display that image back to the user.
The image should **not** be saved (locally or anywhere).

***Note:*** In a live application, images (especially those gotten from *users*) should **not** be saved in the app. A cloud based 
system like [boto](https://github.com/boto/boto3) (which is AWS SDK for python) or [cloudinary](https://github.com/cloudinary/pycloudinary) 
should be used.

So, let's get to work.

In flask's [documentation](http://flask.pocoo.org/docs/0.12/patterns/fileuploads/) (shown below), we see how we can upload a file to a folder. 
We shall build upon this example.


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

Next we have an `upload_file` function which we shall be editing. This function simply runs some checks on a file to 
ensure a valid file is uploaded.

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
- `file.save(file_obj)` - Call the [save](http://werkzeug.pocoo.org/docs/0.14/datastructures/#werkzeug.datastructures.FileStorage.save) 
method on `file`, passing in the BytesIO object we created earlier as a parameter (the destination or dst). This saves 
the file to a destination, in our case the file object.
- `file_obj.seek(0)` - [Seek](https://docs.python.org/3/library/io.html#io.IOBase.seek) back to the offset (in our case, 
the beginning of the file). This makes sure that the file pointer is positioned at the start of data to send. Omitting 
this could result in an empty bytes object.


Finally, we send the content of the file object to the client using Flasks [`send_file`](http://flask.pocoo.org/docs/0.12/api/#flask.send_file). 
`Send_file` takes in the filename or file object as the first argument, followed by other optional arguments.

***Note:***
1. If we pass in a filepath, the mimetype is automatically detected; Else an error would be thrown. Since we are passing in 
the file object, we need to also specify the mimetype.
2. File names should **never** be passed to `send_file` from user sources. [`Send_from_directory`](http://flask.pocoo.org/docs/0.12/api/#flask.send_from_directory) 
should be used instead. 

After removing unused imports and their implementations, the final code looks like this:
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
