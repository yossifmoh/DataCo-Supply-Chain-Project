<div align="center">

#  DataCo Smart Supply Chain Analytics

### End-to-End Data Pipeline — Medallion Architecture · Power BI · Machine Learning · Streamlit

**DEPI Final Project**

[![Streamlit App](https://img.shields.io/badge/Streamlit-Live%20Demo-FF4B4B?logo=streamlit&logoColor=white)](https://late-delivery-app-0.streamlit.app/)
[![Dataset](https://img.shields.io/badge/Dataset-Kaggle-20BEFF?logo=kaggle&logoColor=white)](https://www.kaggle.com/datasets/saicharankomati/dataco-supply-chain-dataset)
[![Python](https://img.shields.io/badge/Python-3.10%2B-3776AB?logo=python&logoColor=white)](#)
[![License](https://img.shields.io/badge/License-Educational-lightgrey)](#)

</div>

---

##  Overview

**DataCo Global** runs a complex, multi-market supply chain spanning Europe, LATAM, Pacific Asia, USCA, and Africa. Despite healthy sales volume, the business lacks visibility into **where profit leaks, why deliveries run late, and which products or regions truly drive value**.

This project builds a complete, reproducible analytics pipeline — from raw CSV to a live prediction app — to answer those questions and flag high-risk late deliveries *before* they happen.

> ⏱️ **Key finding:** **54.83%** of historical orders were delivered late — the single biggest driver behind this project.

---

##  Business Questions

| # | Theme | Question |
|---|---|---|
| 1 | **Sales & Profit** | How much are we selling, and how profitable is it? Is performance growing or declining? |
| 2 | **Customers & Products** | Which segments and products drive the most sales? What do customers buy most often? |
| 3 | **Shipping & Delivery** | Which shipping modes are used most — and which carry the highest late-delivery risk? |
| 4 | **Geography** | Which markets/regions generate the most sales and profit — and where are we underperforming? |
| 5 | **Prediction** | Can we predict, before an order ships, whether it's at risk of arriving late? |

---

##  Architecture — Medallion Pipeline

```
                 ┌───────────┐        ┌───────────┐        ┌───────────┐
   Raw CSV ───▶ │  BRONZE    │ ────▶ │  SILVER   │ ────▶ │   GOLD    │ ───▶ Power BI / ML / Streamlit
  (Kaggle)       │  Raw load │        │  Cleaned  │        │Star Schema│
                 └───────────┘        └───────────┘        └───────────┘
                180,519 rows          180,516 rows          1 Fact +
                53 columns            40 columns            5 Dimensions
```

###  Bronze Layer — `1789194802825_Bronze_Layers.ipynb`
- Loads `DataCoSupplyChainDataset.csv` exactly as received (encoding `latin1`)
- No transformations — a fully traceable, reproducible raw snapshot
- **Output:** `DataCoSupplyChain_Bronze.csv` — **180,519 rows · 53 columns**

###  Silver Layer — `Silver_Layer.ipynb`
- Drops PII & redundant columns (emails, passwords, zip codes, duplicate IDs, product images/descriptions)
- Fixes data types, handles negative values & outliers, checks distributions
- Standardizes categorical text (Market, Category, Shipping Mode, etc.)
- **Output:** `DataCoSupplyChain_Silver.csv` — **180,516 rows · 40 columns**

###  Gold Layer — `Gold_Layer_DataCo_Final_Version.ipynb`
- **Business process:** Order Item Sales & Delivery Performance
- **Grain:** one row = one order item (`Order Item Id` is the natural key — no surrogate needed)
- **Model:** `1 Fact + 5 Dimensions` (no `Dim_Date` — dates stay as plain Fact attributes)

```
Dim_Customer   ─┐
Dim_Product    ─┤
Dim_Shipping   ─┼──▶  Fact_Order_Items
Dim_Payment    ─┤
Dim_Geography  ─┘
```

| Table | Grain | Key | Main Attributes |
|---|---|---|---|
| `Fact_Order_Items` | 1 row = 1 order item | `Order Item Id` | Sales, profit, discount, quantity, delivery flags, order/shipping date |
| `Dim_Customer` | 1 row = 1 customer | `Customer Key` | Segment, city, state, country, lat/long |
| `Dim_Product` | 1 row = 1 product | `Product Key` | Name, price, category, department |
| `Dim_Shipping` | 1 row = 1 shipping mode | `Shipping Key` | Shipping Mode, scheduled days |
| `Dim_Payment` | 1 row = 1 payment type | `Payment Key` | Payment / order type |
| `Dim_Geography` | 1 row = 1 destination combo | `Geography Key` | Market, order city/state/country/region |

 Validated for fact-grain integrity, surrogate-key uniqueness, and zero orphaned foreign keys.

---

##  Exploratory Data Analysis — `EDA_DataCo_Supply.ipynb`

The Gold Fact and all five Dimensions are merged into one flat table and explored across **5 business-driven pages**, later rebuilt as Power BI report pages:

1. **Overview** — Total sales, profit, orders & multi-year trend
2. **Sales & Profit** — Top categories, departments, and products
3. **Customers & Products** — Segment value, market mix, best sellers
4. **Shipping & Delivery** — Shipping-mode usage & late-delivery risk
5. **Geography** — Sales & profit by market, country, and region

---

##  Power BI Report

An interactive, filterable 5-page dashboard built on the Gold star schema.

| Page | Highlights |
|---|---|
| **Overview** | $36.78M total sales · $3.97M profit · 10.78% margin · 66K orders |
| **Sales & Profit** | Segment/market filters · sales by department & top products |
| **Customers & Products** | 21K customers · 118 products · sales by segment & category |
| **Shipping & Delivery** | 54.83% late-delivery rate · shipping-mode breakdown |
| **Geography** | Sales/profit by market & country, with a world map view |

>  See `/PowerBI` for the `.pbix` file and dashboard screenshots.

---

##  Machine Learning — `AI_Model.ipynb`

**Goal:** predict `Late_delivery_risk` before an order ships.

- **Model:** `RandomForestClassifier` inside an `sklearn` `Pipeline`
- **Features:** 34 total — 20 numerical (sales, discount, shipping days, order-date parts) + 14 categorical (market, category, shipping mode, etc.), one-hot encoded via `ColumnTransformer`
- **Evaluation:** accuracy, classification report, confusion matrix, feature importance


---

##  Streamlit Deployment

The trained pipeline is served as an interactive web app — users enter order details and get an instant late-delivery risk prediction.

###  [**Try the Live App**](https://late-delivery-app-0.streamlit.app/)

```
https://late-delivery-app-0.streamlit.app/
```

---

##  Repository Structure

```
DataCo-Supply-Chain-Analytics/
│
├── notebooks/
│   ├── 1_Bronze_Layer.ipynb
│   ├── 2_Silver_Layer.ipynb
│   ├── 3_Gold_Layer.ipynb
│   ├── 4_EDA_DataCo_Supply.ipynb
│   └── 5_AI_Model.ipynb
│
├── data/
│   ├── bronze/DataCoSupplyChain_Bronze.csv
│   ├── silver/DataCoSupplyChain_Silver.csv
│   └── gold/
│       ├── Fact_Order_Items.csv
│       ├── Dim_Customer.csv
│       ├── Dim_Product.csv
│       ├── Dim_Shipping.csv
│       ├── Dim_Payment.csv
│       └── Dim_Geography.csv
│
├── powerbi/
│   └── DataCo_Dashboard.pbix
│
├── streamlit_app/
│   ├── app.py
│   └── model.pkl
│
├── presentation/
│   └── DataCo_Final_Presentation.pptx
│
└── README.md
```

---

##  Tech Stack

`Python` · `Pandas` / `NumPy` · `Scikit-learn` · `Matplotlib` / `Seaborn` · `Power BI` · `Streamlit` · `Google Colab`

---

##  Dataset

**DataCo Smart Supply Chain Dataset** — sourced from Kaggle:
 https://www.kaggle.com/datasets/saicharankomati/dataco-supply-chain-dataset

---

##  Team

| Name | Role |
|---|---|
| Yahya Abdel-Nasser Abdel-Naim | Team Member |
| Yossif Mohamed Abbas | Team Member |
| Sondos Mohamed Abdeen Tawfiq | Team Member |
| Romisaa Mohamed Mostafa | Team Member |

---

<div align="center">

Built as part of the **National Telecommunication Institute(NTI)**

 *If you found this project useful, consider giving it a star!*

</div>
