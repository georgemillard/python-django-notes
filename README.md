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

### Using django-storages with Amazon S3 Storage

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

STATICFILES_LOCATION = 'static'
# Tell the staticfiles app to use S3Boto3 storage when writing the collected static files (when you run `collectstatic`).
STATICFILES_STORAGE = 'myapp.custom_storages.StaticStorage'
STATIC_URL = "https://%s/%s/" % (AWS_S3_CUSTOM_DOMAIN, STATICFILES_LOCATION)

MEDIAFILES_LOCATION = 'media'

MEDIA_URL = "https://%s/%s/" % (AWS_S3_CUSTOM_DOMAIN, MEDIAFILES_LOCATION)
DEFAULT_FILE_STORAGE = 'myapp.custom_storages.MediaStorage'
```

Test Static Files Collection:

```
python manage.py collectstatic
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

### Unit Testing
Django uses Python's default UnitTest module, but this lacks coloured output (what an oversight). 'Green' is a good alternative.

```
pip install green
python manage.py test --testrunner=green.djangorunner.DjangoRunner
 OR
settings.py:
TEST_RUNNER="green.djangorunner.DjangoRunner"

python manage.py test
```
