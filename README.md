# Prompt-Sphere

A Django REST Framework API for a community-driven prompt-sharing platform: users create a profile and share prompts, and anyone can browse what has been shared.

This repository contains the **backend API only** — there is no frontend in this repo.

## Stack

- **Django 5.0.2** + **Django REST Framework**
- **Token authentication** (DRF `authtoken`), with session authentication for the browsable API
- **SQLite** database
- `django-cors-headers` with CORS open during development

> Note: the repo has no `requirements.txt` yet, so the dependencies are installed manually (see [Getting started](#getting-started)).

## Features

The endpoint list below comes from the URL configuration in `Interact/`; known problems are flagged inline and summarized in [Current state](#current-state-honest-notes).

### Prompts — `/prompts/`

| Method | Endpoint | What it does |
|--------|----------|--------------|
| GET | `/prompts/` | List all prompts |
| POST | `/prompts/create/` | Create a prompt; the owner is looked up by the `email` of an existing profile |
| POST | `/prompts/my-prompts/` | List the prompts of the profile matching the given `email` |
| PATCH | `/prompts/<id>/edit/` | Update a prompt — currently requires the full field set; a partial PATCH returns 400 |
| DELETE | `/prompts/<id>/delete/` | Delete a prompt |

### Users and profiles — `/users/`

| Method | Endpoint | What it does |
|--------|----------|--------------|
| POST | `/users/auth/` | Obtain an auth token (DRF built-in) |
| POST | `/users/auth/v2/` | Obtain an auth token (custom view) |
| POST | `/users/create-profile/` | Create a GitHub-style profile (username, email, image URL; email must be unique) |
| GET | `/users/` | List GitHub-style profiles |
| GET | `/users/<username>/` | Retrieve a profile (requires authentication) — **currently broken** (errors at runtime) |
| DELETE | `/users/delete-profile/<username>/` | Delete a profile (requires authentication) — **currently broken** (always returns 403, even for the owner) |

There is also `GET /api/`, a route index that is **currently broken** (returns 500).

### Models (`Interact/users/models.py`, `Interact/prompts/models.py`)

- `Profile` — one-to-one with Django's `User`, with optional Facebook and GitHub links
- `ProfileGithub` — username, email and image URL, used as the author of prompts
- `Prompt` — `tag`, `body`, `created` timestamp, and an owner (`ProfileGithub`)

Deleting a `Profile` also deletes its linked Django `User` (signal in `users/signals.py`).

## Getting started

```bash
cd Interact

# (recommended) virtual environment
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate

pip install django djangorestframework django-cors-headers

python manage.py migrate
python manage.py runserver
```

The API is then available at `http://127.0.0.1:8000/`. Example flow:

1. `POST /users/create-profile/` with `username`, `email` and `image`
2. `POST /users/auth/` with a Django user's `username` / `password` to get a token
3. Use the prompt endpoints (pass the token as `Authorization: Token <key>` where authentication is required)

A SQLite `db.sqlite3` is committed with the repo. Django admin is available at `/admin/` (create a superuser with `python manage.py createsuperuser`).

## Current state (honest notes)

This is an early-stage learning/portfolio project, and the code reflects that:

- Development settings are committed (`DEBUG = True`, open CORS, generated secret key) — fine for local work, not production-ready.
- Authentication scaffolding (token auth, permission classes, owner checks) exists, but the prompt endpoints are currently **open**; only some profile endpoints actually require authentication.
- Two profile endpoints are currently non-functional: `GET /users/<username>/` errors at runtime, and `DELETE /users/delete-profile/<username>/` always returns 403, even for the owner. The `GET /api/` route index is also broken (returns 500).
- The prompt edit endpoint (`PATCH /prompts/<id>/edit/`) currently requires the full field set; partial updates return 400.
- Social features are limited to profile links (Facebook/GitHub fields); there are no likes, comments or follows.
- No tests (the test files are Django placeholders) and no `requirements.txt`.
- Some views and URL routes are commented-out experiments kept for reference.

---

Built by [Osama Doage](https://osamadoage.vercel.app).
