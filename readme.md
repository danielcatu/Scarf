# SCARF — Serverless Cloud Auto Resource Framework

A framework that automatically adjusts CPU and memory limits for serverless functions running on Knative/Kubernetes, using LSTM-based CPU usage forecasting and Bayesian optimization.

## Problem

Serverless platforms require manual resource configuration (CPU limits, memory). Static allocation leads to over-provisioning (wasted cost) or under-provisioning (latency spikes under load). SCARF predicts future resource demand and proactively reconfigures Knative services before load changes occurs.

## Architecture

```
Prometheus  →  Historical CPU metrics
                        ↓
               LSTM / Bayesian Opt
                        ↓
              Resource predictions
                        ↓
         resource_manage.py  →  kubectl / kn  →  Knative services
```

1. **Serverless functions** run on Knative (Kubernetes) using Docker with Sysbox runtime.
2. **Prometheus** collects CPU usage and invocation metrics per pod.
3. **LSTM model** (Keras/TensorFlow) is trained on historical CPU time series to forecast future usage.
4. **Bayesian optimization** is evaluated as an alternative optimization strategy.
5. **Resource manager** modifies Knative service YAML files programmatically and applies them via `kn service apply`, adjusting CPU/memory limits and requests per function.

## Benchmark

Uses the [Black-Scholes](https://parsec.cs.princeton.edu/) workload from the PARSEC benchmark suite as the serverless function under test. Invocation traces sourced from the [Azure Functions dataset](https://github.com/Azure/AzurePublicDataset).

## Tech stack

| Layer | Tools |
|---|---|
| Serverless runtime | Knative, Kubernetes, Docker + Sysbox |
| Monitoring | Prometheus |
| ML | TensorFlow, Keras, scikit-learn |
| Optimization | bayesian-optimization |
| Data | pandas, NumPy |
| Notebooks | Jupyter |

## Project structure

```
├── Azure/                  # Azure invocation traces and exploration notebook
├── dataset/                # Black-Scholes execution metrics (CSV)
├── result/                 # Experiment results per run
├── experiments/
│   ├── newoptMethod.ipynb  # LSTM-based resource prediction
│   ├── graph.ipynb         # Results visualization
│   ├── dataset_explore.ipynb
│   ├── bayes_op.py         # Bayesian optimization method
│   ├── resource_manage.py  # Knative resource configurator
│   ├── prometheus.py       # Metrics collection
│   └── serverless-functions/  # Knative YAML definitions (Parsec/Black-Scholes)
├── Dockerfile              # Sysbox-compatible image for Knative
├── requirements.txt
└── deploy.md
```

## Setup

### Prerequisites

- Docker with [Sysbox](https://github.com/nestybox/sysbox) runtime
- Kubernetes cluster with Knative installed
- Prometheus instance scraping function pods

### Install Sysbox

```bash
wget https://downloads.nestybox.com/sysbox/releases/v0.6.1/sysbox-ce_0.6.1-0.linux_amd64.deb
sudo apt-get install jq
sudo apt-get install ./sysbox-ce_0.6.1-0.linux_amd64.deb
```

### Build and run

```bash
# Build image
docker build -t scarf .

# Run with Sysbox (enables Knative inside container)
docker run -p 80:80 --runtime=sysbox-runc --rm -it scarf /bin/bash
```

### Python dependencies

```bash
pip install -r requirements.txt
```

## Usage

1. Deploy serverless functions to Knative using the YAMLs in `test/serverless-functions/`.
2. Start Prometheus scraping and collect CPU metrics.
3. Run `test/newoptMethod.ipynb` to train the LSTM and generate resource predictions.
4. `resource_manage.py` applies the predicted resource configuration to the live Knative services.

## Context

Master's thesis — Universidad del Norte, Barranquilla, Colombia.
Master's in Systems and Computer Engineering, 2024.
