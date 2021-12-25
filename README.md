# Pràctica CI - Pol Rivero

## Vídeo amb el funcionament:
https://drive.google.com/file/d/1YrSeRII2t1cpR1WHyMAINwVrEniTwqHq/view?usp=sharing

Requereix compte de Google de la UPC.


## Explicació del CI
L'objectiu d'aquesta pràctica és afegir *Continuous Integration* a un repositori de GitHub, fet que permet assegurar que tots els commits afegits al repositori passen tots els tests especificats.

Per fer-ho, cal crear un fitxer `python-app.yml` dins del directori `.github/workflows`, en el qual s'especifiquen els passos a seguir quan es fa un commit.

Les primeres línies del fitxer indiquen que aquest s'ha d'executar cada vegada que es fa *push* a la branch `master`, o quan es fa una *pull request* a aquesta.
```yml
name: Python package

on:
  push:
    branches: [ master ]
  pull_request:
    branches: [ master ]
```

---

Tot seguit s'especifica la *job* `build`, que s'ha d'executar en un contenidor amb Ubuntu i es replica 4 vegades (per diferents versions de Python).
```yml
jobs:
  build:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: [3.6, 3.7, 3.8, 3.9]
```

---

Els passos de la *job* s'executen de forma seqüencial. En primer lloc s'executen una sèrie de passos previs que són necessaris per poder fer els tests:

- **Set up Python [versió]:** Prepara Python per poder ser utilitzat, cal indicar quina versió del llenguatge es prepara.
- **Install dependencies:** Utilitza `pip` per instal·lar els *packages* necessaris.
- **Lint with flake8:** Comprova que el codi no conté errors i atura l'execució de la *job* si en troba.

```yml
    steps:
      - uses: actions/checkout@v2
      
      - name: Set up Python ${{ matrix.python-version }}
        uses: actions/setup-python@v2
        with:
          python-version: ${{ matrix.python-version }}
          
      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install flake8 pytest
          pip install coverage
          if [ -f requirements.txt ]; then pip install -r requirements.txt; fi
          
      - name: Lint with flake8
        run: |
          flake8 . --count --select=E9,F63,F7,F82 --show-source --statistics
          flake8 . --count --exit-zero --max-complexity=10 --max-line-length=127 --statistics
```

---

Finalment, es poden executar els tests en si:

- **Test with pytest:** Executa els tests unitaris que es troben `test.py`.
- **Report test coverage:** Utilitza `coverage` per mostrar la cobertura dels tests executats.

```yml
      - name: Test with pytest
        run: |
          python test.py
          
      - name: Report test coverage
        run: |
          coverage run test.py
          coverage report
```
