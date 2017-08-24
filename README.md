# python-django-notes
Notes made while learning Python/Django

Installed python 3.6 and python package manager, 'pip'

### Python Virtual Environments

Create a virtual environment:
`python -m venv .virtualenvs/virtual_env_name`

Activate virtual environment:
`source .virtualenvs/virtual_env_name/bin/activate`


### Django Project Setup

Install Django: 
`pip install django`

Initialise Django project, and create app:

`django-admin startproject project_name`

`cd project_name`

`python manage.py startapp app_name`

Add app to Project settings.py

```
settings.py 

INSTALLED_APPS = (
    ...
    'myapp',
    ...
)
```

To set Django which settings file Django will use:
```
export DJANGO_SETTINGS_MODULE=mysite.settings
```
(By default this will be automatically set)

### Install Django Extensions
A collection of useful command line extensions

`pip install django-extensions`

Add to Project settings.py:

```
INSTALLED_APPS = (
    ...
    'django_extensions',
    ...
)
```

`python manage.py shell_plus`

Launches a Python shell with all your django models imported

`python manage.py runscript <script>`

Runs a Python script within your django environment. Create a folder, `scripts`, in your app. 

Create your script eg. `myscript.py` within this folder.

```
def run():
    # do something here...
```

Run with `python manage.py runscript myscript --traceback`

### Using django-storages with Amazon S3 Storage

AWS S3 Bucket Policy:

```
{
    "Version": "2008-10-17",
    "Statement": [
        {
            "Sid": "PublicReadForGetBucketObjects",
            "Effect": "Allow",
            "Principal": {
                "AWS": "*"
            },
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::myBucket/*"
        },
        {
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::830686100485:user/DjangoS3Storages"
            },
            "Action": "s3:*",
            "Resource": [
                "arn:aws:s3:::myBucket",
                "arn:aws:s3:::myBucket/*"
            ]
        }
    ]
}
```

`pip install django-storages`

`pip install boto3`

Add django-storages and AWS settings to Project settings.py:

`settings.py`

```
INSTALLED_APPS = [
	...  
	'storages',
	...  
]

...

STATIC_ROOT = 'static'

AWS_STORAGE_BUCKET_NAME = 'bucket_name'
AWS_ACCESS_KEY_ID = 'aws_access_key'
AWS_SECRET_ACCESS_KEY = 'aws_secret_access_key'

AWS_S3_CUSTOM_DOMAIN = '%s.s3.amazonaws.com' % AWS_STORAGE_BUCKET_NAME

STATICFILES_STORAGE = 'django.contrib.staticfiles.storage.StaticFilesStorage'
STATICFILES_LOCATION = 'static'
STATIC_URL = '/static/'

DEFAULT_FILE_STORAGE = 'asset_manager.custom_storages.MediaStorage'
MEDIAFILES_LOCATION = 'media'
MEDIA_URL = "https://%s/%s/" % (AWS_S3_CUSTOM_DOMAIN, MEDIAFILES_LOCATION)
```

To allow media and static files to be stored separately, and use AWS S3, create `myapp/custom_storages.py':

```
from django.conf import settings
from storages.backends.s3boto3 import S3Boto3Storage

class StaticStorage(S3Boto3Storage):
    location = settings.STATICFILES_LOCATION

class MediaStorage(S3Boto3Storage):
    location = settings.MEDIAFILES_LOCATION
```

Test Static Files Collection:

```
python manage.py collectstatic
```

### Models
Create model classes in models.py
Add to DB:

```
python manage.py makemigrations
python manage.py migrate
```

### Django Admin View
1. python manage.py createsuperuser
2. Register your models in `admin.py:`

    `admin.site.register(MyModel)`
  
3. Run Django's simple dev server: `python manage.py runserver`
4. Login at http://localhost:8000/admin

### Signals
Used to monitor model pre-save, post-save etc functions, and run some code ie update AWS S3 when a Folder model is renamed

myproject.settings.py:

```
INSTALLED APPS=[
...
```
replace 'myapp' with
```
'myapp.apps.MyappConfig',
...
]



myapp.apps.py:

from django.apps import AppConfig

class MyappConfig(AppConfig):
    name = 'myapp'

    def ready(self):
        import myapp.signals


myapp.signals.py

from django.db.models.signals import pre_save
from django.dispatch import receiver

from .models import MyModel

@receiver(pre_save, sender=MyModel)
def intercept_pre_save(sender, instance, **kwargs):
    print('pre_save signal intercepted')
