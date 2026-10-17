# Dataset

The experiment uses the synthetic PaySim mobile-money log. The notebook reads one file:

`data/PS_20174392719_1491204439457_log.csv`

That path is set by `CSV_PATH` in `XGBoost_Fraud_Detection.ipynb`. Do not commit the CSV.

## Source

- Simulation: Edgar Alonso Lopez-Rojas, Ahmad Elmir, and Stefan Axelsson, “PaySim: A financial mobile money simulator for fraud detection,” 28th European Modeling and Simulation Symposium, 2016.
- Distribution used here: [PaySim1 on Kaggle](https://www.kaggle.com/datasets/ealaxi/paysim1).

PaySim generates a synthetic mobile-money network. It is not a dump of customer transactions from a live provider.

## Download and placement

1. Open [the Kaggle dataset](https://www.kaggle.com/datasets/ealaxi/paysim1) and accept its terms.
2. Download `PS_20174392719_1491204439457_log.csv`. The file is about 494 MB.
3. Place it at `data/PS_20174392719_1491204439457_log.csv`, beside this README.
4. Leave the file untracked. `.gitignore` excludes `data/*.csv`.

The executed notebook loaded 6,362,620 data rows and 11 columns: `step`, `type`, `amount`, `nameOrig`, `oldbalanceOrg`, `newbalanceOrig`, `nameDest`, `oldbalanceDest`, `newbalanceDest`, `isFraud`, and `isFlaggedFraud`. Account identifiers are read only as columns to skip. They are not model inputs.

A local copy is already in this folder on the machine used to prepare the repository. Clone recipients still need to download it themselves.
