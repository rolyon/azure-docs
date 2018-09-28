---
title: Understand how the reservation discount is applied to Azure Cosmos DB | Microsoft Docs
description: Learn how reservation discount is applied to provisioned throughput (RU/s) in Azure Cosmos DB.
services: cosmos-db
author: rimman
manager: kfile

ms.service: cosmos-db
ms.topic: conceptual
ms.date: 09/24/2018
ms.author: cwatson
ms.reviewer: sngun
---

# Understand how the reservation discount is applied to Azure Cosmos DB

After you buy an Azure Cosmos DB reserved capacity, the reservation discount is automatically applied to Azure Cosmos DB resources that match the attributes and quantity of the reservation. A reservation covers the throughput provisioned for Azure Cosmos DB resources and it doesn’t cover software, networking, storage, or pre-defined container charges.

## Reservation discount applied to Azure Cosmos DB accounts

A reservation discount is applied to [provisioned throughput](../cosmos-db/request-units.md) in terms of request units per second(RU/s) on an hour-by-hour basis. For Azure Cosmos DB resources that don't run the full hour, the reservation discount is automatically applied to other Cosmos DB resources that match the reservation attributes. The discount can apply to Azure Cosmos DB resources that are running concurrently. If you don't have Cosmos DB resources that run for the full hour and that match the reservation attributes, you don't get the full benefit of the reservation discount for that hour.

