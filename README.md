# V19M-154
Reserving stock receipt for web sale backorders fails in nonprod env due to missing FULFILLMENT_LOCATION element in MW Schema

This repository now includes an OCI UAT MW docket schema update that allows the
`FULFILMENT_LOCATION` element inside `TRANS_LINE`.