```

### Django Model Utils
A useful utility package. FieldTracker is particularly helpful for tracking field changes

`pip install django-model-utils`

`from model_utils import FieldTracker`

Note that FieldTrackers track Foreign Key fields by db_column name, not model field name.

To find the db_column name of your fields:

```
>>> for field in MyModel._meta.fields:
        field.get_attname_column 
('id', 'id')
('name', 'name')
('parent_id', 'parent_id')
```

### UnitTests

Delete `tests.py` from your app folder.
Replace with a `tests` folder, and add an empty `__init__.py`
Create a test file of the form `test*.py`:

```
from django.test import TestCase

class ModelTest(TestCase):
    
    def setUp(self):
        # code to run before each test
    def tearDown(self):
        # code to run after each test

    def my_test(self):
        """
        test description
        """
        # test code here ...
        self.assertEqual(a, b)
```

Run tests:
`python manage.py test myapp`


Run a specific test:
`python manage.py test myapp.tests.test_file.test_class.test_function`

### Coverage.py
To test code coverage

```
pip install coverage
coverage run --source='.' manage.py test
coverage report -m

  ALSO
  
covergae html
```
This produces an html report at `project/htmlcov/index.html`

You can use a .coveragerc file, in the same directory as manage.py to specify options:

```
[run]

branch=true

source=asset_manager
       hive
omit=asset_manager/migrations/*
     asset_manager/tests.py
     hive/wsgi.py
     */__init__.py
     manage.py
```

see `https://coverage.readthedocs.io/en/coverage-4.4.1/config.html`

Then you can run: `coverage run manage.py test`

To run tests with a different settings file:
`coverage run manage.py test --settings=project.settings-test`

To run just a single test file eg. model_tests.py
`coverage run manage.py test myapp.tests.model_tests`

To disable signals during certain tests:

```
    signals.post_save.disconnect(save_folder_to_s3, sender=Folder)
    signals.post_delete.disconnect(delete_folder, sender=Folder)
```

Note TestCase methods are run alphabetically, independently of their TestCase class! This means that each method needs to be atomic, and must undo any changes that it has made. So signals will need to be reconnected if a method has disconnected them, after it has finished.

```
    signals.post_save.disconnect(save_folder_to_s3, sender=Folder)
    signals.post_delete.disconnect(delete_folder, sender=Folder)
```

### Logging

```
setings.py:
TIME_ZONE = 'Europe/London'
LOFILE = 'logs/project-logs.txt'

from django.conf import settings

import logging
logging.basicConfig(
    filename=settings.LOGFILE,
    level=logging.INFO,
    format=' %(asctime)s - %(levelname)s - %(message)s'
    )
# logging.disable(logging.CRITICAL)
logging.debug('Start of program')
```

### Boto 3
To get contents of AWS S3 Bucket programmatically (objects returned alphabetically)

```
s3 = boto3.client(
    's3',
    aws_access_key_id = settings.AWS_ACCESS_KEY_ID,
    aws_secret_access_key = settings.AWS_SECRET_ACCESS_KEY)
bucket = settings.AWS_STORAGE_BUCKET_NAME
```

```
bucket_contents = []
for obj in s3.list_objects(Bucket = s3_utils.bucket)['Contents']:
            bucket_contents.append(obj['Key'])
```

To delete programmatically

```
contents = s3.list_objects(Bucket = s3_utils.bucket)
        if 'Contents' in contents:
            for obj in contents['Contents']:
                s3_utils.s3.delete_object(Bucket = s3_utils.bucket, Key = obj['Key'])
```

### Postgres

`pip install psycopg2`

You will need to install Postgres on your machine, then:

```
psql -U postgres
CREATE DATABASE 'db';
CREATE USER 'user' WITH PASSWORD 'password';
GRANT ALL PRIVILEGES ON DATABASE db TO user;
ALTER USER user CREATEDB;
```

settings.py:

```
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql_psycopg2',
        'NAME': 'db',
        'USER': 'user',
        'PASSWORD': 'password',
        'HOST': '',
        'PORT': '',
    }
}
```

`\l` list databases

`\dt` list tables in database

`\q` quit

`\du` list users

`\connect DBNAME` connect to a database

`\conninfo` Connection info

`DROP DATABASE database_name;` To delete a database (not the one currently connected) 

`DELETE FROM django_migrations WHERE app='your-app-name';` Delete all migrations

