# Bridg CDP Data Specification | Standard Implementation

## Introduction

Welcome to **Bridg.** We’re excited to partner with you and your team so you can start leveraging Bridg’s proprietary tools and insights to accelerate your company's business intelligence and targeted marketing capabilities.

This document is a consolidated guide detailing Bridg's requirements for source data and receiving the data from your company's data warehouse and/or third-party systems.

We’ll use this document to:

- Outline what type of data you should send to Bridg, including information on required and optional data fields;
- Specify our required schema for data transferred to Bridg including data formats, naming conventions, and file organization;
- Identify gaps or potential augmentation opportunities in source data prior to ingestion and processing by Bridg.

Please use this document to identify areas where your source system data varies from the Bridg specification. Review the specification carefully and highlight to the Bridg Implementation Team any areas where there are potential differences which may need to be accommodated before sending sample data to Bridg.

## List of Contents for Quick Reference

- [Overview of File Structure](#overview-of-file-structure)
- [Required Data Format](#required-data-format)
- [Daily Data Transfer & Cadence](#daily-data-transfer--cadence)
- [Catchup Data](#catchup-data)
- [Historical Data](#historical-data)
- [Data QA and Validation](#data-qa-and-validation)
- [Customizations and Change Requests:](#customizations-and-change-requests)
- [File Details: Data Formats and Naming Conventions](#file-details-data-formats-and-naming-conventions)
  - [Transaction Summary](#1-transaction-summary-file)
  - [Tender](#2-tender-file)
  - [Line Item](#3-line-item-file)
  - [Discount Summary](#4-discount-summary-file)
  - [Customers](#5-customers-file)
  - [Product dimension](#6-product-dimension-file)
  - [Discount Detail dimension](#7-discount-detail-dimension-file)
  - [Location dimension](#8-location-dimension-file)

## Overview of File Structure

### Summary of Files:

Our implementation data specification includes (5) fact files and (3) dimension files.

These Fact Files contain data that changes frequently and is the core data used in the Bridg platform. Bridg expects to receive the following Fact Files on at least a daily basis for each of your locations.

* **Transaction Summary** - Provides summary facts for each customer order recorded, where each order is represented as a single row of data.
* **Tender** - Contains an entry for each payment or tender included as part of an order included in Transaction Summary. If an order in Transaction Summary includes three payment types (or tenders), there should be one row of data in the Transaction Summary file and three rows in the Tender file. There should be at least one row in Tender for each row in Transaction Summary.
* **Line Item** - Contains all individual transaction log entries (e.g. products purchased, tax applied, etc.) that are associated with each order shown in Transaction Summary. If 10 products were purchased, there will be 10 rows for that transaction in Line Item and 1 row in Transaction Summary.
* **Discount Summary** - Contains the facts for all discounts and promotions applied to transactions included in Transaction Summary. There will be at least one row for each transaction included in Transaction Summary where a discount, coupon, or promotion was applied.  If multiple discounts were applied to a single transaction, the number of rows in Discount Summary should be equal to the number of discounts applied for 1 row in Transaction Summary.
* **Customers** - Contains all customers and related customer identifying data including unique IDs for each individual consumer. There should be one row for each consumer identified.

These Dimension Files contain metadata for the Fact Files. Though data in Dimension Files may change less frequently than the facts, it is necessary for Bridg to receive these Dimension Files on the same frequency as the related Fact Files. Dimension Files are used to make fact data meaningful in Bridg CDP and Bridg Data Lake for end users.

* **Product** - Provides metadata for products shown in the Line Item fact file. Each row of data in Product is expected to correspond to a unique Product ID used to identify the product in Line Item.
* **Discount Detail** - Contains metadata for each type of discount which may appear in the Discount Summary fact file. Each row of data in Discount Detail is expected to correspond to a unique Discount ID used to identify the discount in Discount Summary.
* **Location** - Contains facts for all open and closed store locations where transactions in Transaction Summary may have occurred or consumers identified in Customers may have completed a transaction. This file contains a row of store location facts for each opened or closed store location and any related data for each.

## Required Data Format

All files must be in standard utf-8 encoded CSV format and sent as a single level atomic zip file.

It is important that each of the files specified follow the naming convention listed below and that they include the transaction date at the end of the file name as 'YYYY-MM-DD'.  For example, Transaction Summary must be provided using the naming convention "transaction_summary_YYYY-MM-DD.csv", where YYYY-MM-DD is replaced by the year, month, and day the transaction occurred.

## Daily Data Transfer & Cadence

All files must be transferred at the same time daily, either through a Secure File Transfer Protocol (SFTP) or AWS S3 Sync.

For SFTP: Bridg will provide the SFTP host; you should provide a public key for Bridg to attach to the SFTP.

For AWS S3 Syncs: Files should be pushed to our AWS infrastructure.

## Catchup Data

Sometimes, Fact data for some locations may not be available on time and there will be a need to transmit catch-up, or lagged, data to Bridg.  If that occurs, Fact data rows should be included in the files for whatever day they become available.

## Historical Data

As part of the original data feed, we’ll be ingesting two or more years of historical data. The historical data should come through in an identical format as the daily data both in terms of column formatting as well as the daily zip file (YYYY-MM-DD).

## Data QA and Validation

Data processing at Bridg cannot commence until we have tested and validated that data sent to Bridg conforms to the specification detailed herein and that we can properly map the data for our ingestion and normalization steps.  The first step of this validation process is for your team to provide to Bridg one month of Fact and Dimension sample data.

## Customizations and Change Requests:

It is important that the data schema of the source input data delivered to Bridg matches the specification detailed herein. If the schema of the data delivered to Bridg or the final data transfer does not match the tested and approved sample data, Bridg will contact the project sponsor to discuss implications which may include modifications to our standard specification, timeline implications, and potential implementation fees.

Prior to sending data, please review this document in detail and highlight to your Bridg Implementation Team any deviations from the requested data or specification. The Bridg Implementation Team will confirm which changes can be accomodated as well as areas which require a workaround.

## File Details: Data Formats and Naming Conventions

This section outlines each of the Fact Files and Dimension Files we expect, including details around how each field is used, acceptable data formats, and naming conventions.  Please review the tables included for each file type for specifics about the headers and content of the data we expect to receive.

### 1. Transaction Summary File

* <ins>File Description</ins>: Provides summary facts for each customer order recorded, where each order is represented as a single row of data.
* <ins>Required File Naming Convention</ins>: "transaction_summary_YYYY-MM-DD.csv", where YYYY-MM-DD is replaced by the year, month, and day the transaction occurred.
* <ins>Required File Format</ins>: Standard UTF-8 encoded CSV format zipped as a single level atomic file.  The YYYY-MM-DD of the atomic zip file must match the YYYY-MM-DD of the individual files

#### *Data Specification for Transaction Summary:*


|   | ColumnName<br />*System Field* | Required | FieldDescription | LongExplanationofDataFormat<br />*SampleData* | EvenLongerReferencesetcetc |
| :- | - | :-: | :- | :- | :- |
| 1 | Date*date* | Y | The date each transaction was placed | Provide in ISO 8601 format (YYYY-MM-DD):*2017-12-14* |   |
| 2 | Location ID*location_id* | Y | The ID used in our system to uniquely identify a specific store / location for each tender | Provide as an alphanumeric string:*5812* | Cross-referenced in multiple files; used to join across files Transaction Summary, Tender, Line Item, and Discount Summary |
| 3 | Check ID*check_id* | Y | The ID used in our system to uniquely identify a transaction within a store on a given day | Provide as an alphanumeric string:*10001* | • Cross-referenced in multiple files; used to join across files Transaction Summary, Tender, Line Item, and Discount Summary• Does not have to be unique across all stores or all time. The same ID could be used on multiple days, however Bridg needs to understand any rules which make this field a unique key. |
| 4 | Date & Time (Local)*datetime_local* | Y | Date and time of the transaction in the local timezone of the store / location | Provide in ISO 8601 format (yyyy-mm-ddThh:mm:ss):*2017-12-14T21:05:00* |   |
| 5 | Date & Time (UTC)*datetime_utc* | N | Date and time of the transaction in Coordinated Universal Time (UTC) | Provide in ISO 8601 format (yyyy-mm-ddThh:mm:ss):*2017-12-14T21:05:00* |   |
| 6 | Tender Amount*tender* | Y | Dollar value of the total Tendered amount for the transaction (with Discount and Tax included) | Provide as a decimal value:*24.19* | Should be the sum of Pre-Tax / Pre-Discount Amount + Discounts + Tax |
| 7 | Tender Tax Amount*tender_tax* | Y | Dollar value of total tax charged for the transaction | Provide as a decimal value:*2.5* | Should match tax shown for the same Check ID present in Line Item |
| 8 | Discount Amount*discount* | Y | Dollar value of all Discounts of all types applied to a transaction | Provide as a decimal value:*3.25* | Should be the sum of all comps, discounts, and promos with the same Check ID in Discount Summary, and equivalent to any discount with the same Check ID if present in Line Item. |
| 9 | Pre-Tax / Pre-Discount Amount*base_amount* | Y | The base price (sub-total) for each transaction before Discounts and Tax have been applied | Provide as a decimal value:*18.44* | • Should be equal to the total of all tenders with the same Check ID present in Tender, minus Tax and Discounts.• Should be equal to the total of all line items with the same Check ID present in Line Item, excluding any line items present in Discount Summary or Tax line items. |
| 10 | Order Source ID*order_source_id* | Y | A numerical identifier that represents a specific order source for each transaction | Provide as an integer that corresponds to one of the predefined order sources in our system:*2* | Acceptable Order Source ID Values:1 = Store2 = Web3 = Mobile4 = Call Center5 = Other(In-Store)99 = UndefinedAdditional detail is provided in Sales Channel below. |
| 11 | Fulfillment Channel ID*fulfillment_channel_id* | N | An optional identifier that can be used in addition to Order Source ID that represents a specific fulfillment channel for each transaction | Provide as an integer that corresponds to one of the predefined fulfillment channels in our system:*2* | Acceptable Fulfillment Channel ID Values:<br />1 = In-Store<br />2 = Delivery<br />3 = Pickup<br />4 = Drive-Through<br />5 = Curbside<br />6 = Other<br />99 = Undefined |
| 12 | Sales Channel ID*sales_channel_id* | N | An optional identifier that can be used in addition to Order Source ID that represents a specific sales channel for each transaction | Provide as an integer that corresponds to one of the predefined sales channels in our system:*2* | Acceptable Sales Channel ID Values:<br />1 = In-Store<br />2 = Web<br />3 = Mobile<br />4 = Call Center<br />5 = Kiosk<br />6 = Third-Party<br />7 = GrubHub<br />8 = Postmates<br />9 = Doordash<br />10 = UberEats<br />11 = Delivery.com<br />12 = Caviar<br />13 = ChowNow<br />14 = EatStreet<br />99 = Undefined |
| 13 | Sales Platform ID*sales_platform_id* | N | An optional identifier that can be used in addition to Order Source ID that represents a specific sales platform for each transaction | Provide as an integer that corresponds to one of the predefined sales platforms in our system:*2* | Acceptable Sales Platform ID Values:<br />1 = Kiosk<br />2 = POS<br />3 = Call Center<br />4 = Android<br />5 = iOS<br />6 = Internet Explorer<br />7 = Google Chrome<br />8 = Opera<br />9 = Safari<br />99 = Undefined |
| 14 | Employee ID*employee_id* | N | The ID number of the employee who completed each transaction, represented as it would be in your internal POS system | Provide as an alphanumeric value:*1222233* | Cross-referenced in multiple files; used to join across files Transaction Summary, and Discount Summary |
| 15 | Register ID*register_id* | N | The ID number associated with the register where each transaction was placed, represented as it would be in your internal POS system | Provide as a numeric value:*3322* | Cross-referenced in multiple files; used to join across files Transaction Summary and Tender |

### 2. Tender File

* <ins>File Description</ins>: Contains an entry for each payment or tender included as part of an order included in Transaction Summary. If an order in Transaction Summary includes three payment types (or tenders), there should be one row of data in the Transaction Summary file and three rows in the Tender file. There should be at least one row in Tender for each row in Transaction Summary.
* <ins>Required File Naming Convention</ins>: "tender_YYYY-MM-DD.csv", where YYYY-MM-DD is replaced by the year, month, and day the transaction occurred.
* <ins>Required File Format</ins>: Standard UTF-8 encoded CSV format zipped as a single level atomic file. For each file, The YYYY-MM-DD of the atomic zip file must match the YYYY-MM-DD of the individual files.

#### *Data Specification for Tender:*


|   | ColumnName<br />*System Field* | Required | FieldDescription | LongExplanationofDataFormat<br />*SampleData* | EvenLongerReferencesetc |
| :- | - | :-: | :- | :- | :- |
| 1 | Date*date* | Y | The date each transaction was placed | Provide in ISO 8601 format (YYYY-MM-DD):*2017-12-14* |   |
| 2 | Location ID*location_id* | Y | The ID used in our system to uniquely identify a specific store / location for each tender | Provide as an alphanumeric string:*5812* | Cross-referenced in multiple files; used to join across files Transaction Summary, Tender, Line Item, and Discount Summary |
| 3 | Check ID*check_id* | Y | The ID used in our system to uniquely identify a transaction within a store on a given day | Provide as an alphanumeric string:*10001* | • Cross-referenced in multiple files; used to join across files Transaction Summary, Tender, Line Item, and Discount Summary• Does not have to be unique across all stores or all time. The same ID could be used on multiple days, however Bridg needs to understand any rules which make this field a unique key. |
| 4 | Date & Time (Local)*datetime_local* | Y | Date and time of the transaction in the local timezone of the store / location | Provide in ISO 8601 format (yyyy-mm-ddThh:mm:ss):*2017-12-14T21:05:00* |   |
| 5 | Date & Time (UTC)*datetime_utc* | N | Date and time of the transaction in Coordinated Universal Time (UTC) | Provide in ISO 8601 format (yyyy-mm-ddThh:mm:ss):*2017-12-14T21:05:00* |   |
| 6 | Tender Type ID*tender_type_id* | Y | A numerical identifier that represents a specific tender type for each transaction | Provide as an integer that corresponds to one of the predefined tender types in our system:*2* | **Insert all available tender type IDs and their corresponding tender types |
| 7 | Tender Type Detail ID*tender_type_detail_id* | N | An optional identifier that can be used in addition to Tender Type ID that specifies a second level of detail for each transaction's tender type (i.e. a type of credit card or a type of check) | Provide as an integer that corresponds to one of the predefined detailed tender types in our system:*2* | **Insert all available tender type detail IDs and their corresponding detailed tender types |
| 8 | Cardholder Name*cardholder_name* | Y | The full cardholder name associated with each credit card or debit card transaction | Provided as a string in the format used by the associated payment network (either First + Middle + Last or Last, First Middle) :*Jackson, Robert L.* or *Robert L. Jackson* | Please specify which format will be used |
| 9 | Credit Card Remnants*cc_last_four* | Y | The obfuscated digits from the card number, used to map each transaction to the credit or debit card used for the purchase | Provided as a numeric string in the format used by the associated payment network (either last four digits, First 6 digits + Last 4 digits, or a Token):*123456XXXXXX8466* | In the case of a credit card Token, tokens must be unique across all customers in all locations |
| 10 | Tender Amount*tender_amount* | Y | Dollar value of the total Tendered amount for the transaction (with Discount and Tax included) | Provide as a decimal value:*24.19* | Should be the sum of Pre-Tax / Pre-Discount Amount + Discounts + Tax |
| 11 | Tender Tip Amount*tender_tip* | Y | Dollar value of tip applied to the transaction | Provide as a decimal value:*6.00* |   |
| 12 | Tender Tax Amount*tender_tax* | N | Dollar value of total tax charged for the transaction | Provide as a decimal value:*2.5 | Should match tax shown for the same Check ID present in Line Item |
| 13 | Currency*currency* | N | Currency of the transaction; will default to USD if no data is provided | Provide in ISO 4217 format (3 digit country + currency code):*USD* |   |
| 14 | Loyalty ID*loyalty_id* | N | The primary identifier used to determine each unique loyalty customer, used to map loyalty transactions back to the customer record | Provided as a string in the format used by the loyalty vendor:*1234567* | • Used as part of Bridg's identification process; data must be formatted consistently across Tender and Customer files• If loyalty data is being sent as part of this implementation this field should be considered Required |
| 15 | Loyalty Vendor*loyalty_vendor* | N | The name of the loyalty vendor (also the source of the loyalty data) | Provide as a string:*aloha* | Should be provided in addition to Loyalty ID if loyalty data is being sent as part of this implementation |
| 16 | Customer ID*customer_id* | N | The unique customer identifier used internally at your organization | Provided as a string in the format used by your organization:*CDM12345* | Cross-referenced in multiple files; used to join across files Tender, Discount Summary, and Customer |
| 17 | Employee ID*employee_id* | N | The ID number of the employee who completed each transaction, represented as it would be in your POS system | Provide as an alphanumeric value:*1222233* | Cross-referenced in multiple files; used to join across files Transaction Summary, and Discount Summary |
| 18 | Register ID*register_id* | N | The ID number associated with the register where the transaction was placed | Provided as a numeric string in the format used by your internal organization:*3322* | Cross-referenced in multiple files; used to join across files Transaction Summary and Tender |

### 3. Line Item File

* <ins>File Description</ins>: Contains all individual transaction log entries (e.g. products purchased, tax applied, etc.) that are associated with each order shown in Transaction Summary. If 10 products were purchased, there will be 10 rows for that transaction in Line Item and 1 row in Transaction Summary.
* <ins>Data Structure Considerations</ins>: If there are multiple of the same line item, data can be provided as individual line items for each item or as a single line item with an incremented amount.  Refunded items should have a negative amount.
* <ins>Required File Naming Convention</ins>: "line_item_YYYY-MM-DD.csv", where YYYY-MM-DD is replaced by the year, month, and day the transaction occurred.
* <ins>Required File Format</ins>: Standard UTF-8 encoded CSV format zipped as a single level atomic file. For each file, The YYYY-MM-DD of the atomic zip file must match the YYYY-MM-DD of the individual files.

#### ***Data Specification for Line Item:*


|   | ColumnName<br />*System Field* | Required | FieldDescription | LongExplanationofDataFormat<br />*SampleData* | EvenLongerReferencesetc |
| :- | - | :-: | :- | :- | :- |
| 1 | Date*date* | Y | The date each transaction was placed | Provide in ISO 8601 format (YYYY-MM-DD):*2017-12-14* |   |
| 2 | Location ID*location_id* | Y | The ID used in our system to uniquely identify a specific store / location for each tender | Provide as an alphanumeric string:*5812* | Cross-referenced in multiple files; used to join across files Transaction Summary, Tender, Line Item, and Discount Summary |
| 3 | Check ID*check_id* | Y | The ID used in our system to uniquely identify a transaction within a store on a given day | Provide as an alphanumeric string:*10001* | • Cross-referenced in multiple files; used to join across files Transaction Summary, Tender, Line Item, and Discount Summary• Does not have to be unique across all stores or all time. The same ID could be used on multiple days, however Bridg needs to understand any rules which make this field a unique key. |
| 4 | Date & Time (Local)*datetime_local* | Y | Date and time of the transaction in the local timezone of the store / location | Provide in ISO 8601 format (yyyy-mm-ddThh:mm:ss):*2017-12-14T21:05:00* |   |
| 5 | Date & Time (UTC)*datetime_utc* | N | Date and time of the transaction in Coordinated Universal Time (UTC) | Provide in ISO 8601 format (yyyy-mm-ddThh:mm:ss):*2017-12-14T21:05:00* |   |
| 6 | Product ID*product_id* | Y | The code captured by your POS system to uniquely identify each individual product | Provide as an integer:*24* | Cross-referenced in multiple files; used to join across files Line Item, Discount Summary, and Product |
| 7 | Product Detail ID*product_detail_id* | N | An optional identifier that can be used in addition to Product ID to specify a second level of detail for products in a check (most often used to identify individual products sold as part of a product bundle or combo) | Provide as an integer:*24* |   |
| 8 | Product Modifier ID*product_modifier_id* | N | An optional identifier that can be used to specify modifications made to each product in a check (most often used by restaurants i.e. 'no pickles') | Provided as an integer that represents your internal code used for each modifier:*86* | A single product can have multiple product identifiers |
| 9 | Category ID*category_id* | Y | A numerical identifier that represents a specific product category for each product purchased in a check | Provide as an integer that represents one of your internal product categories:*24* | If product categories exist, this field should be considered Required |
| 10 | Product Quantity*product_quantity* | Y | The count of identical products or units of identical products purchased in a transaction (i.e. 2 pencils, or 2 pounds of apples) | Provide as an integer:*3* |   |
| 11 | Product Unit of Measurement*product_measurement_unit* | N | The unit of measurement associated with the quantity of each product purchased in a transaction (i.e. pounds or liters) | Provide as a string with the name of the unit of measurement; do not include the integer value of the quantity:*lbs* |   |
| 12 | Price Per Unit*unit_price* | N | The dollar value of a single product or a single unit of a product purchased in a transaction | Provide as a decimal value:*0.65* |   |
| 13 | Product Total Amount*product_total_amount* | Y | The total dollar amount of all units of identical products purchased in a transaction | Provide as a decimal value:*1.95* | • Should equal Price Per Unit multiplied by Product Quantity• Refunds should be represented as a negative amount |
| 14 | Product Tax Amount*product_tax* | N | Dollar value of tax charged for a given product in a transaction | Provide as a decimal value:*2.5* |   |
| 15 | Sale Type*sale_type* | Y | The type of sale for each line item in a check (i.e. Refund, Sale, Revenue Item) where sale type is one of several predefined values in our system | Provide as a string:*Refund* | • Must be one of the following acceptable values:<br />Sale<br />Refund<br />Revenue Item<br />• For Refunds, the Product Total Amount should be listed as a negative |
| 16 | Line Item ID*line_item_id* | N | Identifier used to map data back to a specific line item in a check in the case there are multiple rows of data for a single product (i.e. more than one product modifier, or a product bundle) | Provide as an integer:*3* | For Combo items or Product Bundles, there will often be one line per product |
| 17 | Parent Line Item ID*parent_line_item_id* | N | Identifier used to map second-level line items to a parent line item in the case there are nested line items for a check (i.e. a combo or product bundle where individual products are separated) | Provide as an integer:*1* | For Combo items or Product Bundles, the Line Item ID of the Combo should be entered as the Parent Line Item ID |

### 4. Discount Summary File

* <ins>File Description</ins>: Contains the facts for all discounts and promotions applied to transactions included in Transaction Summary. There will be at least one row for each transaction included in Transaction Summary where a discount, coupon, or promotion was applied.  If multiple discounts were applied to a single transaction, the number of rows in Discount Summary should be equal to the number of discounts applied for 1 row in Transaction Summary.
* <ins>Required File Naming Convention</ins>: "discount_summary_YYYY-MM-DD.csv", where YYYY-MM-DD is replaced by the year, month, and day the transaction occurred.
* <ins>Required File Format</ins>: Standard UTF-8 encoded CSV format zipped as a single level atomic file. For each file, The YYYY-MM-DD of the atomic zip file must match the YYYY-MM-DD of the individual files.

#### *Data Specification for Discount Summary:*


|   | ColumnName<br />*System Field* | Required | FieldDescription | LongExplanationofDataFormat<br />*SampleData* | EvenLongerReferencesetc |
| :- | - | :-: | :- | :- | :- |
| 1 | Date*date* | Y | The date each transaction was placed | Provide in ISO 8601 format (YYYY-MM-DD):*2017-12-14* |   |
| 2 | Location ID*location_id* | Y | The ID used in our system to uniquely identify a specific store / location for each tender | Provide as an alphanumeric string:*5812* | Cross-referenced in multiple files; used to join across files Transaction Summary, Tender, Line Item, and Discount Summary |
| 3 | Check ID*check_id* | Y | The ID used in our system to uniquely identify a transaction within a store on a given day | Provide as an alphanumeric string:*10001* | • Cross-referenced in multiple files; used to join across files Transaction Summary, Tender, Line Item, and Discount Summary• Does not have to be unique across all stores or all time. The same ID could be used on multiple days, however Bridg needs to understand any rules which make this field a unique key. |
| 4 | Date & Time (Local)*datetime_local* | Y | Date and time of the transaction in the local timezone of the store / location | ISO 8601 format (yyyy-mm-ddThh:mm:ss):*2017-12-14T21:05:00* |   |
| 5 | Date & Time (UTC)*datetime_utc* | N | Date and time of the transaction in Coordinated Universal Time (UTC) | ISO 8601 format (yyyy-mm-ddThh:mm:ss):*2017-12-14T21:05:00* |   |
| 6 | Discount ID*discount_id* | Y | The code used to uniquely identify each discount applied to a check, as captured by your POS system or entered by the cashier | Provide as a numeric string:*371* | • Each Discount ID will correspond to a Discount Description in Discount Detail<br />• Cross-referenced in multiple files; used to join across files Discount Summary and Discount Detail |
| 7 | Discount Type*discount_type* | Y | The type of discount applied for a given check, where discount type is one of several predefined values in our system | Provide as a string:*Comp* | • Must be one of the following acceptable values:<br />Sale<br />Refund<br />Revenue Item<br />Comp<br />Promo<br />Discount<br />• Cross-referenced in multiple files; used to join across files Discount Summary and Discount Detail |
| 8 | Discount Amount*discount_amount* | Y | The dollar value associated with each individual Discount ID for a given check | Provide as a decimal value:*2.5* |   |
| 9 | Discount Code*discount_code* | N | The code used by your internal organization and redeemed by the customer for a given check | Provide as an alphanumeric string:*AQNZ2* |   |
| 10 | Discounted Price*discounted_price* | N | The reduced price for a given check after the discount was applied |   | Should be equal to the Pre-Tax / Pre-Discount Amount minus Discount Amount |
| 11 | Currency*currency* | N | Currency of the transaction; will default to USD if no data is provided | Provide in ISO 4217 format (3 digit country + currency code):*USD* |   |
| 12 | Product ID*product_id* | N | The code captured by your POS system to uniquely identify each individual product | Provide as an integer:*24* | Cross-referenced in multiple files; used to join across files Line Item, Discount Summary, and Product |
| 13 | Customer ID*customer_id* | N | The unique customer identifier used internally at your organization | Provided as a string value in the format used by your organization:*CDM12345* | Cross-referenced in multiple files; used to join across files Tender, Discount Summary, and Customer |
| 14 | Employee ID*employee_id* | N | The ID number of the employee who completed each transaction, represented as it would be in your internal POS system | Provide as an alphanumeric string:*AB1222233* | Cross-referenced in multiple files; used to join across files Transaction Summary, and Discount Summary |

### 5. Customers File

* <ins>File Description</ins>: Contains all customers and related customer identifying data including unique IDs for each individual consumer. There should be one row for each consumer identified.
* <ins>Required File Naming Convention</ins>: "customers_YYYY-MM-DD.csv", where YYYY-MM-DD is replaced by the year, month, and day the transfer occurred.
* <ins>Required File Format</ins>: Standard UTF-8 encoded CSV format zipped as a single level atomic file. For each file, The YYYY-MM-DD of the atomic zip file must match the YYYY-MM-DD of the individual files.

#### *Data Specification for Customer:*


|   | ColumnName<br />*System Field* | Required | FieldDescription | LongExplanationofDataFormat<br />*SampleData* | EvenLongerReferencesetc |
| :- | - | :-: | :- | :- | :- |
| 1 | Created Date*customer_creation_date* | Y | The date and time the customer record was created | Provide in ISO 8601 format (yyyy-mm-ddThh:mm:ss):*2017-12-14T21:05:00* |   |
| 2 | Customer ID*customer_id* | Y | The unique customer identifier used internally at your organization | Provided as an alphanumeric string in the format used by your organization:*CDM12345* | Cross-referenced in multiple files; used to join across files Tender, Discount Summary, and Customer |
| 3 | Loyalty ID*loyalty_id* | N | The primary identifier used to determine each unique loyalty customer, used to map loyalty transactions back to the customer record | Provided as a string value in the format used by the loyalty vendor:*1234567* | • Used as part of Bridg's identification process; data must be formatted consistently across Tender and Customer files• If loyalty data is being sent as part of this implementation this field should be considered Required |
| 4 | External ID*external_id* | N | An optional field that can be used to store an additional customer identifier used by your organization (often used to pass an ID from a loyalty vendor or ESP) | Provide as an alphanumeric string:*EB1222233* |   |
| 5 | Favorite Store*location_id* | Y | The Location ID of the store / location where the customer record was created | Provided as a string value in the format used by the loyalty vendor:*1234567* |   |
| 6 | First Name*first_name* | Y | Customer's first name | Provide as a string:*John* |   |
| 7 | Middle Name*middle_name* | N | Customer's middle name | Provide as a string:*James* |   |
| 8 | Last Name*last_name* | Y | Customer's last name | Provide as a string:*Doe* |   |
| 9 | Full Name*full_name* | N | The full name of the customer in the case data for first and last name are not separated | Provide as a string:*John Doe* |   |
| 10 | Title*title* | N | The customer's title | Provide as a string:*Mr.* |   |
| 11 | Suffix*suffix* | N | The customer's suffix | Provide as a string:*Esq.* |   |
| 12 | Email*email* | Y | The primary email address on file for the customer | Provide as a string:*name@email.com* |   |
| 13 | Cell Phone*cell_phone* | N | The cell phone on file for the customer | Provide as a string:*312-263-0019* |   |
| 14 | Home Phone*home_phone* | N | The home phone on file for the customer | Provide as a string:*312-263-0019* |   |
| 15 | Other Phone*other_phone* | N | An additional phone number on file for the customer | Provide as a string:*312-263-0019* |   |
| 16 | Address Line 1*address_line_1* | Y | The first line in the customer's address on file | Provide as a string:*326 N. Michigan Avenue* |   |
| 16 | Address Line 2*address_line_2* | N | The second line in the customer's address on file | Provide as a string:*Apt. 2* | This field should be considered Required for addresses with apartment or unit numbers |
| 17 | Primary City*city* | N | The city associated with the customer's primary address on file | Provide as a string:*Chicago* |   |
| 18 | Primary State*state* | N | The state associated with the customer's primary address on file | Provide as a string:*IL* |   |
| 19 | Primary Postal Code*postal_code* | N | The postal code associated with the customer's primary address on file | Provide as a string:*60601* |   |
| 20 | Date of Birth*dob* | N | The customer's date of birth | Provided in the format YYYY-MM-DD:*1980-04-22* |   |
| 21 | Age*age* | N | The customer's age (sometimes passed from third-party vendor system) | Provide as a numeric string:*37* |   |
| 22 | Gender*gender* | N | The customer's gender (sometimes passed from third-party vendor system) | Provide as a string:*Female* |   |
| 23 | Marital Status*marital_status* | N | The customer's marital status (sometimes passed from third-party vendor system) | Provide as a string:*Married* |   |
| 24 | Wedding Anniversary Date*wedding_anniversary_date* | N | The customer's Wedding Anniversary date (sometimes passed from third-party vendor system) | Provided in the format YYYY-MM-DD:*1990-10-05* |   |
| 25 | Email Registration Date*registration_date* | N | The date on which the Customer ID linked to the account record first became active | Provided in the format YYYY-MM-DD:*2005-01-10* |   |
| 26 | Loyalty Enrollment Date*loyalty_enrollment_date* | N | The date on which the customer was first enrolled in the loyalty program (the date a Loyalty ID was associated with the customer record) | Provided in the format YYYY-MM-DD:*2005-01-10* |   |
| 27 | Loyalty Card Number*loyalty_card_number* | N | The full, unredacted card number associated with the loyalty account | Provided as a numeric string in the format used by the associated payment network:*1234567812345678* | For some loyalty programs the Loyalty Card Number is used in place of a unique Loyalty ID |
| 28 | Lifetime Loyalty Points Earned*loyalty_points_earned* | N | The total accumulated loyalty points since the loyalty account was activated | Provide as a numeric string:*12550* | Displayed in Bridg's UI |
| 29 | Loyalty Points Balance*loyalty_points_balance* | N | The current points available to be redeemed in a customer's loyalty account | Provide as a numeric string:*550* |   |
| 30 | Loyalty Tier*loyalty_tier* | N | The tier associated with the customer's loyalty account | Provide as a string:*Premier* |   |
| 31 | Loyalty Vendor*loyalty_vendor* | N | The name of the loyalty vendor (also the source of the loyalty data) | Provide as a string:*Aloha* |   |
| 32 | App Activation Date*app_activation_date* | N | The date the customer downloaded the app (in the case a mobile app is used to transact on a mobile device) | Provide in the format YYYY-MM-DD:*2005-01-10* |   |
| 33 | Mobile Device ID*mobile_device_id* | N | The unique customer identifier used to represent a customer's mobile device(s) | Provide as an alphanumeric string based on the format used by the device manufacturer:*EA7583CD-A667-48BC-B806-42ECB2B48606* |   |
| 34 | Current Opt Out Status*opt_out_status* | N | Indicates if the customer is currently opted out of email communication | Provide as a string:*Yes* or *No* |   |
| 35 | Opt Out Date*opt_out_on* | N | If the customer is currently opted out of email communication, the date on which the customer's Opt Out Status was last updated | Provide in the format YYYY-MM-DD:*2005-01-10* |   |
| 36 | Opt Out IP Address*opt_out_ip* | N | The IP Address captured in the log files at the time of the opt out | Provide the IP's numeric string:*172.16.254.1* |   |

### 6. Product Dimension File

* <ins>File Description</ins>: Provides metadata for products shown in the Line Item fact file. Each row of data in Product is expected to correspond to a unique Product ID used to identify the product in Line Item.
* <ins>Required File Naming Convention</ins>: "product_dimension_YYYY-MM-DD.csv", where YYYY-MM-DD is replaced by the year, month, and day the transfer occurred.
* <ins>Required File Format</ins>: Standard UTF-8 encoded CSV format zipped as a single level atomic file. For each file, The YYYY-MM-DD of the atomic zip file must match the YYYY-MM-DD of the individual files.

#### *Data Specification for Product:*


|   | ColumnName<br />*System Field* | Required | FieldDescription | LongExplanationofDataFormat<br />*SampleData* | EvenLongerReferencesetc |
| :- | - | :-: | :- | :- | :- |
| 1 | Product ID*product_id* | Y | The code captured by your POS system to uniquely identify each individual product | Provide as an integer:*24* | Cross-referenced in multiple files; used to join across files Line Item, Discount Summary, and Product |
| 2 | Product Name*product_name* | Y | The name of each product in a check as used in reporting and Bridg's UI (often this is provided as a shortened name) | String value:*CB Burrito* | If this field is left empty, Bridg will default to the Product Description |
| 3 | Product Description*product_description* | N | The full name of each product | String value:*Cheesy Beef Burrito* |   |
| 4 | Product UPC*product_upc* | N | The Universal Product Code for each product as referenced on product barcodes | Provide the 12-digit UPC numeric string:*123456789123* |   |
| 5 | Price Per Unit*unit_price* | Y | The dollar value of a single product or a single unit of a product purchased in a transaction | Provide as a decimal value:*0.65* |   |
| 6 | Currency*currency* | N | Currency of the transaction; will default to USD if no data is provided | Provide in ISO 4217 format (3 digit country + currency code):*USD* |   |
| 7 | Category ID*category_id* | Y | A numerical identifier that represents a specific product category for each product purchased in a check | Provide as an integer that corresponds to one of your product categories:*24* | If product categories exist, this field should be considered Required |
| 8 | Category Name*category_name* | Y | The name of the product category | Provide as a string:*Burritos* |   |
| 9 | Category Description*category_description* | N | The description of the product category | Provide as a string:*Regular priced burritos* |   |
| 10 | Section ID*section_id* | N | An optional identifier that can be used in addition to Category ID to define an additional level of segmentation within your product hierarchy | Provide as an integer that corresponds to one of your product sections:*10* |   |
| 11 | Section Name*section_name* | N | The name of the product section category | Provide as a string:*Wraps* |   |
| 12 | Section Description*section_description* | N | The description of the product section category | Provide as a string:*Includes wraps and burritos* |   |
| 13 | Parent Product ID*parent_id* | N | An optional identifier that can be used to define an additional level of segmentation within your product hierarchy | Provide as an integer that corresponds to one of your parent products :*24* |   |
| 14 | Parent Product Name*parent_product_name* | N | The name of the parent product | Provide as a string:*Bread* |   |
| 15 | Parent Product Description*parent_product_description* | N | The description of the parent product | Provide as a string:*Breads and Grains* |   |
| 16 | Parent UPC*parent_upc* | N | The primary Universal Product Code for the parent product in the case there are nested products in your product hierarchy | Provide the 12-digit UPC numeric string:*123456789123* |   |
| 17 | Location ID*location_id* | N | The ID used in our system to uniquely identify a specific store / location for each tender | Provide as an alphanumeric string:*5812* | Cross-referenced in multiple files; used to join across files Transaction Summary, Tender, Line Item, and Discount Summary |

### 7. Discount Detail Dimension File

* <ins>File Description</ins>: Contains metadata for each type of discount which may appear in the Discount Summary fact file. Each row of data in Discount Detail is expected to correspond to a unique Discount ID used to identify the discount in Discount Summary.
* <ins>Required File Naming Convention</ins>: "discount_detail_dimension_YYYY-MM-DD.csv", where YYYY-MM-DD is replaced by the year, month, and day the transaction occurred.
* <ins>Required File Format</ins>: Standard UTF-8 encoded CSV format zipped as a single level atomic file. For each file, The YYYY-MM-DD of the atomic zip file must match the YYYY-MM-DD of the individual files.

#### *Data Specification for Discount Detail*:


|   | ColumnName<br />*System Field* | Required | FieldDescription | LongExplanationofDataFormat<br />*SampleData* | EvenLongerReferencesetc |
| :- | - | :-: | :- | :- | :- |
| 1 | Discount ID*discount_id* | Y | The code used to uniquely identify each discount applied to a check, as captured by your POS system or entered by the cashier | Provide as a numeric string:*371* | Cross-referenced in multiple files; used to join across files Discount Summary and Discount Detail |
| 2 | Discount Description*discount_description* | Y | The written description of the discount | Provide as a string:*Taco Tuesday Discount* |   |
| 3 | Discount Type*discount_type* | Y | The type of discount applied for a given check, where discount type is one of several predefined values in our system | Provide as a string:*Comp* | • Must be one of the following acceptable values:<br />Sale<br />Refund<br />Revenue Item<br />Comp<br />Promo<br />Discount<br />• Cross-referenced in multiple files; used to join across files Discount Summary and Discount Detail |

### 8. Location Dimension File

* <ins>File Description</ins>: Contains facts for all open and closed store locations where transactions in Transaction Summary may have occurred or consumers identified in Customers may have completed a transaction. This file contains a row of store location facts for each opened or closed store location and any related data for each.
* <ins>Required File Naming Convention</ins>: "location_dimension_YYYY-MM-DD.csv", where YYYY-MM-DD is replaced by the year, month, and day the transfer occurred.
* <ins>Required File Format</ins>: Standard UTF-8 encoded CSV format zipped as a single level atomic file. For each file, The YYYY-MM-DD of the atomic zip file must match the YYYY-MM-DD of the individual files.

#### *Data Specification for Location:*


|   | ColumnName<br />*System Field* | Required | FieldDescription | LongExplanationofDataFormat<br />*SampleData* | EvenLongerReferencesetc |
| :- | - | :-: | :- | :- | :- |
| 1 | Location ID*location_id* | Y | The ID used in our system to uniquely identify a specific store / location for each tender | Provide as alphanumeric string:*5812* | Cross-referenced in multiple files; used to join across files Transaction Summary, Tender, Line Item, and Discount Summary |
| 2 | Location Name*location_name* | Y | The name used by your internal organization to refer to each store / location | Provide as a string or integer:*Michigan Avenue* or *5812* |   |
| 3 | Location Active Status*location_status* | N | An indication as to whether the store / location is currently open for business | Provide as a string:*Open* | If left empty, all locations will be considered as Active |
| 4 | Phone Number*phone* | Y | The primary phone number associated with each store / location | Provide as a string:*312-263-0019* |   |
| 5 | Address Line 1*address_line_1* | Y | The first line in the store / location's business address | Provide as a string:*326 N. Michigan Avenue* |   |
| 6 | Address Line 2*address_line_2* | N | The second line in the store / location's business address | Provide as a string:*Fl 2* | Should be considered as required if there is a unit or suite number associated with the store / location |
| 7 | City*city* | Y | The city where the store / location is located | Provide as a string:*Chicago* |   |
| 8 | State*state* | N | The state where the store / location is located | Provide as a string:*IL* |   |
| 9 | Postal Code*postal_code* | Y | The Zip or Postal Code of the store / location | Provide as a string:*60601* | Not constrained to 5 digits |
| 10 | Country*country* | N | The country where the store / location is located; if empty, will default to USA | Provide as a 3 digit country code:*USA* |   |
| 11 | Location Time Zone*timezone* | Y | The local timezone of the store / location | Provided in the IANA standard format:*America/New_York* |   |
| 12 | Location Open Date*open_date* | N | The date the store / location was opened for business | Provide in ISO 8601 format (YYYY-MM-DD):*2017-12-14* |   |
| 13 | Comp Status*comp_status* | N | Specifies if the store / location is considered as part of your organization's comparable store sales for year over year comparisons | Provide as a string:*Yes* | A status of No is often used to indicate a store / location that has been open for fewer than 12 months |
| 14 | Location Group Name*location_group_name* | N | Specifies which group each individual store / location is assigned to (in organizations that have defined regions or markets) | Provide as a string:*Midwest* |   |
| 15 | Market ID*market_id* | N | An identifier used to represent a specific market each store / location is associated with as defined by your internal organization | Provide as an integer:*9* |   |
| 16 | Market Description*market_description* | N | An additional identifier that can be used to represent another level of market segmentation for each store / location | Provide as a string:*Nielsen DMA* |   |
| 17 | Franchisee*franchisee* | N | The name of the Franchisee who operates a given store / location | Provide as a string:*KFC* |   |