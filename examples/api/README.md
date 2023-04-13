
# CPR+ sends the Referral Approval to the Supply Agency
There are several ways that CPR+ can send the Referral Approval to the supply agency once the request has been approved. For more details, see the section below

## Option 1: CPR+ → ACCloud CSI (HQ branch) → Marketplace -> Supply Agency

In this option, CSI will sync clients, services, and authorizations from CPR+ to ACCloud CSI (HQ branch), before sending a Referral Approval from ACCloud via Marketplace to the supply agency. 

* **Host:** `ACCLOUD_URL = https://csi.uat.alayacare.com`  
* **Authentication:** Basic authentication using API keys 

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
  * Groups can be fetched from `$ACCLOUD_URL/ext/api/v2/patients/groups`


### Step 2: Create the Service

* **API reference:** [service-api-external](https://app.swaggerhub.com/apis/AlayaCare/services-api-external/1.0.1#/Services/post_services) 

* **URL:** `$ACCLOUD_URL/ext/api/v2/scheduler/services`

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
   "use_client_address":true
}
```

* **Notes:**
  * Service codes can be fetched from `$ACCLOUD_URL/ext/api/v2/scheduler/service_codes`
  * Skills can be fetched from `$ACCLOUD_URL/ext/api/v2/employees/skills`
  * Payors/Funders can be fetched from `$ACCLOUD_URL/ext/api/v2/accounting/funders`
  * Client can be referenced using one of `client_id` OR `alayacare_client_id`
    * The `client_id` is returned when a client is created

### Step 3: Create the Authorization

* **API reference:** External API specifications are not yet available for this endpoint 

* **URL:** `$ACCLOUD_URL/api/v1/scheduler/authorizations`

* **Example payload:**
```json
{
   "client_id":1060,
   "service_ids":[
      21
   ],   
   "payor_id":9,
   "payor_id_type":"FUNDER",
   "start_date":"2023-04-01",
   "end_date":"2023-04-30",
   "authorization_number":"111-111",
   "member_number":"123456",
   "program_id":"ABCDEF",
   "case_manager_name":"Frank Winter",
   "case_manager_email":"",
   "case_manager_phone":"555-5555",
   "case_manager_fax":"",
   "notes":"",
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
   "methodology":"PAYOR_SERVICE",
   "state_type_id":null,
   "first_day_of_week":6
}
```

* **Notes:**
  * Payors/Funders can be fetched from `$ACCLOUD_URL/ext/api/v2/accounting/funders`
  * The `client_id` and `service_id` are returned when creating a client and service, respectively


### Step 4: Send the Referral Approval

* **API reference:** External API specifications are not yet available for this endpoint 

* **URL:** `$ACCLOUD_URL/api/v1/alayamarket/outbox/referrals/from_service/{service_id}`

* **Example payload:**
```json
{
   "referral_type":"service",
   "care_type":"nursing",
   "start_at":"2023-04-01T05:00:00+00:00",
   "end_at":"2023-04-30T05:00:00+00:00",
   "supply_persona_id":"91c79089-a3ca-4cca-acf4-52fc1f1302c6",
   "authorization_id":8
}
```

* **Notes:**
  * The `supply_persona_id` will be provided in the referral request payload under `alayamarket_org_id`
  * Authorizations can be fetched from `$ACCLOUD_URL/api/v1/scheduler/authorizations`
    * The `service_id` and `authorization_id` are returned when creating a service and authorization, respectively


## Option 2: CPR+ → Marketplace -> Supply Agency

In this option, CSI will send the Referral Approval from CPR+ via Marketplace to the supply agency. 

* **Host:** `MARKETPLACE_URL = https://api.sandbox.alayamarket.com`  

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