* The discounts are tiered which means reservations with higher request units provide higher discounts.  
* The reservation purchase will apply discounts to all regions in the ratio equivalent to the regional on-demand pricing. For reservation discount ratios in each region see [reservation discount per region](#reservation-discount-per-region) section of this article.

## Reservation discount per region
Reservation discount is applied to Azure Cosmos DB throughput costs on an hour-by-hour basis at either the single subscription or enrolled/account scope. The reservation discount applies to meter usage in different regions in the following ratio:

|Meter Description  |Region |Ratio  |
|---------|---------|---------|
|Azure Cosmos DB - 100 RU/s/Hour - AP Southeast  |  AP Southeast    |   1      |
|Azure Cosmos DB - 100 RU/s/Hour - AP East |   AP East   |    1     |
|Azure Cosmos DB - 100 RU/s/Hour - EU North|  EU North       |    1     |
|Azure Cosmos DB - 100 RU/s/Hour - KR South|    KR South     |     1    |
|Azure Cosmos DB - 100 RU/s/Hour - EU West|    EU West     |      1   |
|Azure Cosmos DB - 100 RU/s/Hour - KR Central|   KR Central    |       1  |
|Azure Cosmos DB - 100 RU/s/Hour - UK South|   UK South      |     1    |
|Azure Cosmos DB - 100 RU/s/Hour - UK West|   UK West      |    1     |
|Azure Cosmos DB - 100 RU/s/Hour - UK North |   UK North    |     1    |
|Azure Cosmos DB - 100 RU/s/Hour - UK South 2|   UK South 2      |     1    |
|Azure Cosmos DB - 100 RU/s/Hour - US East 2|  US East 2     |     1    |
|Azure Cosmos DB - 100 RU/s/Hour - US North Central|   US North Central      |     1    |
|Azure Cosmos DB - 100 RU/s/Hour - US West|   US West      |     1    |
|Azure Cosmos DB - 100 RU/s/Hour - US Central| US Central        |     1    |
|Azure Cosmos DB - 100 RU/s/Hour - US West 2|   US West 2      |      1   |
|Azure Cosmos DB - 100 RU/s/Hour - US West Central|   US West Central      |       1  |
|Azure Cosmos DB - 100 RU/s/Hour - US East|   US East      |  1       |
|Azure Cosmos DB - 100 RU/s/Hour - SA North|     SA North    |   1      |
|Azure Cosmos DB - 100 RU/s/Hour - SA West |    SA West      |    1     |
|Azure Cosmos DB - 100 RU/s/Hour - IN South|    IN South     |    1.0375    |
|Azure Cosmos DB - 100 RU/s/Hour - CA East|   CA East      |    1.1      |
|Azure Cosmos DB - 100 RU/s/Hour - JA East|   JA East      |    1.125     |
|Azure Cosmos DB - 100 RU/s/Hour - JA West|     JA West    |   1.125       |
|Azure Cosmos DB - 100 RU/s/Hour - IN West|     IN West    |    1.1375     |
|Azure Cosmos DB - 100 RU/s/Hour - IN Central|    IN Central     |  1.1375       |
|Azure Cosmos DB - 100 RU/s/Hour - AU East|     AU East    |   1.15       |
|Azure Cosmos DB - 100 RU/s/Hour - CA Central|  CA Central       |   1.2       |
|Azure Cosmos DB - 100 RU/s/Hour - FR Central|   FR Central      |    1.25      |
|Azure Cosmos DB - 100 RU/s/Hour - BR South|  BR South       |   1.5      |
|Azure Cosmos DB - 100 RU/s/Hour - AU Central|   AU Central      |   1.5      |
|Azure Cosmos DB - 100 RU/s/Hour - AU Central 2| AU Central 2        |    1.5     |
|Azure Cosmos DB - 100 RU/s/Hour - FR South|    FR South     |    1.625     |

## Scenarios that show how the reservation discount is applied

Consider the following requirements for a reservation:

* Required throughput: 50,000 RU/s  
* Regions used: 2. 

In this case, your total on-demand charges are for 500 quantity of 100 RU/s meter in these two regions, for a total RU/s consumption of 100,000 every hour. 

**Scenario 1**

For example, if you need Azure Cosmos DB deployments in "US North Central" and "US West" regions, and if each region has a throughput consumption of 50,000 RU/s. A reservation purchase of 100,000 RU/s would completely balance your on-demand charges.

The discount covered by a reservation is computed as (throughput consumption * reservation_discount_ratio_for_that_region). For "US North Central" and "US West" regions, the reservation discount ratio is "1". So, the total discounted RU/s are 100,000 RU/s (this value is computed as: 50,000 * 1 + 50,000 * 1 = 100,000 RU/s), and you don't have to pay any additional charges at the regular pay-as-you-go rates. 

|Meter description | Region |Throughput consumption (RU/s) |Reservation discount applied to RU/s |
|---------|---------|---------|---------|
|Azure Cosmos DB - 100 RU/s/Hour - US North Central  |   US North Central  | 50,000  | 50,000  |
|Azure Cosmos DB - 100 RU/s/Hour - US West  |  US West   |  50,000  |  50,000 |

**Scenario 2**

For example, if you need Azure Cosmos DB deployments in "AU Central 2" and "FR South" regions, and if each region has a throughput consumption of 50,000 RU/s. A reservation purchase of 100,000 RU/s would be applicable as follows (assuming that AU Central 2 usage was discounted first):

|Meter description | Region |Throughput consumption (RU/s) |Reservation discount applied to RU/s |
|---------|---------|---------|---------|
|Azure Cosmos DB - 100 RU/s/Hour - AU Central 2  |  AU Central 2   |  50,000  |  50,000   |
|Azure Cosmos DB - 100 RU/s/Hour - FR South  |  FR South   |  50,000 |  15,384  |

50,000 units of usage in "AU Central 2" region corresponds to 75,000 RU/s of billable usage (or normalized usage). This value is computed as (throughput consumption * reservation_discount_ratio_for_that_region) which equals to 75,000 RU/s (this value is computed as: 50,000 * 1.5 = 75,000 RU/s) of billable or normalized usage. 

100,000 RU/s of reservation purchase would offset the 75,000 RU/s in "AU Central 2" and leaves 25,000 RU/s to the "FR South" region. From the remaining 25,000 RU/s, a reservation discount of 15,384 RU/s (this value is computed as: 25,000 / 1.625 = 15,384 RU/s) is applied to "FR South" region. The remaining 34,616 RU/s in "FR South" region are charged at the normal pay-as-you-go rates. 

The Azure billing system will assign the Reservation billing benefit to the first instance that is processed which matches the Reservation configuration (for example, AU Central 2 in this case).

To understand and view the application of your Azure reservations in billing usage reports, see [Understand Azure reservation usage](../billing/billing-understand-reserved-instance-usage-ea.md).

## Next steps

To learn more about Azure Reservations, see the following articles:

* [What are Azure reservations?](../billing/billing-save-compute-costs-reservations.md)  
* [Prepay for Azure Cosmos DB resources with Azure Cosmos DB reserved capacity](../cosmos-db/cosmos-db-reserved-capacity.md)  
* [Prepay for SQL Database compute resources with Azure SQL Database reserved capacity](../sql-database/sql-database-reserved-capacity.md)  
* [Manage Azure Reservations](../billing/billing-manage-reserved-vm-instance.md)  
* [Understand reservation usage for your Pay-As-You-Go subscription](../billing/billing-understand-reserved-instance-usage.md)  
* [Understand reservation usage for your Enterprise enrollment](../billing/billing-understand-reserved-instance-usage-ea.md)  
* [Understand reservation usage for CSP subscriptions](https://docs.microsoft.com/partner-center/azure-reservations)

## Need help? Contact support

If you still have further questions, [contact support](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) to get your issue resolved quickly.

