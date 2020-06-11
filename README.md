# Spectral Temporal Graph Neural Network for Multivariate Time-series Forecasting

This repository is the official implementation of Spectral Temporal Graph Neural Network for
Multivariate Time-series Forecasting.

## Requirements

Recommended version of OS & Python:

* **OS**: Ubuntu 18.04.2 LTS
* **Python**: python3.7 ([instructions to install python3.7](https://linuxize.com/post/how-to-install-python-3-7-on-ubuntu-18-04/)).

To install python dependencies, virtualenv is recommended, `sudo apt install python3.7-venv` to install virtualenv for python3.7. All the python dependencies are verified for `pip==20.1.1` and `setuptools==41.2.0`. Run the following commands to create a venv and install python dependencies:

```setup
python3.7 -m venv venv
source venv/bin/activate
pip install --upgrade pip
pip install -r requirements.txt
```

## Datasets

[PEMS03](http://pems.dot.ca.gov/?dnode=Clearinghouse&type=station_5min&district_id=3&submit=Submit),
[PEMS04](http://pems.dot.ca.gov/?dnode=Clearinghouse&type=station_5min&district_id=4&submit=Submit),
[PEMS07](http://pems.dot.ca.gov/?dnode=Clearinghouse&type=station_5min&district_id=7&submit=Submit),
[PEMS08](http://pems.dot.ca.gov/?dnode=Clearinghouse&type=station_5min&district_id=8&submit=Submit),
[METR-LA](https://github.com/liyaguang/DCRNN),
[PEMS-BAY](https://github.com/liyaguang/DCRNN),
[Solar](https://www.nrel.gov/grid/solar-power-data.html),
[Electricity](https://archive.ics.uci.edu/ml/datasets/ElectricityLoadDiagrams20112014),
[ECG5000](http://www.timeseriesclassification.com/description.php?Dataset=ECG5000),
[COVID-19](https://github.com/CSSEGISandData/COVID-19/tree/master)

We can get the raw data through the links above. We evaluate the performance of traffic flow forecasting on PEMS03, PEMS07, PEMS08 and traffic speed forecasting on PEMS04, PEMS-BAY and METR-LA. So we use the traffic flow table of PEMS03, PEMS07, PEMS08 and the traffic speed table of PEMS04, PEMS-BAY and METR-LA as our datasets. We download the solar power data of Alabama (Eastern States) and merge the 5-minute csv files (totally 137 time series) as our Solar dataset. We delete the header and index of Electricity file downloaded from the link above as our Electricity dataset. For COVID-19 dataset, the raw data is under the folder `csse_covid_19_data/csse_covid_19_time_series/` of the above github link. We use `time_series_covid19_confirmed_global.csv` to calculate the daily number of newly confirmed infected people from 1/22/2020 to 5/10/2020. The 25 countries we take into consideration are 'US','Canada','Mexico','Russia','UK','Italy','Germany','France','Belarus ','Brazil','Peru','Ecuador','Chile','India','Turkey','Saudi Arabia','Pakistan','Iran','Singapore','Qatar','Bangladesh','Arab','China','Japan','Korea'.

The input csv file should contain **no header** and its **shape should be `T*N`**, where `T` denotes total number of timestamps, `N` denotes number of nodes.

Since complex data cleansing is needed on the above datasets provided in the urls before fed into the StemGNN model, we provide a cleaned version of ECG5000 ([./dataset/ECG_data.csv](./dataset/ECG_data.csv)) for reproduction convenience. The ECG_data.csv is in shape of `5000*140`, where `5000` denotes number of timestamps and `140` denotes total number of nodes. Run command `python main.py` to trigger training and evaluation on ECG_data.csv.

## Training and Evaluation

The training procedure and evaluation procedure are all included in the `main.py`. To train and evaluate on some dataset, run the following command:

```train & evaluate
python main.py --train True --evaluate True --dataset <path to csv file> --output_dir <path to output directory> --n_route <number of nodes> --n_his <length of sliding window> --n_pred <predict horizon> --scalar z_score --train_length 6 --validate_length 2 --test_length 2
```

The detailed descriptions about the parameters are as following:
| Parameter name | Description of parameter |
| --- | --- |
| train | whether to enable training, default True |
| evaluate | whether to enable evaluation, default True |
| dataset | path to the input csv file |
| output_dir | output directory, will store models and tensorboard information in this directory, and evaluation restores model from this directory |
| scalar | method to normalize, 'z_score' or 'min_max' |
| train_length | length of training data, default 6 |
| validate_length | length of validation data, default 2 |
| test_length | length of testing data, default 2 |
| n_route | number of time series, i.e. number of nodes |
| n_his | length of sliding window, default 12 |
| n_pred | predict horizon, default 3 |

**Table 1** Configurations for all datasets
| Dataset | train | evaluate | n_route | n_his | n_pred | scalar |
| -----   | ---- | ---- |---- |---- |---- | --- |
| PEMS03 | True | True |  358 | 12 | 12 | z_score |
| PEMS04 | True | True |  307 | 12 | 12 | z_score |
| PEMS08 | True | True |  170 | 12 | 12 | z_score |
| METR-LA | True | True | 207 | 12 | 3 | z_score |
| PEMS-BAY | True | True |  325 | 12 | 3 | z_score |
| PEMS07 | True | True | 228 | 12 | 3 | z_score |
| Solar | True | True |  137 | 12 | 3 | z_score |
| Electricity | True | True | 321 | 12 | 3 | z_score |
| ECG5000| True | True | 140 | 12 | 3 | min_max |
| COVID-19| True | True | 25 | 28 | 28 | z_score |

## Results

Our model achieves the following performance on the 9 datasets included in the code repo:

**Table 2** (predict horizon: 3 steps)
| Dataset | MAE  | RMSE | MAPE(%) |
| -----   | ---- | ---- | ---- |
| METR-LA | 2.56 | 5.06 | 6.46 |
| PEMS-BAY | 1.23 | 2.48 | 2.63 |
| PEMS07 | 2.14 | 4.01 | 5.01 |
| Solar | 1.52 | 1.53 | 1.42 |
| Electricity | 0.04 | 0.06 | 14.77 |
| ECG5000 | 0.05 | 0.07 | 10.58 |

**Table 3** (predict horizon: 12 steps)
| Dataset | MAE  | RMSE | MAPE |
| -----   | ---- | ---- | ---- |
| PEMS03 | 14.32 | 21.64 | 16.24 |
| PEMS04 | 20.24 | 32.15 | 10.03 |
| PEMS08 | 15.83 | 24.93 | 9.26 |

**Table 4** (predict horizon: 28 steps)
| Dataset | MAE  | RMSE | MAPE |
| -----   | ---- | ---- | ---- |
| COVID-19 | 662.24 | 1023.19| 19.3|

