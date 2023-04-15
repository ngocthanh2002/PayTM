# 🏦 PayTM

<h1 style="font-size:60px;">📚 Table of Contents</h1>
    <p>Data</p>
    <p>Data Exploration</p> 
    
 <hr>
💡 Data
Paytm is an Indian multinational financial technology company. It specializes in digital payment system, e-commerce and financial services. Paytm wallet is a secure and RBI (Reserve Bank of India)-approved digital/mobile wallet that provides a myriad of financial features to fulfill every consumer’s payment needs. Paytm wallet can be topped up through UPI (Unified Payments Interface), internet banking, or credit/debit cards. Users can also transfer money from a Paytm wallet to recipient’s bank account or their own Paytm wallet

<picture>
  <img  src="https://user-images.githubusercontent.com/129776645/232193405-ba0ec96a-dcb6-480f-93a7-34ca2d335031.png">
</picture>


🧾A small database of payment transactions from 2019 to 2020 of Paytm Wallet. The database includes 6 tables: 
* fact_transaction: Store information of all types of transactions: Payments, Top-up, Transfers, Withdrawals
* dim_scenario: Detailed description of transaction types
* dim_payment_channel: Detailed description of payment methods
* dim_platform: Detailed description of payment devices
* dim_status: Detailed description of the results of the transaction

<hr>

🔎 Data Exploration
* Retrieve an overview report of payment types
* Retrieve an overview report of customer’s payment behaviors
* Time Series Analysis
* Cohorts Derived from the Time Series
* Building an RFM model

