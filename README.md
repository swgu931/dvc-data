# how to use DVC (Data Version Control)
- ref: http://dvc.org/doc

## install
```
pip3 install dvc
pip3 install dvc[ssh]  # for local private as storage
pip3 install 'dvc[gdrive]'  # for google drive as storage
```

-----------------------------------
1) Data Versioning
- ref: https://dvc.org/doc/start/data-versioning

```
git clone https://github.com/data-registry
```
```
cd data-registry
git init
dvc init
mkdir data
dvc get https://github.com/iterative/dataset-registry get-started/data.xml -o data/data.xml
ls -lh data
```
```
dvc add data/data.xml
git add data/.gitignore data/data.xml.dvc
git commit -m "Add raw data"
```
-- https://drive.google.com/drive/folders/1nt2EUfiDMJeLTneZAFy373zuVEU20y0J
dvc remote add -d storage gdrive://1nt2EUfiDMJeLTneZAFy373zuVEU20y0J
git commit .dvc/config -m "Configure remote storage"
dvc push
--Authentication with link
git push
```
```
cat data/.gitignore
rm -f data/data.xml
rm -rf .dvc/cache
```
```
ls ./data
 data.xml.dvc
dvc pull
ls data/

- data revert
```
cp data/data.xml /tmp/data.xml
cat /tmp/data.xml >> data/data.xml
ls -lh data
```
```
dvc add data/data.xml
git add data/data.xml.dvc
git commit -m "Dataset updates"
git push
dvc push
git log --oneline
```
```
# two data.xml version exist
# checkout previous version
git checkout HEAD^1 data/data.xml.dvc
dvc checkout
ls -lh data

git commit data/data.xml.dvc -m "Revert dataset updates"
```
------------------------------------------------------------

- ref: git remote add origin git@github.com:elleobrien/dvc_basics.git

------------------------------------------------------------
2) Data Access
- ref: https://dvc.org/doc/start/data-access
```
dvc list https://github.com/iterative/dataset-registry get-started
```
```
dvc get https://github.com/iterative/dataset-registry use-cases/cats-dogs
```
```
# dvc import == dvc get + dvc add
dvc import https://github.com/iterative/dataset-registry \
             get-started/data.xml -o data/data.xml
dvc update data/data.xml.dvc
```

- vi py_api.py
```
import dvc.api

with dvc.api.open(
  'get-started/data.xml',
        repo='https://github.com/iterative/dataset-registry'
        ) as fd:
  # fd.read()
```

------------------------------------------------------------
3) Data Pipelines
- ref: https://dvc.org/doc/start/data-pipelines
```
wget https://code.dvc.org/get-started/code.zip
unzip code.zip
rm -f code.zip
tree
```
```
pip install -r src/requirements.txt
```
```
dvc run -n prepare \
          -p prepare.seed,prepare.split \
          -d src/prepare.py -d data/data.xml \
          -o data/prepared \
          python3 src/prepare.py data/data.xml
tree
git add dvc.yaml data/.gitignore dvc.lock
```
```
dvc run -n featurize \
          -p featurize.max_features,featurize.ngrams \
          -d src/featurization.py -d data/prepared \
          -o data/features \
          python src/featurization.py data/prepared data/features
tree
git add data/.gitignore dvc.yaml dvc.lock
```
```
dvc run -n train \
          -p train.seed,train.n_estimators \
          -d src/train.py -d data/features \
          -o model.pkl \
          python3 src/train.py data/features model.pkl
tree
git add dvc.lock .gitignore dvc.yaml
```

- vi params.yaml 
 : Change n_estimators to 100
```
dvc repro
git add dvc.lock
dvc dag
```
==============================================
4) Experiments
- ref: https://dvc.org/doc/start/experiments
```
dvc run -n evaluate \
          -d src/evaluate.py -d model.pkl -d data/features \
          -M scores.json \
          --plots-no-cache prc.json \
          python3 src/evaluate.py model.pkl \
                 data/features scores.json prc.json
git add dvc.lock dvc.yaml

git add scores.json prc.json
git commit -a -m "Create evaluation stage"
```

## change featurize as below and dvc repro for tunning more
- max_features: 500
- ngrams: 2
```
dvc repro
git add dvc.lock

dvc params diff
dvc metrics diff
dvc plots diff -x recall -y precision
dvc dag
```
