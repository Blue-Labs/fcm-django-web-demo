# fcm-django-web-demo

Quick demo to demonstrate the use of firebase web push notifications with the use of `javascript` on frontend, `django` on backend and push notifications via `fcm-django` pypi package for django.

## Quick start

### prerequisites
- in `fcm-django-web-demo`:
  - create virtual environment with `python -m virtualenv env` (or `python -m venv env` in Python 3)
  - activate virtual environment with `. env/bin/activate`
  - install necessary Python packages with `pip install -r mysite/requirements.txt`
  - configure your FCM at https://console.firebase.google.com/u/0/project/<your project>/settings/cloudmessaging/

### frontend
- in `fcm-django-web-demo/frontend`:
  - run server with `python -m SimpleHTTPServer 8001`

### backend
- in `fcm-django-web-demo/mysite`:
  - run database migrations with `python manage.py migrate`
  - create Django administrator with `python manage.py createsuperuser`
  - collect static files with `python manage.py collectstatic`
  - run server with `python manage.py runserver 0.0.0.0:8000`.

### how to use
- open http://localhost:8001 in your browser of choice
- request token and allow firebase to send notifications to your browser (device)
- you should now be seeing your instance id token on the aforementioned URL
- if you go to django admin, http://localhost:8000/admin/fcm_django/fcmdevice/, you should be seeing a FCMDevice instance for your browser
- send yourself a test notification with django admin actions
  - shell example (run `python manage.py shell` from `fcm-django-web-demo/mysite`):
    ```python
	   from fcm_django.models import FCMDevice
	   device = FCMDevice.objects.all().first()
	   device.send_message(title='title', body='message')
    ```
- voila :)

### NOT really optional HTTPS support
- recommended flow:
  - Use nginx (or other webserver) as your frontend for SSL
  - e.g.:
```nginx
    upstream ab-dev {
        server localhost:8000;
    }

    server {
        # listen to 443 only, IPv4 or IPv6, and accept HTTP2
        listen               443 ssl http2;
        listen               [::]:443 ssl http2;
        server_name          ab-dev.blue-labs.org;

        ssl_certificate      /etc/letsencrypt/live/ab-dev.blue-labs.org/fullchain.pem;
        ssl_certificate_key  /etc/letsencrypt/live/ab-dev.blue-labs.org/privkey.pem;

        # info to properly forward the connection to Django after SSL is
        # setup
        location / {
            proxy_http_version 1.1;
            proxy_read_timeout 300s;
            proxy_set_header Host $host;
            proxy_set_header X-Forwarded-For $remote_addr;
            proxy_set_header X-Forwarded-Proto https;
            proxy_set_header X-Forwarded-Host $host;
            proxy_set_header HTTP-Host $host;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection $connection_upgrade;
            proxy_pass http://ab-dev;
        }

        # used for Let's Encrypt certbot
        location /.well-known/acme-challenge {
            root /var/lib/letsencrypt;
            default_type "text/plain";
            try_files $uri =404;
        }
    }
```

- *why would you want to do this?* because service workers will not work on http, unless you are running them on localhost
- generate certificate and key with `openssl req -nodes -new -x509 -keyout key.pem -out cert.pem` in `fcm-django-web-demo`
- in `fcm-django-web-demo/frontend`:
  - update URL protocol to `https` and `localhost` to your server's IP address in [index.html](https://github.com/xtrinch/fcm-django-web-demo/blob/b8d552830de2b5d82e2d3f787e98d160160c0844/frontend/index.html#L194)
  - run frontend server with `python server.py` 
- in `fcm-django-web-demo/mysite`:
  - add your server's IP address to allowed hosts in project settings (shell example: `echo "ALLOWED_HOSTS = ['172.20.1.10']" > mysite/local_settings.py`)
  - run backend server with `python manage.py runsslserver --certificate ../cert.pem --key ../key.pem 0.0.0.0:8000`
- testing this demo in Chrome may require to run it with `--ignore-certificate-errors` flag to avoid SSL certificate fetch errors
- during the testing allow untrusted connections to the demo servers on browser prompt

### fcm-django DRF API URL docs demo

- available via `coreapi` and `djangorestframework` pypi packages, can be accessed at http://localhost:8000/docs
