# SAP Operational Data Provisioning(ODP) OData Client with S/4HANA: "Hello World"
For data integration scenarios based on SAP S4/HANA source systems, the following requirements are mainly relevant for designing integration architectures:

* Logical data models abstracting the complexity SAP source tables and corresponding columns
* The data source must be delta-/CDC-enabled to avoid full delta loads 
* Open interfaces and protocols to support the customer demand for cloud-based architectures. 
* Support of frequent intraday data-loads instead of nightly batches

The updated ODP feature in SAP NW 7.5 is the main technology for achieving the requirements describe above. Further information: ([New ODP feature in SAP NetWeaver 7.5]( https://wiki.scn.sap.com/wiki/display/BI/New+ODP+feature+in+SAP+NetWeaver+7.5))

# High level scenario description and architecture 
This tutorial will describe a scenario which mainly consists of the following implementation steps 
1)	Extend an existing ABAP CDS view for data extraction
2)	Expose the ABP CDS view as ODP-enabled ODATA service 
3)	Implement a prototype ODATA client which subscribes to the delta queue. 

![ High level scenario description]( https://github.com/ROBROICH/SAP_ODP_ODATA_CLIENT/blob/master/ODP_SCENARIO.PNG)
To reimplement this scenario for education purposes, the S/4HANA fully activated appliance is recommended to be deployed on SAP CAL. 
(S/4HANA fully activated appliance]( https://blogs.sap.com/2017/12/14/sap-s4hana-1709-fully-activated-appliance-create-your-sap-s4hana-1709-system-in-a-fraction-of-the-usual-setup-time/))

Technically a S/4HANA system is the main building block for this scenario. 
From a high-level perspective the S/4HANA implementation consists of the following main building blocks:
![ High level architecture]( https://github.com/ROBROICH/SAP_ODP_ODATA_CLIENT/blob/master/HIGH_LEVEL_ARCHITECTURE.PNG)


