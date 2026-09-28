# vercel-python-gis-modern

A Vercel Python runtime with **GEOS, PROJ, and GDAL** available for Django GeoDjango applications.

## Usage with Django

This runtime can be used with a Django project that uses GeoDjango and PostGIS.

A typical project structure might look like:

```text
project/
├── apps/
│   └── shops/
│       ├── models.py
│       ├── views.py
│       └── ...
├── core/
│   ├── settings.py
│   ├── urls.py
│   ├── wsgi.py
│   └── ...
├── manage.py
└── vercel.json
```

The names and structure of your Django project can be different. The important part is that `vercel.json` points to the WSGI entry point of your Django project.

### 1. Configure `vercel.json`

Add the custom runtime as a build:

```json
{
    "builds": [
        {
            "src": "core/wsgi.py",
            "use": "vercel-python-gis-modern@1.0.6"
        }
    ],
    "rewrites": [
        {
            "source": "/(.*)",
            "destination": "core/wsgi.py"
        }
    ]
}
```

Replace `core/wsgi.py` with the path to your own WSGI entry point if it is located elsewhere.

### 2. Configure GeoDjango

In `config/settings.py`, configure the PostGIS database backend:

```python
DATABASES = {
    "default": {
        "ENGINE": "django.contrib.gis.db.backends.postgis",
        # ...
    }
}
```

The runtime provides the native GIS libraries. Configure Django to load them:

```python
PROJ_LIBRARY_PATH = "libproj.so"
GDAL_LIBRARY_PATH = "libgdal.so"
GEOS_LIBRARY_PATH = "libgeos_c.so.1"
```

### 3. Use GeoDjango models

You can use GeoDjango fields normally:

```python
from django.contrib.gis.db import models


class MyModel(models.Model):
    geom = models.PolygonField()
```

### 4. Use GeoDjango functionality

GeoDjango's GEOS functionality is also available:

```python
from django.contrib.gis.geos import Point
from django.http import HttpResponse


def my_view(request):
    point = Point(0, 0)

    return HttpResponse(point.wkt)
```

### 5. Configure the WSGI entry point

The Vercel Python runtime expects the WSGI application to be exposed as `app`.

For example, in `core/wsgi.py`:

```python
import os

from django.core.wsgi import get_wsgi_application

os.environ.setdefault("DJANGO_SETTINGS_MODULE", "config.settings")

app = get_wsgi_application()
```

If your project uses Django's conventional `application` variable, you can expose both:

```python
application = get_wsgi_application()
app = application
```

This allows the same WSGI module to remain compatible with Django's conventional `WSGI_APPLICATION` setting while also exposing `app` for Vercel.

## Development

The runtime contains native GIS libraries required by GeoDjango:

* GDAL
* GEOS
* PROJ
* libjpeg
* libtiff
* libsqlite
* libjbig

The binary libraries are packaged with the runtime and made available to the deployed Vercel function.

### Building the runtime

Start the build container:

```bash
docker-compose run --entrypoint='' builder bash
```

Then run:

```bash
./repo-build.sh
```

This produces the stripped libraries in:

```text
/temp/stripped-files
```

Copy them to:

```text
/dist/files
```

### Changing the runtime source

The runtime source in `src/` is based on Vercel's Python runtime.

After modifying files under `src/`, rebuild the package before publishing:

```bash
npm run build
```

## License

MIT

This project contains code derived from Vercel's Python runtime and distributes binary libraries under their respective licenses:

* GDAL — GDAL license
* GEOS — GEOS license
* libjbig — JBIG license
* libjpeg — libjpeg-turbo license
* PROJ — PROJ license
* libsqlite — SQLite license
* libtiff — libtiff license

See the individual projects for their respective license terms.

## Author

Maintained by [@ShohjahonOktamov](https://github.com/ShohjahonOktamov).

Based on the original [`vercel-python-gis`](https://github.com/jperelli/vercel-python-gis) runtime by [@jperelli](https://github.com/jperelli).

