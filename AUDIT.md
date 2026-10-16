# AUDIT: testing_project

Ez az audit egy egyszerű, oktatási célú Django ToDo alkalmazásról készült (repo: dev-nor-django/testing_project). Az elemzés a tárolt fájlok és kód alapján készült.

## What this is
Egyszerű Django alapú ToDo alkalmazás oktatási/gyakorló célra — feladatok létrehozása, szerkesztése, törlése és státusz jelölése; valószínű célközönség: Django-tanulók vagy egy rövid gyakorló óra hallgatói.

### Stack
- **Language(s):** Python (elsődleges), HTML (sablonok)
- **Framework / runtime:** Django
- **Notable libraries:** Django, SQLite (db.sqlite3)

## How it's organized
```
FELADAT.txt        -> feladat leírása (magyar)
README.md         -> rövid leírás
db.sqlite3        -> fejlesztői SQLite adatbázis (migrációk már alkalmazva lehetnek)
manage.py         -> Django CLI entrypoint
templates/        -> HTML sablonok (nem olvasott fájlok itt)
todo/             -> Django app: modellek, nézetek, URL-ek, admin, migrációk, tesztek
todo_main/        -> top-level mappa létezik (nem lett belenézve)
```

How it fits together:
- A fő alkalmazás a `todo/` app. A `Task` modell (`todo/models.py`) tárolja a feladatot, státuszokat és időbélyegeket. A `todo/views.py` kezeli a CRUD műveleteket (`addtask`, `edit_task`, `delete_task`, `mark_as_done/undone`) és sablonok renderelését (pl. `edit_task.html`). Az URL-eket a `todo/urls.py` regisztrálja (pl. `addtask/`, `edit_task/<pk>/`). A `manage.py` indítja a projektet; a `db.sqlite3` a fejlesztői DB-t tartalmazza.

## How to run it
Rövid út egy friss clone-tól a futtatásig (feltételezve, hogy Python és pip telepítve van):

```bash
# ajánlott: virtuális környezet
python -m venv .venv
source .venv/bin/activate   # Windows: .venv\Scripts\activate

# nincs requirements.txt a repo-ban — telepítsd a Django-t (válaszd a verziót, ha szükséges)
pip install django

# ha szükséges: migrációk alkalmazása (lehet, hogy db.sqlite3 már tartalmazza a sémát)
python manage.py migrate

# hozd létre a superuser-t adminhoz (opcionális)
python manage.py createsuperuser

# indítsd a fejlesztői szervert
python manage.py runserver
```

Figyelmeztetések/megjegyzések futtatáshoz:
- Nincs `requirements.txt` vagy `pyproject.toml` — nincs rögzített Django verzió. Ajánlott egy ismert stabil Django verziót használni (pl. 4.x).
- `db.sqlite3` a repo-ban megtalálható — ez azt jelenti, hogy a fejlesztői adatbázis jelen van; ha tiszta környezetet szeretnél, töröld vagy nevezd át, majd futtasd a `migrate`-et.

## Audit megállapítások — konkrét kódrészletekre alapozva
- Modellezés: `todo/models.py` — `Task` modell jól definiált (`task`, `is_completed`, `created_at`, `updated_at`). Admin megjelenítés rendben (`todo/admin.py`).
- URL-ek: `todo/urls.py` — az útvonalak alapvetően megvannak (`addtask/`, `mark_as_done/`, `edit_task/`, `delete_task/`). Észrevehető inkonzisztencia: a legtöbb path végén `/` van, de a `delete_task` útvonalból hiányzik a lezáró slash (`path('delete_task/<int:pk>', ...)`), ez okozhat eltéréseket a helyes URL-összeállításban.
- 'homepage' redirect: a nézetek többször `redirect('homepage')`-t használnak, azonban a vizsgált fájlok között nincs `homepage` névhez tartozó view vagy `urls.py` bejegyzés — valószínű, hogy a projekt más részén (pl. `todo_main` vagy projekt szintű `urls.py`) van definiálva, vagy hiányzik. Ezt érdemes ellenőrizni.
- Input validáció / hibakezelés: `addtask()` (`todo/views.py`) közvetlenül használja `request.POST['task']` — ha a mező hiányzik, KeyError dobódik. Javasolt: Django `forms` használata vagy `request.POST.get('task')` + validáció, és felhasználóbarát hibakezelés.
- CSRF / biztonság: nem vizsgáltuk a sablonokat; a nézetek POST-okat fogadnak, ezért győződj meg arról, hogy a sablonok tartalmazzák a CSRF tokent (`{% csrf_token %}`) és a formok megfelelően érvényesítenek.
- Jogosultságok: nincs látható autentikációs/authorization logika a vizsgált view-ekben. Ha több felhasználós környezetre szánod, érdemes korlátozni a szerkesztés/törlés/állapotváltás hozzáférését (pl. `@login_required` dekorátor vagy per-object permissionek).
- Kódminőség / stílus: `delete_task()` definiálásában a behúzás és kommentek (pl. kommentelt `task.save()`) rendezetlenségre utalnak — kisebb refaktor és linting javasolt (flake8/black).
- Tesztek: `todo/tests.py` létezik, de nincs érdemi tartalom a fájlban. Javasolt unit/integration tesztek hozzáadása (model CRUD, view-ek, form validáció, URL-ek).
- Migrációk: `todo/migrations/0001_initial.py` jelen van — a modell sémája rögzítve lett.

## Javasolt következő lépések (prioritással)
1. Biztonság és validáció
   - Minden POST nézethez biztosíts form-validációt (Django `forms`) és ellenőrizd a sablonokban a `csrf_token` meglétét.
   - Védd a szerkesztés/törlés/állapotváltás végpontokat autentikációval/authorization-nel.
2. Stabil futtatás és reprodukálhatóság
   - Hozz létre `requirements.txt`-t (pl. `django>=4.2,<5`) vagy `pyproject.toml`/`constraints.txt`-et.
   - Távolítsd el vagy ignoráld (`.gitignore`) a `db.sqlite3`-t, vagy legalább dokumentáld, hogy fejlesztői DB.
3. Kódminőség és tesztek
   - Futtass lintinget és formázást (black, flake8) és javítsd a kisebb style/problema pontokat.
   - Implementálj alapvető teszteket (`tests.py` bővítése vagy pytest + pytest-django használata).
4. Dokumentáció
   - Bővítsd a README-t futtatási és telepítési instrukciókkal, valamint a projekt szerkezetének rövid leírásával.

## Kérdések, amiket érdemes feltenni a repo-n dolgozóktól
- Hol van definiálva a `homepage` URL/nézet — a projekt gyökér vagy a `todo_main/` `urls.py`-ában található ez? (ellenőrizd a projekt szintű urls.py-t)
- Milyen sablonfájlok vannak a `templates/` könyvtárban — tartalmaznak-e `csrf_token`-eket és a formok validációját? Van-e `edit_task.html` és a kezdőoldal sablonja?
- Szándékos-e, hogy nincs `requirements.txt` / nincs rögzített Django verzió? Szeretnétek, hogy létrehozzak egy `requirements.txt`-t és egyszerű setup/README futtatási útmutatót?

---

Audit elkészítve a tárolt fájlok és kód alapján. Ha szeretnéd, készíthetek:
- kódfix javaslatokat (diff / patch formájában) a bemutatott problémákra (CSRF, forms, login_required),
- `requirements.txt` hozzáadását,
- `templates/` tartalmának ellenőrzését és javítását.
