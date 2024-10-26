# Moravian Web Apps

## Application Configuration

* to be deployable on `webapps.cs.moravian.edu` your application must be able to accept traffic via a TCP socket (standard ip address and port combination) or a Unix Domain Socket (a path to a socket file on the system)
* TCP sockets are easier to configure, but can have accidental collusions due to ports
* Unix Domain sockets require additional application configuration, but provide better performance per request

## Nginx Configuration

* you provide two files, a configuration file for the location of the server and one for the configuration of routes

### Upstream Blocks

```nginx
upstream your_weather_app {
    server 127.0.0.1:8080;
}
```

```nginx
upstream your_uwsgi_app {
    # if listening via default
    server 127.0.0.1:9095;
    # if listening via Unix socket
    server unix://tmp/uwsgi.sock;
}
```


### Location Blocks

#### Simple Proxy

```nginx
location /<your_app> {
    include proxy_params;
    proxy_pass http://your_weather_app;
}
```

#### Absolute Paths

If you are using absolute paths in your application you must prepend them with `/<your_app>`.
For flask the most convenient way is to set the environment variable `SCRIPT_NAME` to `/<your_app>`, then adjust any hardcoded absolute urls to use `url_for` in a template.
Here is an example

<details>
<summary>templated html example</summary>

```html
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link rel="stylesheet" type="text/css" href="{{ url_for('static', filename='style.css') }}">
    <!-- DO NOT DO THIS <link rel="stylesheet" type="text/css" href="/static/style.css"> -->
    <!-- Remote resources do not need to be adjusted -->
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
</head>
<body>
...
</body>
```


</details>

Alternatively for purely static content the html `<base>` element can be used.
[See MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/base) for more details.

#### Serving Static Files using nginx

Nginx is capable of serving static files.
It is recommended to serve static files directly from nginx for improved performance and reduced application load.
An example block would be as follows

```nginx
location /<your_app>/<your_static_dir> {
    alias /path/to/app_dir/your_static_dir/;
    auto_index off;
    try_files $uri $uri/ =404;
}
```

If you would like to display a different page when a file can't be found you can provide a path to the file

```nginx
location /<your_app>/static {
    alias /path/to/app/your_static_dir/;
    auto_index off;
    try_files $uri $uri/ a_nicer_404.html;
}
```

#### Using uWSGI

If you would like to run an application via wsgi copy the below config


```nginx
locaiton /<your_app> {
    include /etc/nginx/uwsgi_params;
    uwsgi_pass your_uwsgi_app;
}
```

### Advanced Configuration

#### OAUTH-2


