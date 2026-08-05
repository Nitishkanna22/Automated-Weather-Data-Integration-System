# Weather Pipeline Console (Django frontend)

A Django UI on top of your existing pipeline modules. It does **not**
reimplement fetching, transforming, or loading — `dashboard/services.py`
is the only file that imports `Fetcher`, `Transformer`, and `Loader`, and
it calls them exactly the way `Pipeline.py` already does.

## What's here

```
weather_dashboard/
├── manage.py
├── Config.py, Fetcher.py, Transformer.py, Loader.py, Pipeline.py, Queries.sql   ← your existing modules, unmodified
├── weather_web/            ← Django project (settings, urls, wsgi/asgi)
└── dashboard/               ← Django app (the actual console)
    ├── models.py            PipelineRun — a local run log (SQLite), separate from the
    │                        BigQuery weather data itself
    ├── forms.py             RunPipelineForm, AnalyticsQueryForm
    ├── services.py           bridge to Fetcher / Transformer / Loader + BigQuery analytics
    ├── views.py, urls.py
    ├── templates/dashboard/  run_console.html, run_history.html, run_detail.html, analytics.html
    ├── static/dashboard/css/dashboard.css
    └── templatetags/dashboard_extras.py   `get_item` filter for dict-by-variable-key lookups in templates
```

### Pages

- **`/` — Run.** Pick cities (defaults to `Config.CITIES`, or add your own),
  a day range, and dry-run or load-to-BigQuery. Runs `Fetcher →
  Transformer → (Loader)` and shows a live readout plus a preview table
  of the transformed rows.
- **`/runs/` and `/runs/<id>/` — Run log.** Every run is recorded locally
  in SQLite (`PipelineRun`) — status, row counts, errors — independent of
  whether BigQuery is reachable, so dry runs still leave a history.
- **`/analytics/`** — the seven reports from `Queries.sql`, re-expressed
  as parameterized queries in `services.ANALYTICS_QUERIES` and run live
  against `BIGQUERY_PROJECT_ID.BIGQUERY_DATASET_ID.BIGQUERY_TABLE_ID`
  from `Config.py`. If BigQuery credentials aren't configured, the page
  still renders — it just shows the error inline instead of a table.

## Setup

```bash
python -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install -r requirements.txt

python manage.py migrate
python manage.py createsuperuser   # optional, for /admin/
python manage.py runserver
```

## Notes

- **BigQuery auth** works the same as it does for `Pipeline.py` today —
  set `GOOGLE_APPLICATION_CREDENTIALS` to your service-account key path.
  Nothing in the Django app changes that.
- **`Config.py` is still the source of truth.** City list, date defaults,
  and the BigQuery project/dataset/table all come from there. Change it
  once, both the CLI (`Pipeline.py`) and the web console pick it up.
- **Safety limits** on the web form (max days, max cities per run) live
  in `weather_web/settings.py` as `PIPELINE_MAX_DAYS` /
  `PIPELINE_MAX_CITIES`, since a browser form is easier to fat-finger
  into a huge request than a CLI flag.

  
- The **local SQLite DB only stores run metadata** (`PipelineRun`), never
  weather observations — those still live exclusively in BigQuery, loaded
  by the unmodified `Loader.py`.
- For production: set `DJANGO_DEBUG=0`, a real `DJANGO_SECRET_KEY`, and
  `DJANGO_ALLOWED_HOSTS`, then run behind gunicorn/uwsgi + a real
  webserver instead of `runserver`.

## Interface of the Project

![image_alt](https://github.com/Nitishkanna22/Automated-Whether-Data-Website/blob/b28a29e6706144c5e5a4324285a761bdf48ed4a8/Front%20Page.png)
![image_alt](https://github.com/Nitishkanna22/Automated-Whether-Data-Website/blob/af4dc5358dd25ab5d45e3bb1273102ab75fd550b/Front%20page%20search%20results.png)
![image_alt](https://github.com/Nitishkanna22/Automated-Whether-Data-Website/blob/af4dc5358dd25ab5d45e3bb1273102ab75fd550b/Analytics%20page.png)
![image_alt](https://github.com/Nitishkanna22/Automated-Whether-Data-Website/blob/af4dc5358dd25ab5d45e3bb1273102ab75fd550b/Analytics%20search%20results.png)
