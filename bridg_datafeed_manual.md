# Implementation Data Specification | Bridg CDP

## Introduction

Welcome to **Bridg!** We’re excited to work with you and your team so you can start leveraging Bridg’s unique tools and insights to better target and reach your customers as quickly and painlessly as possible. This CDP Implementation Data Specification will help get us started.

This guide consolidates the agreed-upon methodology and mechanism surrounding data transfer.

## List of Contents for Quick Reference

- [Data Format](#data-format)
- [Daily Data Transfer](#daily-data-transfer-&-transfer)
- [Catchup Data](#catchup-data)
- [Historical Data](#historical-data)
- [Data QA and Validation](#data-qa-and-validation)
- [Files and naming convention](#files-and-naming-convention)
  - [Check Summary](#check-summary)
    - [Data Specification for Check Summary](#data-specification-for-check-summary)
  - [Tender](#Tender)
    - [Data Specification for Tender](#data-specification-for-tender)
  - [Line Item](#line-item)
    - [Data Specification for Line Item](#data-specification-for-line-item)
  - [Discount](#discount)
    - [Data Specification for Discount](#data-specification-for-discount)
  - [Comp](#comp)
    - [Data Specification for Comp](#data-specification-for-comp)
  - [Tender Type Dimension](#tender-type-dimension)
    - [Data Specification for Tender Type Dimension](#data-specification-for-tender-type-dimension)
  - [Tender Type Detail Dimension](#tender-type-detail-dimension)
    - [Data Specification for Tender Type Detail Dimension](#data-specification-for-tender-type-detail-dimension)
  - [Order Source Dimension](#order-source-dimension)
    - [Data Specification for Order Source Dimension](#data-specification-for-order-source-dimension)
  - [Discount Dimension](#discount-dimension)
    - [Data Specification for Discount Dimension](#data-specification-for-discount-dimension)
  - [Comp Dimension](#comp-dimension)
    - [Data Specification for Comp Dimension](#data-specification-for-comp-dimension)
  - [Location](#location)
    - [Data Specification for Location](#data-specification-for-location)
  - [Customer](#customer)
    - [Data Specification for Customer](#data-specification-for-customer)
- [Customizations and Change Requests](#customizations-and-change-requests)

## Data Format

We are expecting the incoming files to be in standard utf-8 encoded CSV format. In following sections we have provided information about the required data fields and their format that Bridg typically ingests from a client’s data warehouse or a third party system. We’ll use this document to:

- Gain a better understanding of what type of data you should be sending to Bridg.
- Identify data gaps or potential augmentation opportunities in existing data sources.
- Develop a schema to transfer data to Bridg.

At a high level, the daily data should include the following files:

- Transaction Summary
- Tender
- Line Item
- Discount
- Customer
- Dimension files
  - Order Source dimension
  - Tender Type dimension
  - Product dimension
  - Comp dimension
  - Category dimension

**Please confirm which fields you have available and will be sending. If a field won’t be available or sent, Bridg needs to agree and confirm that we can proceed without it.**

## Daily Data Transfer & Cadence

All files must be in a single level atomic zip file named for the date *(YYYY-MM-DD)* and transferred at the same time daily either through a Secure File Transfer Protocol (SFTP) or AWS S3 Sync. Bridg will provide the SFTP host. You will provide a public key for Bridg to attach to the SFTP. If you’re using AWS S3 Sync, the files have to be pushed to our AWS infrastructure.

## Catchup Data

If transactional data comes in after the date of transfer, those rows should be included in the following day’s files (or whatever day they become available).

## Historical Data

As part of the original data feed, we’ll be ingesting two or more years of historical data. The historical data should come through in an identical format as the daily data both in terms of column formatting as well as the daily zip file _(YYYY-MM-DD)_.

## Data QA and Validation

While you build out any necessary extract, transform, load processes (ETLs) to provide Bridg the data export, we will configure your account to prepare for data ingestion. To facilitate the configuration, please provide between one month’s worth of the files listed above, so we can run your data through a high-level validation process.

### In addition to general QA, we need to know:

- What is the unique identifier for a transaction?
- How frequently are check IDs recycled (daily, weekly, monthly, etc.)?
- Are check IDs unique across all stores?
- Will there be duplicates in the data, and if so, why?
- How many transactions per store per day?
- How late can data come in for a business date?
- What are all possible tender types and their IDs?
- What are all possible tender types details and their IDs?
- What are all possible order sources and their IDs?

## Overview of File Structure

### Summary of Files:

Our spec includes (5) fact files and (5) dimension files.

*Fact Files:*

* **Transaction Summary** - The ‘Transaction Summary’ file contains a top-level summary of all transactions in each order, where each order is represented as a single row of data.
* **Tender** - The ‘Tender’ file contains each individual split charge within a given order.  If a check is split three ways, there should be one row of data in the ‘Transaction Summary’ file and three Tender rows.
* **Line Item** - The ‘Line Item’ file contains all items ordered on a given check.
* **Discount** - The ‘Discount’ file contains all discounts and promotions applied to an individual check, where each discount applied to a check has a separate row of data.
* **Customer** - The ‘Customer’ file contains all customers and customer data, including any unique IDs used to identify them.

*Dimension Files:*

* **Order Source** - The ‘Order Source’ dimension file contains more detailed information for each Order Source from the ‘Transaction Summary’ file, where each row of data corresponds to a unique Order Source ID.
* **Tender Type** - The ‘Tender Type’ dimension file contains more detailed information for each Item from the 'Tender' file, where each row of data corresponds to a unique Tender Type ID.
* **Product** - The ‘Product’ dimension file contains more detailed information for each Item contained in the 'Line Item' file, where each row of data corresponds to a unique Product ID.
* **Discount Detail** - The ‘Discount Detail’ dimension file contains more detailed information for each Discount Type included in the ‘Discount’ fact file (including promotions and comps), where each row of data corresponds to a unique Discount ID.
* **Location** - The ‘Location’ dimension file contains all open and closed locations and data for each.

### Customizations and Change Requests:

If the data schema doesn’t match the agreed upon format below, or if the data transfer comes through differently, both parties will need to sign a project change request that will include any modification to our standard specifications, timeline updates, and potential implementation fees.

## File Details: Data Formats & Naming Conventions

This section outlines each of the Fact Files and Dimension Files we expect, including details around how each field is used, acceptable data formats, and naming conventions.  Please review the tables included for each file type for specifics about the headers and content of the data we expect to receive.

**Note:**
For each file, The YYYY-MM-DD of the atomic zip file must match the YYYY-MM-DD of the individual files.

### 1. Transaction Summary File

| File Name: | transaction_summary_YYYY-MM-DD.csv |
| :- | :- |
| **Description:** | A top level summary of all transactions in each order,<br />where each order is represented as a single row of data. |
| **Considerations:** | Contains lookups to the Order Source dimension file |

*Data Specification for Check Summary:*

| # | Column / System Field | Required | Description | Data Format / Sample Data | References |
| :-: | - | :-: | - | - | - |
| 1 | Date<br />date | Y | The date each transaction was placed | Provide in ISO 8601 format (YYYY-MM-DD):<br />2017-12-14 |   |
| 2 | Location ID<br />location_id | Y | The unique identifier used in our system to identify each location | Provide as alphanumeric string:<br />5812 | Cross-referenced in multiple files; used to map transactions and line items to a specific location |
| 3 | Check ID<br />check_id | Y | The single unique identifier used in our system to identify each check | Provide as alphanumeric string:<br />10001 | Cross-referenced in multiple files; used to map transactions to a specific store on a given day<br />Does not have to be unique across all stores or all time. |
| 4 | Date & Time (Local)<br />datetime_local | Y | Date and time of the transaction in the location's local timezone | ISO 8601 format (yyyy-mm-ddThh:mm:ss):<br />2017-12-14T21:05:00 | There is a separate (optional) field to store Date & Time in UTC format (see below) |
| 5 | Tender Amount<br />tender |   |   |   |   |
| 10001 |   |   |   |   |   |
| Check ID | Yes | 10001 | Alphanumeric string that uniquely identifies a transaction within a store on a given day. |   |   |
| Date Time (UTC) | No | 2017-12-14T21:05:00 | Date and time of the transaction in UTC(ISO 8601 format yyyy-mm-ddThh:mm:ss) |   |   |
| Date_Time (Local) | Yes | 2017-12-14T21:05:00 | Date and time of the transaction in store's local timezone (ISO 8601 format yyyy-mm-ddThh:mm:ss) |   |   |
| Tender Amount | Yes | 24.19 | Post discount and post tax tendered amount |   |   |
| Tax Amount | Yes | 4.19 | Amount of tax charged for the transaction |   |   |
| Discount | Yes | 2.5 | Amount of discount applied for the specific transaction. Discount should be the sum of all comps, discounts, and promos applied to check |   |   |
| Pre-tax | No | 25.69 | Pre-discount and pre-tax amount |   |   |
| Party Size | No | 24.19 | Number of people dining |   |   |
| Order Source | Yes | 2 | Integer representing where the transaction was initiated (i.e., 1, 2, 3, 64, 89) |   |   |
| Currency | No | USD | Default to USD |   |   |
| --- | --- | --- | --- | --- | --- |
|   |   |   |   |   |   |

### Tender

**File name:** _YYYY-MM-DD_tender.csv_

**Description:** The tender file is each individual charge. If a check is split three ways, there should be one check summary and three tender rows.

- The tender file has both_Tender Type_ and_Detailed Tender Type_. Tender Type is an integer that we can use with the Tender Type dimension file. For example,
  - 1 => Cash
  - 2 => Credit Card
  - 3 => Gift Card
- We also include an optional Detailed Tender Type, which allows us to do further segmentation on the specific tender type. For example,
  - 40 => Visa
  - 41 => Amex
- If you’re sending tokens as opposed to credit card last four, they have to unique across all customers and all stores.

#### Data Specification for Tender

| Column | Required | Example | Description |
| - | - | - | - |
| Date | Yes | 2020-01-01 | Date of the transaction (ISO 8601 format yyyy-mm-dd). |
| Store ID | Yes | 5812 | Alphanumeric string that uniquely identifies a store. |
| Check ID | Yes | 10001 | Alphanumeric string that uniquely identifies a transaction within a store on a given day. |
| Date Time (UTC) | No | 2017-12-14T21:05:00 | Date and time of the transaction in UTC(ISO 8601 format yyyy-mm-ddThh:mm:ss) |
| Date_Time (Local) | Yes | 2017-12-14T21:05:00 | Date and time of the transaction in store's local timezone (ISO 8601 format yyyy-mm-ddThh:mm:ss) |
| Tender Type | Yes | 3 | Integer that identifies the tender type used for the transaction. E.g. 1 (Cash), 2 (Credit Card), 3 (Gift Card) |
| Detailed Tender Type | No |   | See Tender Type Detail Dimension dimension for more details |
| Cardholder Name | Yes | JACKSON ROBERT | String representing the cardholder name from credit/debit card purchases. Please include name ordering (i.e. “First Middle Last” or “Last, First Middle”) |
| CC Last Four, Last 4, or CC Token | Yes | 123456XXXXXX8466 | String representing the obfuscated last 4 digits from credit/debit card purchases. |
| Tender Amount | Yes | 24.19 | Post discount and post tax tendered amount |
| Tip Amount | Yes | 2 | Tip amount for the tender |
| Loyalty ID | No | Loy1234 | Alphanumeric String representing loyalty membership Id of a customer |
| Customer ID | No | Cust1234 | Alphanumeric String representing customer Id of a customer |

### Line Item

**File name:** _YYYY-MM-DD_lineitem.csv_

**Description:** The line item file is all items ordered on a check.

- The gross amount from check summary should be equal to the sum of line items.
- Refunded items should have a negative amount.
- If you have multiple of the same line items, you can either have individual line items for each or you can have one line item with an incremented count.
- The item count multiplied by item amount (amount of single item) must equal item total.

#### Data Specification for Line Item

| Column | Required | Example | Description |
| - | - | - | - |
| Date | Yes | 2020-01-01 | Date of the transaction (ISO 8601 format yyyy-mm-dd). |
| Store ID | Yes | 5812 | Alphanumeric string that uniquely identifies a store. |
| Check ID | Yes | 10001 | Alphanumeric string that uniquely identifies a transaction within a store on a given day. |
| Date Time (UTC) | No | 2017-12-14T21:05:00 | Date and time of the transaction in UTC(ISO 8601 format yyyy-mm-ddThh:mm:ss) |
| Date_Time (Local) | Yes | 2017-12-14T21:05:00 | Date and time of the transaction in store's local timezone (ISO 8601 format yyyy-mm-ddThh:mm:ss) |
| Item ID | Yes | 24 | Integer representing the code used by the POS to reference the item |
| Item Name | Yes | S Burrito | String representing name of item in reporting and on UI |
| Description | No | Steak Burrito | String representing long name of item |
| Category ID | Yes | 3 | Integer representing category ID of the item |
| Category Name | Yes | Burritos | String representing name of category |
| Category Description | No | All kinds of burritos | String representing description of category |
| Item Count | Yes | 2 | Integer representing the number of identical items purchased in the current transaction |
| Item Amount | Yes | 6.5 | Amount of single item |
| Item Tax | No | 3 | Amount of tax of item |
| Item Total | Yes | 13 | Amount of all identical items purchased in the transaction |
| Item Type | Yes | Sale | String representing item type (i.e. Sale, Refund, Revenue Item). If it's a refund, item amount has to be negative |
| Parent ID | Yes | NULL | If the item is a combo, one row should represent the combo and individual items should have item ID of the combo as their parent ID |

### Discount

**File name:** _YYYY-MM-DD_discount.csv_

**Description:** The discount file is all discounts applied to an individual check. We assume that a single check can have multiple discounts.

#### Data Specification for Discount

| Column | Required | Example | Description |
| - | - | - | - |
| Date | Yes | 2020-01-01 | Date of the transaction (ISO 8601 format yyyy-mm-dd). |
| Store ID | Yes | 5812 | Alphanumeric string that uniquely identifies a store. |
| Check ID | Yes | 10001 | Alphanumeric string that uniquely identifies a transaction within a store on a given day. |
| Date Time (UTC) | No | 2017-12-14T21:05:00 | Date and time of the transaction in UTC(ISO 8601 format yyyy-mm-ddThh:mm:ss) |
| Date_Time (Local) | Yes | 2017-12-14T21:05:00 | Date and time of the transaction in store's local timezone (ISO 8601 format yyyy-mm-ddThh:mm:ss) |
| Discount ID | Yes | 371 | Integer representing the discount |
| Discount Amount | Yes | 2.5 | Amount of discount applied for the specific discount_code |
| Discount Code | No | AQNZ2 | Alphanumeric string representing the specific discount code a customer redeemed |

### Comp

**File name:** _YYYY-MM-DD_comp.csv_

**Description:** The comp file is all comps applied to an individual check. We assume that a single check can have multiple comps.

#### Data Specification for Comp

| Column | Required | Example | Description |
| - | - | - | - |
| Date | Yes | 2020-01-01 | Date of the transaction (ISO 8601 format yyyy-mm-dd). |
| Store ID | Yes | 5812 | Alphanumeric string that uniquely identifies a store. |
| Check ID | Yes | 10001 | Alphanumeric string that uniquely identifies a transaction within a store on a given day. |
| Date Time (UTC) | No | 2017-12-14T21:05:00 | Date and time of the transaction in UTC(ISO 8601 format yyyy-mm-ddThh:mm:ss) |
| Date_Time (Local) | Yes | 2017-12-14T21:05:00 | Date and time of the transaction in store's local timezone (ISO 8601 format yyyy-mm-ddThh:mm:ss) |
| Comp ID | Yes | 371 | Integer representing the comp |
| Comp Amount | Yes | 2.5 | Amount of comp applied for the specific comp Id |
| Comp Code | No | AQNZ2 | Alphanumeric string representing the specific comp code a customer redeemed |

### Tender Type Dimension

**File name:** _YYYY-MM-DD_tendertype_dim.csv_

**Description:** The tender type file is a list of tenders with their corresponding ID. For example: Cash, Credit, Gift Card.

#### Data Specification for Tender Type Dimension

| Column | Required | Example | Description |
| - | - | - | - |
| Tender Type ID | Yes | 1 | Integer representing the tender type |
| Tender Description | Yes | Cash | String representing the tender type description |

### Tender Type Detail Dimension

**File name:** _YYYY-MM-DD_tenderdetail_dim.csv_

**Description:** The tender type detail file is a list of more detailed tender types with their corresponding ID. For example: Visa, Mastercard, Amex.

#### Data Specification for Tender Type Detail Dimension

| Column | Required | Example | Description |
| - | - | - | - |
| Tender Type ID | Yes | 1 | Integer representing the tender type |
| Tender Description | Yes | Visa | String representing the tender type description |

### Order Source Dimension

**File name:** _YYYY-MM-DD_ordersource_dim.csv_

**Description:** The order source file is a list of order sources with their corresponding ID. You’ll be able to create segments on the UI based on these order sources, so it’s up to you how granular to get.

#### Data Specification for Order Source Dimension

| Column | Required | Example | Description |
| - | - | - | - |
| Order Source ID | Yes | 1 | Integer representing the order source |
| Order Source Description | Yes | To Go | String representing the order source description |

### Discount Dimension

**File name:** _YYYY-MM-DD_discount_dim.csv_

**Description:** The discount file is a list of all discounts and their corresponding ID.

#### Data Specification for Discount Dimension

| Column | Required | Example | Description |
| - | - | - | - |
| Promo Code ID | Yes | 371 | String representing the discount code captured by POS or entered by cashier |
| Promo Desc | Yes | Taco Tuesday discount | Human understandable description of the discount |

### Comp Dimension

**File name:** _YYYY-MM-DD_comp_dim.csv_

**Description:** The comp file is a list of all comps and their corresponding ID.

#### Data Specification for Comp Dimension

| Column | Required | Example | Description |
| - | - | - | - |
| Comp Code ID | Yes | 123 | String representing the comp code captured by POS or entered by cashier |
| Comp Desc | Yes | BOGO Entree | Human understandable description of the comp |

### Location

**File name:** _YYYY-MM-DD_location.csv_

**Description:** The location file is a list of all open and closed locations.

#### Data Specification for Location

| Column | Required | Example | Description |
| - | - | - | - |
| Business Name | Yes | Michigan Avenue | String or integer representing how the business refers to the location |
| Site ID | Yes | 3572 | Integer representing how the data warehouse and/or business refers to the location |
| Phone | Yes | 312-263-0019 | 10 digit phone number |
| Address Line 1 | Yes | 326 N. Michigan Ave | Primary street address of customer |
| Address Line 2 | No | Apt #2 | Second street address of customer |
| City | Yes | Chicago | Name of the city |
| State | No | IL | Name of the state |
| Zip Code | Yes | 60601 | Numeric string representing the zip code |
| Time Zone | Yes | America/New_York |   |
| Status | Yes | Open | Status of store (open/closed) |

### Customer

**File name:** _YYYY-MM-DD_customer.csv_

**Description:** The customer file is a list of all customers, including any unique IDs you use to identify them. Each day's transmission should include any new or changed contacts from the previous day.

#### Data Specification for Customer

| Column | Required | Example | Description |
| - | - | - | - |
| Created Date | Yes | 2020-06-01 | Date and time the customer record was created |
| Customer ID | Yes | Cust007 | Alphanumeric string that uniquely identifies a specific customer |
| Loyalty ID | Yes | Loya007 | Alphanumeric string that uniquely identifies a specific customer, if different from Customer ID |
| Favorite Store | Yes | 3123 | Alphanumeric string that uniquely identifies the specific store where the customer record was created |
| First Name | Yes | James | First name of the customer |
| Last Name | Yes | Bond | Last name of the customer |
| Full Name | No | James Bond | Full name of the customer in case there is no separation of first and last |
| Email | Yes | james@bond.com | Customer’s email address |
| Cell Phone | No | 609-232-2312 | Cell Phone |
| Home Phone | No | 609-232-2312 | Primary phone number |
| Other Phone | No | 609-232-2312 | Any additional phone number associated with the customer |
| Address Line 1 | Yes | 326 N. Michigan Ave | Primary street address of customer |
| Address Line 2 | Yes | Apt #2 | Second street address of customer |
| Primary City | No | Chicago | Primary city |
| Primary State | No | IL | Primary state |
| Primary Zip | No | 60601 | Primary zip |
| Date of Birth | No | 1977-01-01 | Date of birth in YYYY-MM-DD |
| Loyalty Enrollment Date | No | America/New_York | Date on which customer enrolled in loyalty program. |
| Loyalty Points | No | 1234 | Current points associated with loyalty profile |

## Customizations and Change Requests

If the data schema doesn’t match the agreed upon format above or the data transfer comes through differently, both parties should sign a project change request. That change request will include any modification, timeline updates, and potential implementation fees.