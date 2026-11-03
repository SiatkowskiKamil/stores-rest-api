# Stores REST API

REST API do zarządzania sklepami, produktami i tagami, napisane we Flasku.
Projekt powstał podczas kursu *REST APIs with Flask and Python* i jest moim
pierwszym projektem backendowym — zostawiam go takim, jaki jest, bo pokazuje,
od czego zaczynałem.

## Co robi

| Zasób | Operacje |
|---|---|
| `/store`, `/store/<id>` | lista sklepów, podgląd, utworzenie, usunięcie |
| `/item`, `/item/<id>` | lista produktów, podgląd, utworzenie, aktualizacja, usunięcie |
| `/tag/<id>`, `/store/<id>/tag` | tagi w obrębie sklepu, podgląd, utworzenie, usunięcie |
| `/item/<id>/tag/<id>` | powiązanie i odpięcie tagu od produktu |
| `/register`, `/login`, `/refresh`, `/logout` | rejestracja i logowanie użytkownika |
| `/user/<id>` | podgląd i usunięcie użytkownika |

Operacje zapisujące wymagają tokenu JWT. Wylogowanie unieważnia token przez
listę odrzuconych identyfikatorów (`blocklist.py`), a `/refresh` wydaje token
nieświeży, którego nie przyjmują operacje wymagające świeżego logowania.

Dokumentacja API generuje się sama — po uruchomieniu jest pod
`/swagger-ui` (OpenAPI 3.0.3, przez flask-smorest).

## Czego użyłem

- **Flask** + **flask-smorest** — trasy, walidacja i dokumentacja OpenAPI
- **SQLAlchemy** + **flask-migrate** (Alembic) — model danych i migracje
- **flask-jwt-extended** — tokeny dostępu, odświeżania i unieważnianie
- **passlib** — hashowanie haseł
- **rq** + **redis** — kolejka zadań do wysyłki maila powitalnego (Mailgun)
- **Docker** — obraz z `gunicorn` i wejściem przez `docker-entrypoint.sh`

Model danych: sklep ma wiele produktów i wiele tagów, a produkt i tag łączy
relacja wiele-do-wielu przez tabelę pośrednią (`models/item_tags.py`).

## Jak uruchomić

```bash
python -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install -r requirements.txt
cp .env.example .env               # uzupełnij, jeśli chcesz wysyłkę maili
flask run
```

Bez zmiennej `DATABASE_URL` aplikacja użyje lokalnego SQLite (`data.db`).
Mail powitalny wymaga kluczy Mailguna i działającego Redisa — bez nich
rejestracja zadziała, tylko mail nie wyjdzie.

W Dockerze:

```bash
docker build -t stores-api .
docker run -dp 5000:5000 stores-api
```

## Czego tu świadomie nie ma

- **Testów.** Projekt kursowy, sprawdzany ręcznie w Insomnii.
- **`.env` i bazy danych.** `.env.example` pokazuje, jakie zmienne są potrzebne.
- **Produkcyjnego klucza JWT.** `JWT_SECRET_KEY` czytany jest ze zmiennej
  środowiskowej, a wartość domyślna pochodzi z kursu i nadaje się wyłącznie do
  pracy lokalnej.
