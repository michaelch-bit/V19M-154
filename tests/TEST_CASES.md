# Test Cases – V19M-154: MW OCI UAT Reservation Schema Fix

## Background

When reserving PDA stock receipt for web sale backorders in a non-prod environment via Middleware
(MW), the MW transmission was failing because `FULFILMENT_LOCATION` was not included in the OCI UAT
docket schema (`TRANS_LINE`).  After the element was added to the TST environment schema the
transmission succeeded.  The same fix was then applied to the OCI UAT environment schema
(`schema/mw-oci-uat-docket.xsd`).

---

## Test Case 1 – Reserve PDA stock receipt for web sale backorder (with FULFILMENT_LOCATION)

| Field | Value |
|---|---|
| **Test ID** | TC-V19M154-001 |
| **Environment** | MW OCI UAT (non-prod) |
| **Pre-condition** | A web sale backorder exists and PDA stock has been received against it |
| **Related fixture** | `tests/fixtures/reservation-with-fulfilment-location.xml` |

### Steps

1. In POS, complete a PDA stock receipt against the web sale backorder.
2. POS creates a reservation docket payload and submits it to MW for transmission.
3. MW validates the payload against the OCI UAT docket schema.

### Expected result

The MW transmission is accepted.  The `FULFILMENT_LOCATION` element inside `TRANS_LINE` passes
schema validation without error.

### Previous (failing) result

MW rejected the payload with:

```
Element not allowed: FULFILMENT_LOCATION@http://thegoodguys.com.au/pos/docket
  in element TRANS_LINE@http://thegoodguys.com.au/pos/docket
```

### Sample payload

```xml
<DOCKET xmlns="http://thegoodguys.com.au/pos/docket">
  <DKT_NBR>D1210421147</DKT_NBR>
  <POS_LOCN_NBR>121</POS_LOCN_NBR>
  <DOC_REF1>S1210123436</DOC_REF1>
  <TRANS_TIMESTAMP>2026-04-13T13:08:33+10:00</TRANS_TIMESTAMP>
  <TRANS_LINES>
    <TRANS_LINE>
      <DKT_LNE_NBR>1</DKT_LNE_NBR>
      <PROD_NBR>50074505</PROD_NBR>
      <TRANS_TYPE>SV</TRANS_TYPE>
      <PREV_TRANS_TYPE>SB</PREV_TRANS_TYPE>
      <TRANS_QTY>1</TRANS_QTY>
      <PREV_QTY>1</PREV_QTY>
      <SRETN_CREATE_IND>N</SRETN_CREATE_IND>
      <FULFILMENT_LOCATION>4</FULFILMENT_LOCATION>
    </TRANS_LINE>
  </TRANS_LINES>
</DOCKET>
```

---

## Test Case 2 – Reserve PDA stock receipt for web sale backorder (without FULFILMENT_LOCATION)

| Field | Value |
|---|---|
| **Test ID** | TC-V19M154-002 |
| **Environment** | MW OCI UAT (non-prod) |
| **Pre-condition** | A web sale backorder exists and PDA stock has been received against it (no fulfilment location set) |

### Steps

1. In POS, complete a PDA stock receipt against the web sale backorder where no fulfilment location
   is applicable.
2. POS creates a reservation docket payload **without** the `FULFILMENT_LOCATION` element and
   submits it to MW.
3. MW validates the payload against the OCI UAT docket schema.

### Expected result

The MW transmission is accepted.  Because `FULFILMENT_LOCATION` is optional (`minOccurs="0"` in the
schema), its absence does not cause a validation error.

---

## Test Case 3 – Schema element order within TRANS_LINE is respected

| Field | Value |
|---|---|
| **Test ID** | TC-V19M154-003 |
| **Environment** | MW OCI UAT (non-prod) |
| **Pre-condition** | Schema file `schema/mw-oci-uat-docket.xsd` is deployed |

### Steps

1. Inspect the deployed schema.
2. Confirm the element ordering in `TRANS_LINE` matches the sequence below.

### Expected element order in TRANS_LINE

1. `DKT_LNE_NBR` (required)
2. `PROD_NBR` (required)
3. `TRANS_TYPE` (required)
4. `PREV_TRANS_TYPE` (optional)
5. `TRANS_QTY` (required)
6. `PREV_QTY` (optional)
7. `SRETN_CREATE_IND` (optional)
8. `FULFILMENT_LOCATION` (optional) ← **newly added element**

### Expected result

Schema is deployed with `FULFILMENT_LOCATION` as the last optional element in the `TRANS_LINE`
sequence, consistent with the TST environment fix.
