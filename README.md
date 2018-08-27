# Blockchain-eKYC (Hyperledger Sawtooth)
### Open source eKYC blockchain built on Hyperledger Sawtooth

![Dashboard](http://www.primechaintech.com/img/sawtooth/dashboard.png)

[1. Introduction](#1-introduction)

[2. Uploading records](#2-uploading-records)

[3. Transaction Processor and State](#3-transaction-processor-and-state)

[4. Client Application](#4-client-application)

[5. Installation and setup](#5-installation-and-setup)

[6. Third party software and components](#6-third-party-software-and-components)

[7. License](#7-license)

## 1. Introduction
Financial and capital markets use the KYC (Know Your Customer) system to identify "bad" customers and minimise money laundering, tax evasion and terrorism financing. Efforts to prevent money laundering and the financing of terrorism are costing the financial sector billions of dollars. Banks are also exposed to huge penalties for failure to follow KYC guidelines. Costs aside, KYC can delay transactions and lead to duplication of effort between banks.

Blockchain-eKYC (Hyperledger Sawtooth) is a permissioned Hyperledger Sawtooth blockchain for sharing corporate KYC records amongst banks and other financial institutions. 

The records are stored in the blockchain in an encrypted form and can only be viewed by entities that have been "whitelisted" by the issuer entity. This ensures data privacy and confidentiality while at the same time ensuring that records are shared only between entities that trust each other.

Blockchain-eKYC (Hyperledger Sawtooth) is maintained by Rahul Tiwari, Blockchain Developer, [Primechain Technologies Pvt. Ltd.](http://www.primechaintech.com/)

***Primary benefits***

1. Removes duplication of effort, automates processes and reduces compliance errors.

2. Enables the distribution of encrypted updates to client information in real time.

3. Provides the historical record of all compliance activities undertaken for each customer.

4. Provides the historical record of all documents pertaining to each customer.

5. Records can be used as evidence to prove to regulators that the bank has complied with all relevant regulations.

6. Enables identification of entities attempting to create fraudulent histories.

7. Enables data and records to be analyzed to spot criminal activities.


## 2. Uploading records

![Uploading records](http://www.primechaintech.com/img/sawtooth/upload2.png)

Records can be uploaded in any format (doc, pdf, jpg etc.) upto a maximum of 10 MB per record. These records are automatically encrypted using AES symmetric encryption algorithm and the decryption keys are automatically stored in the exclusive web application of the of the uploading entity. 

***When a new record is uploaded to the blockchain, the following information must be provided:***

1. ***Corporate Identity Number (CIN)*** of the entity to which this document relates - this information is stored in the blockchain in plain text / un-encrypted form and cannot be changed.

2. ***Document category*** - this information is stored in the blockchain in plain text / un-encrypted form and cannot be changed.

3. ***Document type*** - this information is stored in the blockchain in plain text / un-encrypted form and cannot be changed.

4. ***A brief description of the document*** - This information is stored in the blockchain in plain text / un-encrypted form and cannot be changed.

5. ***The document*** - this can be in pdf, word, excel, image or other format and is stored in the blockchain in AES encrypted form and cannot be changed. The decryption key is stored in the relevant bank's dedicated database and does NOT go into the blockchain. 

***When the above information is provided, this is what happens:***
1. Hash of the uploaded file is calculated.
2. The file is digitally signed using the private key of the uploader bank.
3. The file is encrypted using AES symmetric encryption.
4. The encrypted data is converted into hexadecimal.
5. The non-encrypted data is converted into hexadecimal.
6. Hexadecimal content is uploaded to the blockchain.

![Code](http://www.primechaintech.com/img/sawtooth/encrypt.png)

***Sample output:***
```
    {
      file_hash: 84a9ceb1ee3a8b0dc509dded516483d1c4d976c13260ffcedf508cfc32b52fbe
      file_txid: 2e770002051216052b3fdb94bf78d43a8420878063f9c3411b223b38a60da81d
      data_txid: 85fc7ff1320dd43d28d459520fe5b06ebe7ad89346a819b31a5a61b01e7aac74
      signature: IBJNCjmclS2d3jd/jfepfJHFeevLdfYiN22V0T2VuetiBDMH05vziUWhUUH/tgn5HXdpSXjMFISOqFl7JPU8Tt8=
      secrect_key: ZOwWyWHiOvLGgEr4sTssiir6qUX0g3u0
      initialisation_vector: FAaZB6MuHIuX
    }
```
![Search](http://www.primechaintech.com/img/sawtooth/search.png)

![Search](http://www.primechaintech.com/img/sawtooth/search2.png)


## 3. Transaction Processor and State

This section uses the following terminology: 
* [Transaction Processor](https://intelledger.github.io/architecture/transactions_and_batches.html) - this is the business logic / smart contracts layer.
* [Validator Process](https://sawtooth.hyperledger.org/docs/core/releases/latest/architecture/global_state.html) - this is the Global State Store layer. 
* Client Application (User)	- this implies a user of the solution; the user’s public key executes the transactions.

The Transaction Processor of the eKYC application is written in Java. It contains all the business logic of the application. Hyperledger Sawtooth stores data within a Merkle Tree. Data is stored in leaf nodes and each node is accessed using an addressing scheme that is composed of 35 bytes, represented as 70 hex characters. 

Using the Corporate Identity Number or CIN, provided by user while uploading, a 70 characters (35 bytes) address is created for uploading a record to the blockchain. To understand the address creation and namespace design process, [see this](https://sawtooth.hyperledger.org/docs/core/releases/1.0/app_developers_guide/address_and_namespace.html).

Below is the address creation logic in the application:

![address creation logic](http://www.primechaintech.com/img/sawtooth/address_creation.png)

Note:

* `uniqueValue` is the type of data (can be any value)
* `kycAddress` is the CIN of the uploaded document.

The User can upload multiple files using the same CIN. However, state will return only the latest uploaded document. To get all the uploaded documents on the same address, business logic is written in Transaction Processor.  

![](http://www.primechaintech.com/img/sawtooth/txn_logic.png)

The `else {` part will do the uploading of multiple documents on the same address and fetching every uploaded document from the State.

## 4. Client Application

The client application uses REST API endpoints to upload (POST), get (GET) documents on the sawtooth blockchain platform. It is written in Nodejs. In case of uploading, few steps to be considered:
* Creating and encoding transactions having header, header signature, payload (Transaction payloads are composed of binary-encoded data that is opaque to the validator.)

![](http://www.primechaintech.com/img/sawtooth/transaction.png)

* Creating BatchHeader, Batch, and encoding Batches

![](http://www.primechaintech.com/img/sawtooth/batches.png)

* Submitting batches to the Validator

![](http://www.primechaintech.com/img/sawtooth/post_validator.png)

In case of getting uploaded data from blockchain, following steps needs to be considered:

1. Creating the same address from the CIN given by the User, using GET method to fetch the data stored on the particular address. As given in following code snippet, `updatedAddress` created by getting user input either from User (search using CIN in the network) or from private database of the user (Records uploaded by the user). Similarly, `splitStringArray` splits the data returned from a particular address because of the transaction logic written in the Transaction Processor to upload multiple documents on the same address while updating state with the list of all the uploaded data (not only the current payload).

![](http://www.primechaintech.com/img/sawtooth/get_addr.png)

2. After this client side logic is written to convert the `splitStringArray` by decoding it to the required format and giving User an option to download the same in the form of a file.

## 5. Installation and setup
Please refer to the guide here:
https://github.com/Primechain/blockchain-ekyc-sawtooth/blob/master/setup.MD


## 6. Third party software and components
Third party software and components: bcryptjs, body-parser, connect-flash, cookie-parser, express, express-fileupload, express-handlebars, express-session, express-validator, mongodb, mongoose, multichain, passport, passport-local, sendgrid/mail.

## 7. License
Blockchain-eKYC (Hyperledger Sawtooth) is available under Apache License 2.0. This license does not extend to third party software and components.
