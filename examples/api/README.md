
# CPR+ sends the Referral Approval to the Supply Agency
CPR+ will received a [Referral Request payload](https://github.com/AlayaCare/alayamarket-optioncare-csi/blob/main/examples/payloads/new_referral_request_payload.json) from a supplier agency requesting authorization to provide a service for a client. Once the request is approved, there are several ways that CPR+ can send the Referral Approval back to the supply agency. These are described in detail below. 

## Option 1: CPR+ → ACCloud CSI (HQ branch) → Marketplace -> Supply Agency
_This is the recommended approach._

In this option, CSI will sync clients, services, and authorizations from CPR+ to ACCloud CSI (HQ branch), before sending a Referral Approval from ACCloud via Marketplace to the supply agency. 

* **Host:** `ACCLOUD_URL`
  * DEV: `https://csidev.uat.alayacare.com`  
  * QA: `https://csiqa.uat.alayacare.com` (not available yet)
  * UAT: `https://csi.uat.alayacare.com`     
  * PROD: `https://csi.alayacare.com` (not available yet)
* **Authentication:** Basic authentication using API keys 

The options below are based on entities that exist on `https://csi.uat.alayacare.com`. 

### Step 1: Create the Client

* **API reference:** [client-api-external](https://app.swaggerhub.com/apis/AlayaCare/client-api-external/1.0.6#/Clients/post_clients) 

* **URL:** `POST $ACCLOUD_URL/ext/api/v2/patients/clients`

* **Example payload:**
```json
{
   "demographics":{
      "first_name":"Frank",
      "last_name":"Smith",
      "address":"3000 Lakeside Dr",
      "city":"Bannockburn",
      "state":"IL",
      "zip":"60015",
      "country":"US",
      "birthday":"1999-01-31",
      "email":"john.smith@example.com",
      "phone_main":"515-223-4455",
      "gender":"M"
   },
   "external_id":"client_external_id_1",
   "branch_id":1000,
   "language":"en",
   "groups":[
      {
         "id":1008
      }
   ],
   "timezone":"America/Chicago"
}
```

* **Notes:**
  * The `external_id` is the external client ID, and optionally can be set to the ID of the client in CPR+
  * The `groups` must be set to include the ID of the group with name `CSI`
    * Groups can be fetched from `$ACCLOUD_URL/ext/api/v2/patients/groups`, see to the API reference [here](https://app.swaggerhub.com/apis/AlayaCare/client-api-external/1.0.6#/Groups/get_groups) 
    * This will ensure HQ coordinators only see clients in their branch, and do not see clients in the agency branches
  * The response returned when creating a client will include the internal client ID
  * An existing client record can be retrieved or updated using the internal client ID or the external client ID, see the API reference [here](https://app.swaggerhub.com/apis/AlayaCare/client-api-external/1.0.6#/Clients) 

### Step 2: Create the Service

* **API reference:** [service-api-external](https://app.swaggerhub.com/apis/AlayaCare/services-api-external/1.0.1#/Services/post_services) 

* **URL:** `POST $ACCLOUD_URL/ext/api/v2/scheduler/services`

* **Example payload:**
```json
{
   "service_id":"service_external_id_1",
   "name":"Skilled Nursing",
   "client":{
      "alayacare_client_id":1060,
      "client_id":"client_external_id_1"
   },
   "start_date":"2023-04-01",
   "projected_end_date":"2023-06-30",
   "funding_methodology":"single_funder",
   "guid_funder_breakdowns": [
      {
        "funder_id":9, 
        "is_primary": true, 
        "percentage": 100
      }
   ],
   "service_code_id":48,
   "service_instructions":"Please use buzz code 203",
   "use_client_address":true, 
   "profile": {
      "target_agency": "Gem City Home Care"
   }
}
```

* **Notes:**
  * The `service_id` is the external service ID, and should be set to the `request_id` from the [referral request payload](https://github.com/AlayaCare/alayamarket-optioncare-csi/blob/main/examples/payloads/new_referral_request_payload.json#L2)
  * The `client` can reference only one of `alayacare_client_id` or `client_id`
    * The `alayacare_client_id` is the external client ID provided in Step 1
    * The `client_id` is the internal client ID returned in Step 1  
  * The payors/funders can be fetched from `$ACCLOUD_URL/ext/api/v2/accounting/funders`, see the API reference [here](https://app.swaggerhub.com/apis/AlayaCare/accounting-api-external/1.0.6#/Funders/get_funders)
  * The service codes can be fetched from `$ACCLOUD_URL/ext/api/v2/scheduler/service_codes`, see the API reference [here](https://app.swaggerhub.com/apis/AlayaCare/services-api-external/1.0.1#/Service%20Codes/get_service_codes) 
  * The `start_date` and `projected_end_date` are optional. These are only informative and are not used when sending referral approvals to supplier agencies. 


### Step 3: Create the Authorization

* **API reference:** External API specifications are not yet available for this endpoint. 
A snapshot of the internal API specifications can be found [here](https://github.com/AlayaCare/alayamarket-optioncare-csi/blob/main/specs/authorizations/internal.authorizations.spec.yaml). 

* **URL:** `POST $ACCLOUD_URL/api/v1/scheduler/authorizations`

* **Example payload:**
```json
{
   "client_id":1060,
   "service_ids":[
      21
   ],
   "payor_id":9,
   "start_date":"2023-04-01",
   "end_date":"2023-04-30",
   "payor_id_type":"FUNDER",
   "methodology":"PAYOR_SERVICE",
   "notes":"",
   "authorization_number": "123-456", 
   "member_number": "123-456",  
   "program_id": "123-456", 
   "rule_type":"HOURS",
   "rule_daily":300,
   "rule_weekly":6000,
   "rule_monthly":null,
   "rule_period":null,
   "rule_monday":null,
   "rule_tuesday":null,
   "rule_wednesday":null,
   "rule_thursday":null,
   "rule_friday":null,
   "rule_saturday":null,
   "rule_sunday":null,
   "first_day_of_week":6, 
   "case_manager_name": "Jane Smith", 
   "case_manager_phone": "515-111-2233", 
   "case_manager_fax": null, 
   "case_manager_email": "jane.smith@email.com"
}
```

* **Notes:**
  * Payors/Funders can be fetched from `$ACCLOUD_URL/ext/api/v2/accounting/funders`
  * The `client_id` is the internal client ID returned when creating a client in Step 1
  * The `service_ids` is the internal service ID returned when creating a service in Step 2 
  * The `payor_id` is the internal payor/funder ID, and can be fetched from `$ACCLOUD_URL/ext/api/v2/accounting/funders`, see the API reference [here](https://app.swaggerhub.com/apis/AlayaCare/accounting-api-external/1.0.6#/Funders/get_funders)
  * The `notes` are not currently being displayed in the supply agency referral details. This will be supported soon. 
  * `authorization_number`, `member_number` and `program_id` are optional fields that can be used to identify the authorization. 
  * The `case_manager_*` fields are optional. This will be stored on the HQ branch only and will not be sent to the supply agency. 
    * To send a case manager to the supply agency, see Step 4. 
  * The response returned when creating an authorization will include the internal authorization ID

#### Step 3.1: Update the Authorization

* **URL:** `PUT $ACCLOUD_URL/api/v1/scheduler/authorizations/{authorization_id}`

* **Example payload:**
```json
{
   "start_date":"2023-04-01",
   "end_date":"2023-04-30",
   "notes":"",
   "authorization_number": "123-456", 
   "member_number": "123-456",  
   "program_id": "123-456", 
   "rule_type":"HOURS",
   "case_manager_name": "Jane Smith", 
   "case_manager_phone": "515-111-2233", 
   "case_manager_fax": null, 
   "case_manager_email": "jane.smith@email.com"
}
```

### Step 4: Send the Referral Approval

* **API reference:** External API specifications are not yet available for this endpoint. 
A snapshot of the internal API specifications can be found [here](https://github.com/AlayaCare/alayamarket-optioncare-csi/blob/main/specs/referral_approval/internal.referral_approval.spec.yaml). 

* **URL:** `POST $ACCLOUD_URL/api/v1/alayamarket/outbox/referrals/from_service/{service_id}`

* **Example payload:**
```json
{
   "authorization_id":8, 
   "supply_persona_id":"91c79089-a3ca-4cca-acf4-52fc1f1302c6",
   "referral_type":"service",
   "care_type":"nursing",
   "start_at":"2023-04-01T05:00:00+00:00",
   "end_at":"2023-04-30T05:00:00+00:00", 
   "comment": "", 
   "case_manager": {
     "name": "Jane Smith", 
     "phone_number": "515-111-2233", 
     "fax_number": null, 
     "email": "jane.smith@email.com"
   }
}
```

* **Notes:**
  * The `service_id` is the internal service ID returned when creating a service in Step 2 
  * The `authorization_id` is the internal authorization ID and is returned when creating an authorization in Step 3
  * The `supply_persona_id` must be set to the `alayamarket_org_id` from the [referral request payload](https://github.com/AlayaCare/alayamarket-optioncare-csi/blob/main/examples/payloads/new_referral_request_payload.json#L7)  
    * You can get a list of your suppliers from `$ACCLOUD_URL/api/v1/alayamarket/outbox/organizations/trusted_network`
  * If an `authorization_id` is provided, the `start_date` and `end_date` of the authorization must match the date components of the referral approval `start_at` and `end_at`. 
  * The `case_manager` from the authorization in Step 3 will not be automatically sent to the supply agency, and should be specified as required during this step. 

#### Step 4.1: Update the Referral Approval

* **URL:** `POST $ACCLOUD_URL/api/v1/alayamarket/outbox/referrals/from_service/{service_id}/update/{referral_id}`

* **Example payload:**
```json
{
   "authorization_id":8, 
   "supply_persona_id":"91c79089-a3ca-4cca-acf4-52fc1f1302c6",
   "start_at":"2023-04-01T05:00:00+00:00",
   "end_at":"2023-04-30T05:00:00+00:00", 
   "comment": "", 
   "case_manager": {
     "name": "Jane Smith", 
     "phone_number": "515-111-2233", 
     "fax_number": null, 
     "email": "jane.smith@email.com"
   }, 
   "revision": {
      "reasons": [
         "Updated authorization end date"
      ], 
      "comment": ""
   }
}
```
* **Notes:**
  * The `referral_id` is the referral ID returned when creating a referral approval in Step 4. 
  * The updated referral will include any updates to the client, service, and authorization.

## Option 2: CPR+ → Marketplace -> Supply Agency


In this option, CSI will send the Referral Approval from CPR+ via Marketplace to the supply agency. This will skip storing clients, services, and authorizations in ACCloud CSI (HQ). This also means that referral approvals cannot be sent from ACCloud CSI (HQ).

* **Host:** `MARKETPLACE_URL`:
  * https://api.sandbox.alayamarket.com (sandboxed environment for testing)
  * https://api.prod.alayamarket.com (production environment)

* **Authentication:** Bearer (token) authentication 
  * For more details see the [Marketplace user guide](https://alayacare.github.io/alayamarket-external-docs/docs/user_guide/)

### Step 1: Send the Referral Approval

* **API reference:** [referral-outbox-api](https://alayacare.github.io/alayamarket-external-docs/docs/offers/openapi.offers#/referral%20outbox/post_v1_outbox_referrals)

* **URL:** `$MARKETPLACE_URL/offers/v1/outbox/referrals`

* **Example payload:**
```json
{
   "referral":{
      "care_type":"nursing",
      "referral_type":"service"
   },
   "supply_persona_id":"91c79089-a3ca-4cca-acf4-52fc1f1302c6",
   "alayacare":{
      "authorizations":{
         "authorization_number":"111-111",
         "authorization_status":"AuthorizationStatus.PROCESSED",
         "first_day_of_week":6,
         "member_number":"123456",
         "methodology":"PAYOR_SERVICE",
         "notes":"",
         "program_id":"ABCDEF",
         "rule_daily":300,
         "rule_friday":null,
         "rule_monday":null,
         "rule_monthly":null,
         "rule_period":null,
         "rule_saturday":null,
         "rule_sunday":null,
         "rule_thursday":null,
         "rule_tuesday":null,
         "rule_type":"HOURS",
         "rule_wednesday":null,
         "rule_weekly":6000
      }
   },
   "price":{
      "rate":30,
      "unit":"per_visit"
   },
   "client":{
      "address":{
         "address":"3000 Lakeside Dr",
         "city":"Bannockburn",
         "state":"IL",
         "zip":"60015",
         "country":"US",
         "latitude":42.201981,
         "longitude":-87.882164
      },
      "demographics":{
         "first_name":"Frank",
         "last_name":"Smith",
         "date_of_birth":"1999-01-31",
         "email":"john.smith@example.com",
         "gender":"M",
         "language":"en"
      },
      "outbox_id":"1060"
   },
   "service":{
      "outbox_id":"21",
      "address":{
         "address":"3000 Lakeside Dr",
         "city":"Bannockburn",
         "state":"IL",
         "zip":"60015",
         "country":"US",
         "latitude":42.201981,
         "longitude":-87.882164
      },
      "name":"Skilled Nursing",
      "start_at":"2023-04-01T15:04:15.753Z",
      "end_at":"2023-06-30T15:04:15.753Z",
      "instructions":"Please use buzz code 203"
   }
}
```

* **Notes:**
  * The `supply_persona_id` will be provided in the referral request payload under `alayamarket_org_id`
  * The `outbox_id` should reference the entity ID in the originating system
  * The authorizations are expected to match the ACCloud format for authorizations and can be fetched from `$ACCLOUD_URL/api/v1/scheduler/authorizations/{authorization_id}`