#### Drop DB and redo migrations (if they have become muddled!):

1. Delete everything in your app/migrations folder (except __init__.py)
2. DROP, CREATE and GRANT PRIVILEGES TO DB
3. python manage.py makemigrations, migrate

see https://simpleisbetterthancomplex.com/tutorial/2016/07/26/how-to-reset-migrations.html

### Customizing the Django Admin

Check out these links:

https://www.youtube.com/watch?v=XphJRQ3AzMU

https://medium.com/@hakibenita/how-to-turn-django-admin-into-a-lightweight-dashboard-a0e0bbf609ad

https://medium.com/@hakibenita/how-to-add-custom-action-buttons-to-django-admin-8d266f5b0d41

### Django REST Framework

Check out this great tutorial:

http://polyglot.ninja/django-building-rest-apis/

```
pip install djangorestframework

INSTALLED_APPS = [
...
rest_framework
...
]
```

##### Subscribers Example

```
models.py:

class Subscriber(models.Model):
    name = models.CharField("Name", max_length=50)
    age = models.IntegerField("Age")
    email = models.EmailField("Email")
    

serializers.py:

from rest_framework import serializers
from . import models

class SubscriberSerializer(serializers.ModelSerializer):
    class Meta:
        model = models.Subscriber
        fields = '__all__'
	
views.py:

from rest_framework.viewsets import ModelViewSet

from .serializers import SubscriberSerializer
from .models import Subscriber

class SubscriberViewSet(ModelViewSet):
    serializer_class = SubscriberSerializer
    queryset = Subscriber.objects.all()

urls.py:

from rest_framework.routers import SimpleRouter

router = SimpleRouter()
router.register("subscribers", SubscriberViewSet)

urlpatterns = router.urls
```

#### That is all!!!

### Authentication

Create a user programmatically:

```
>>> from django.contrib.auth.models import User
>>> user=User.objects.create_user('foo', password='bar')
>>> user.is_superuser=True
>>> user.is_staff=True
>>> user.save()
```

Settings.py:

```
INSTALLED_APPS = [
...
    rest_framework.authtoken
...
]

REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': (
        'rest_framework.authentication.BasicAuthentication',
        'rest_framework.authentication.SessionAuthentication',
        'rest_framework.authentication.TokenAuthentication',
    )
}
```

Permissions: (views.py)

```
from rest_framework.permissions import IsAuthenticated

class SubscriberViewSet(ModelViewSet):
    serializer_class = SubscriberSerializer
    queryset = Subscriber.objects.all()
    permission_classes = (IsAuthenticated,)
```

JSON Web Tokens:

```
pip install djangorestframework-jwt

REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': (
    	...
    	'rest_framework_jwt.authentication.JSONWebTokenAuthentication',
	...
    )
}
```

urls.py:

```
from rest_framework_jwt.views import obtain_jwt_token

urlpatterns = router.urls + [
    url(r'^jwt-auth/', obtain_jwt_token),
]
```

Obtain a token:

```
 curl --request POST \
  --url http://localhost:8000/api/jwt-auth/ \
  --header 'content-type: application/json' \
  --data '{"username": "user", "password": "pw"}'

{"token":"eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoyLCJlbWFpbCI6IiIsInVzZXJuYW1lIjoidGVzdF91c2VyIiwiZXhwIjoxNDk1OTkyOTg2fQ.sWSzdiBNNcXDqhcdcjWKjwpPsVV7tCIie-uit_Yz7W0"}
```

Use token to access collection:

```
curl -H "Content-Type: application/json" -H "Authorization: JWT eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoyLCJlbWFpbCI6IiIsInVzZXJuYW1lIjoidGVzdF91c2VyIiwiZXhwIjoxNDk1OTkyOTg2fQ.sWSzdiBNNcXDqhcdcjWKjwpPsVV7tCIie-uit_Yz7W0" -X GET  http://localhost:8000/api/subscribers/
```


### Timezones

Auto-setting uploaded_by and uploaded_at fields

settings.py:

```
TIME_ZONE = 'Europe/London'
USE_TC = True
```


admin.py:
```
MyModelAdmin(admin.ModelAdmin):

from django.conf import settings
from datetime import datetime
from pytz import timezone

def save_model(self, request, obj, form, change):
        if not change:
            obj.uploaded_by = request.user
            obj.uploaded_at = datetime.now(timezone(settings.TIME_ZONE))
            super(MyModelAdmin, self).save_model(request, obj, form, change)
```
