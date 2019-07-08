# ODP based data extraction from S/4HANA via OData 

# Introduction 
For data extraction scenarios from S/4HANA the following requirements have typically to be met: 

* Logical data models abstracting the complexity SAP source tables and corresponding columns
* The data source must be delta-/CDC-enabled to avoid full delta loads 
* Open interfaces and protocols to support the customer demand for cloud-based architectures. 
* Support of frequent intraday data-loads instead of nightly batches
* Supports the extraction of transactional and master data 

The updated ODP-OData feature in SAP NW 7.5 is the enabling technology for achieving the requirements describe above. 
Further references: ([New ODP feature in SAP NetWeaver 7.5]( https://wiki.scn.sap.com/wiki/display/BI/New+ODP+feature+in+SAP+NetWeaver+7.5))

This document will describe the required step to enable the OData-based consumption of changed records in a SAP S/4HANA system. 

# High level scenario description and architecture 
This tutorial will describe a scenario which consists of the following implementation steps 
1)	Extend an existing ABAP CDS view for data extraction
2)	Expose the ABAP CDS view as ODP-enabled ODATA service 
3)	Implement a prototype ODATA client which subscribes to the delta queue. 

![ High level scenario description]( https://github.com/ROBROICH/SAP_ODP_ODATA_CLIENT/blob/master/ODP_SCENARIO.PNG)
To reimplement this scenario for education purposes, the S/4HANA fully activated appliance is recommended to be deployed on SAP CAL ([S/4HANA fully activated appliance](https://blogs.sap.com/2018/12/12/sap-s4hana-fully-activated-appliance-create-your-sap-s4hana-1809-system-in-a-fraction-of-the-usual-setup-time/)). 
From a high-level perspective the S/4HANA implementation consists of the following components:
![ High level architecture]( https://github.com/ROBROICH/SAP_ODP_ODATA_CLIENT/blob/master/HIGH_LEVEL_ARCHITECTURE.PNG)

# ABAP Core Data Services (CDS) based data provisioning (ABAP CDS based ODP context)
The data provisioning mechanism used in this tutorial is typically known as SAP BW extractors or SAP BW business content extractors. The API utilized for providing these data extraction functionalities is referenced as Operational Data Provisioning (ODP)
With S/4HANA and NW 7.5 the ODP technology was updated and has to option to leverage SAP HANA virtual data models(CDS-Views) for data extraction. 
Some fundamentals regarding ABAP CDS based ODP-extraction is the prerequisite for this tutorial and this wiki and blogs are a good starting point:
[Operational Data Provisioning (ODP) and Delta Queue (ODQ)]( https://wiki.scn.sap.com/wiki/pages/viewpage.action?pageId=449284646)

[Data Provisioning Supportability of SAP S/4HANA On-Premise Edition 1709
](https://blogs.sap.com/2016/07/07/data-extraction-supportability-of-sap-s4hana-on-premise-edition-1511-fps02/)

[How to create delta-enabled BW DataSource based ABAP CDS views
]( https://blogs.sap.com/2017/03/17/how-to-create-delta-enabled-bw-datasource-based-abap-cds-views/)

The shown CDS-View customization is based on the blog of Maksim Alyapyshev. In his example the sales document CDS views(I_SalesDocument) gets extended for data extraction. 

# Extending the sales document CDS View “I_SalesDocument”

Based on the example of Maksim’s blog the CDS view had to be slightly adjusted by commenting out some lines which prevented the CDS- view from being activated. 
The column 'LastChangeDateTime' is used as timestamp based identifier for CDC. 

```
@AbapCatalog.sqlViewName: 'ZRB_ISALESDOC_1'
@AbapCatalog.compiler.compareFilter: true
@AccessControl.authorizationCheck: #CHECK
@EndUserText.label: 'CDS for Extraction I_SalesDocument'
@Analytics:{dataCategory:#DIMENSION ,
            dataExtraction.enabled:true}
@Analytics.dataExtraction.delta.byElement.name:'LastChangeDateTime'
@Analytics.dataExtraction.delta.byElement.maxDelayInSeconds: 1800
@VDM.viewType: #BASIC

            

define view ZRB_I_Salesdocument as select from I_SalesDocument {
key SalesDocument,

      //Category
      SDDocumentCategory,
      SalesDocumentType,
      SalesDocumentProcessingType,

      CreationDate,
      CreationTime,
      LastChangeDate,
     //@Semantics.systemDate.lastChangedAt: true
     LastChangeDateTime,

      //Organization
      SalesOrganization,
      DistributionChannel,
      OrganizationDivision,
      SalesGroup,
      SalesOffice,
      
      //Pricing
      //TotalNetAmount,
      TransactionCurrency,
      PricingDate,
      RetailPromotion,
      //PriceDetnExchangeRate,
      SalesDocumentCondition
    
}   
```


Remark: 
Currently only timestamp-based CDC-flags are supported by extraction-enabled CDS-views.
[HANA HASH_SHA256](https://help.sap.com/viewer/4fe29514fd584807ac9f2a04f6754767/2.0.01/en-US/d22ecca9d2951014850492e8c88d498c.html/) functions could be evaluated for delta calculation in addition to timestamps. 

# Generating the ODATA service 

After successfully activating the CDS-view for data-extraction, the OData service must be created.
The required steps for activating the OData service are described in this documentation for the [ODP OData client]( https://help.sap.com/viewer/dd104a87ab9249968e6279e61378ff66/11.0.7/en-US/11853413cf124dde91925284133c007d.html) 

Briefly summarized the following steps are required to create the service:
* Transaction SEGW: Create project

* Redefine model based on ODP extraction: Right click --> “Data model” 
![ Redefine model]( https://github.com/ROBROICH/SAP_ODP_ODATA_CLIENT/blob/master/ODP_CREATE_MODEL_1.png)

* Search for technical name of SQL CDS view
![ Select CDS view]( https://github.com/ROBROICH/SAP_ODP_ODATA_CLIENT/blob/master/ODP_CREATE_MODEL_2.png)

* Finish Wizard 
![Finish wizard]( https://github.com/ROBROICH/SAP_ODP_ODATA_CLIENT/blob/master/ODP_CREATE_MODEL_3.jpg)

* Generate Runtime Object  
![Generate runtime object]( https://github.com/ROBROICH/SAP_ODP_ODATA_CLIENT/blob/master/ODP_CREATE_MODEL_4.png)

* Register service 
![Generate runtime object]( https://github.com/ROBROICH/SAP_ODP_ODATA_CLIENT/blob/master/ODP_CREATE_MODEL_5.png)

[X] Done. Now the service can be tested using the NW Gateway ODATA-client.

# Using the ODP OData service with the gateway OData client.
Via the transaction /IWFND/GW_CLIENT you will be able to test the generated service with the gateway client. 
The tests for the prototype will implement the following cases:
1)	Check if the OData client is already subscribed to the ODP queue 
2)	Subscribe to the ODP queue and initialize the delta processing 
3)	Get delta links
4)	Update sales document in VA02 
5)	Fetch updated record from delta queue 

* Check if the OData client is already subscribed to the ODP queue 
```
URL: 
/sap/opu/odata/sap/ZRB_ODP_ODATA_SRV_01/SubscribedToAttrOfZRB_ISALESDOC
Result: 
SubscribedFlag:false
```

Subscribe to the ODP queue and initialize the delta processing 
![Generate delta queue]( https://github.com/ROBROICH/SAP_ODP_ODATA_CLIENT/blob/master/ODP_CREATE_MODEL_6.png)

* Get the delta links 
```
URL: 
/sap/opu/odata/sap/ZRB_ODP_ODATA_SRV_01/DeltaLinksOfAttrOfZRB_ISALESDOC
Result: 
DeltaLinksOfAttrOfZRB_ISALESDOC('D20190708134907_000128000')/ChangesAfter
```
![Get the delta links]( https://github.com/ROBROICH/SAP_ODP_ODATA_CLIENT/blob/master/ODP_CREATE_MODEL_7.png)

* Update sales document in VA02 
![Update document in VA02]( https://github.com/ROBROICH/SAP_ODP_ODATA_CLIENT/blob/master/ODP_CREATE_MODEL_8.png)

* Fetch updated record from delta queue 
```
URL: 
/sap/opu/odata/sap/ZRB_ODP_ODATA_SRV_01/DeltaLinksOfAttrOfZRB_ISALESDOC('D20190708134907_000128000')/ChangesAfter
```
![Updated record in ODP delta queue]( https://github.com/ROBROICH/SAP_ODP_ODATA_CLIENT/blob/master/ODP_CREATE_MODEL_9.png)



