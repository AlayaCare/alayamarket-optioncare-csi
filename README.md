# AlayaCare Marketplace - OptionCare CSI Integration 

OptionCare CSI is a specialty pharmacy working with many Agencies supplying care in the infusion vertical. In general, the Agencies will request authorization from CSI to provide care to the client (referenced as a "Referral Request"). CSI then needs to review these requests before sending the approval back to the Agency (referenced as a "Referral" or "Referral Approval"). 

This repository will contain technical documentation for the integration 
between Supply Agencies using AlayaCare and OptionCare CSI using CPR+. 

## AlayaCare Environments

The AlayaCare instance that is configured to support and test these flows can be found at https://csi.uat.alayacare.com/. 

This is configured as a Multi-Office instance. The "HQ" parent branch provides CSI with global access and visibility to all the agencies, and has access to the Marketplace Outbox where Referrals and Offers can be sent to the agencies. Each agency will have a corresponding branch underneath the "HQ" branch where their visiblity is limited to their clients and associated data only. Agencies can receive Offers and Referrals from CSI in their specific branch. 


## New Client/Service Workflow 

### Step 1. Supply Agency submits a Referral Request form
Custom forms are used to capture data about the client and service that the supply agency is requesting authorization to provide. 

The form template can be viewed in the application [here](https://csi.uat.alayacare.com/#/system-settings/forms), and it's definition can be found [here](https://csi.uat.alayacare.com/api/v1/tasks/forms20/forms/11). 

When a form is filled and submitted, AlayaConnector will gather the form submission data and other relevant metadata to send to CPR+. An example of the payload sent to CPR+ can be found in [new_referral_request_payload.json](examples/payloads/new_referral_request_payload.json). The OpenAPI specification for this data is defined in [referral_request.spec.yaml](specs/referral_request/referral_request.spec.yaml). 

**Note that the `field_id` value in the payload is a primary key and will vary on each environment.**



### Step 2. CPR+ processes the Referral Request
Once the Referral Request data has been received by CPR+, it will be reviewed. CPR+ will keep a record of the client, service, and approved authorizations for the service. Once approved, CPR+ will send a Referral Approval to the supply agency so they can beging providing care. 

### Step 3. CPR+ sends the Referral Approval to the Supply Agency
There are several ways that CPR+ can send the Referral Approval to the supply agency once the request has been approved. For more details and examples, see [examples/api](examples/api/README.md).
