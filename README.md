# SAP Operational Data Provisioning(ODP) OData Client with S/4HANA: "Hello World"
For data integration scenarios based on SAP S4/HANA source systems, the following requirements are mainly relevant for designing integration architectures:

* Logical data models abstracting the complexity SAP source tables and corresponding columns
* The data source must be delta-/CDC-enabled to avoid full delta loads 
* Open interfaces and protocols to support the customer demand for cloud-based architectures. 
* Support of frequent intraday data-loads instead of nightly batches
* Support of transactional and master data 


The updated ODP feature in SAP NW 7.5 is the main technology for achieving the requirements describe above. Further information: ([New ODP feature in SAP NetWeaver 7.5]( https://wiki.scn.sap.com/wiki/display/BI/New+ODP+feature+in+SAP+NetWeaver+7.5))

# High level scenario description and architecture 
This tutorial will describe a scenario which mainly consists of the following implementation steps 
1)	Extend an existing ABAP CDS view for data extraction
2)	Expose the ABP CDS view as ODP-enabled ODATA service 
3)	Implement a prototype ODATA client which subscribes to the delta queue. 

![ High level scenario description]( https://github.com/ROBROICH/SAP_ODP_ODATA_CLIENT/blob/master/ODP_SCENARIO.PNG)
To reimplement this scenario for education purposes, the S/4HANA fully activated appliance is recommended to be deployed on SAP CAL. 
([S/4HANA fully activated appliance](https://blogs.sap.com/2018/12/12/sap-s4hana-fully-activated-appliance-create-your-sap-s4hana-1809-system-in-a-fraction-of-the-usual-setup-time/))
Technically a S/4HANA system is the main building block for this scenario. 
From a high-level perspective the S/4HANA implementation consists of the following main building blocks:
![ High level architecture]( https://github.com/ROBROICH/SAP_ODP_ODATA_CLIENT/blob/master/HIGH_LEVEL_ARCHITECTURE.PNG)

# ABAP Core Data Services (CDS) based data provisioning (ABAP CDS based ODP context)
The data provisioning mechanism used in this tutorial is typically known as SAP BW extractors or SAP BW business content extractors. With S/4HANA the extraction technology was updated and utilizes SAP HANA virtual data models for data extraction. 
Some fundamentals regarding ABAP CDS based ODP-extraction is the prerequisite for this tutorial and this wiki and blogs are a good starting point:

[Operational Data Provisioning (ODP) and Delta Queue (ODQ)]( https://wiki.scn.sap.com/wiki/pages/viewpage.action?pageId=449284646)

[Data Provisioning Supportability of SAP S/4HANA On-Premise Edition 1709
](https://blogs.sap.com/2016/07/07/data-extraction-supportability-of-sap-s4hana-on-premise-edition-1511-fps02/)

[How to create delta-enabled BW DataSource based ABAP CDS views
]( https://blogs.sap.com/2017/03/17/how-to-create-delta-enabled-bw-datasource-based-abap-cds-views/)

The CDS-View customization is based on the blog of Maksim Alyapyshev. In this example sales document CDS views gets extended for data extraction. 

#Extending the sales document CDS View “I_SalesDocument”

Based on the example of Maksim’s blog the CDS view had to be slightly adjusted by commenting out some lines which prevented the CDS- view from being activated. 

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











