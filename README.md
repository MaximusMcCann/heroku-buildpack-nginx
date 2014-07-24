Heroku buildpack: nginx
=======================

This is a [Heroku buildpack](http://devcenter.heroku.com/articles/buildpack)
for nginx.

Usage
-----

Example usage:

    $ ls -R *

    index.js    Procfile  (not shown)--> .buildpacks

    conf:
    mime.types     nginx.conf.erb

    dist:
    index.html


    $ heroku create --stack cedar --buildpack http://github.com/mccannmax/heroku-buildpack-nginx.git
    ...

    $ git push heroku master
    ...
    -----> Heroku receiving push
    -----> Fetching custom buildpack... done
    -----> Nginx app detected
    -----> Fetching nginx binaries
    -----> Vendoring nginx 1.0.11
    ...

Use [heroku-buildpack-multi](https://github.com/ddollar/heroku-buildpack-multi). Create a .buildpacks file in the root directory and include these lines:
```
https://github.com/mccannmax/heroku-buildpack-nginx.git
https://github.com/heroku/heroku-buildpack-nodejs.git
```

The nginx buildpack will detect your app as nginx if it has the file `nginx.conf.erb` in the `conf` directory.  Copy and paste the conf folder from this repo into your apps base directory to start. You must define all `listen`
directives as `listen <%= ENV['PORT'] %>;` and also include `daemon off;` in
order for this buildpack to work correctly.<--which is done in the example file.  The socket (/tmp/nginx.socket) used in the `nginx.conf.erb` is the 'port' you will listen to when creating your node server.

Example index.js file
```
var express = require('express');
var bodyParser = require('body-parser');
var socket = '/tmp/nginx.socket';

var app = express();
app.use(express.compress());
app.use(bodyParser.json());
app.use(app.router);

app.get('/', function(req, res){
  var result = req.rawBody;
  res.send('SUCCESS: ' + result);
});

app.listen(socket, function() {
  console.log("Express server listening on port/socket: " + socket);
});

```


TODO: check if below still works

Hacking //This is old, but should still work
-------

To modify this buildpack, fork it on Github. Push up changes to your fork, then
create a test app with `--buildpack <your-github-url>` and push to it.

To change the vendored binaries for nginx use the helper script in the
`support/` subdirectory. You may wish to edit the helper script to modify
the nginx build options to suit your needs. You'll need an S3-enabled
AWS account and a bucket to store your binaries in.

For example, you can change the vendored version of nginx to 1.0.12.

First you'll need to build a Heroku-compatible version of nginx:

    $  export AWS_ID=xxx AWS_SECRET=yyy S3_BUCKET=zzz
    $ s3 create $S3_BUCKET
    $ support/package_nginx 1.0.12 8.30

The first argument to the package_nginx script is the nginx version. The
second argument is the version of PCRE to compile nginx against.

Open `bin/compile` in your editor, and change the following lines:

    NGINX_VERSION="1.0.12"
    S3_BUCKET=zzz

Commit and push the changes to your buildpack to your Github fork, then push
your sample app to Heroku to test. You should see:

    -----> Vendoring nginx 1.0.12
