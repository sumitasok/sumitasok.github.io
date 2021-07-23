# Blog: Service checklist


## References between client and service

Here, the client is the caller, and the service is the recipient who also provides a unique functionality over this API.

We will use order and payments as an example where order-service will use payment-service to manage payments.


### Functional requirements:



1. An order service can attempt payment via payment service. Payment-service encapsulates nuances associated with the payment and finally return the status of payment.
2. The Order-service should re-request the payment service for managing the user’s payment when the previous payment method fails.
3. The order should be able to fetch the successful Payment from Payment-service.
4. The order should be able to fetch all the payment attempts from the Payment-service, including the failed.


### Architecture:

In this scenario, Payment-service should store the payment-related information. It should be able to return Payment-data filtered on the order that it tried to service.

The aforementioned is a classic example of a Service needing reference to its caller.

The client must send some unique identifiers to the service to identify the client request later.

E.g, `tenant-identifier:string`, `request-type:string` and `request-identifier:string`.



* Tenant-identifier: is used to identify the client in a multi-tenant environment. (there is an authorisation step additionally using data encryption-decryption pattern)
* Request-type: is used to identify the requester within the tenant. It gives the requester flexibility to register multiple callers under a single tenant instead of creating and managing numerous tenants.
* Request-identifier: request identifier is a string as the client is open to choose his version of ID, be it integer or UUID. The requestor should convert their ID into a string and post it to the service.

The Service should generate and return



* service-ID: an identifier unique across the deployment of the service.

After furnishing the service request, the service should return a unique identifier to the caller.

The caller can request this specific service information by calling the service with the client-request-id and service-id.

`GET /<service>/<tenantID>/<service-resource>/<client-request-type>/<client-request-id>/<service-id>`


### Entity Relationships


![alt_text](images/image1.png "image_tooltip")


The service should not restrict the caller to be having a has_one relationship with their service. It is entirely up to the client to define how the relationship should be.

The client should be able to get all the Payments associated with the order using the below URL

`GET /<service>/<tenantID>/<service-resource>/<client-request-type>/<client-request-id>`

At the client end,

It has to cache all the payment IDs, and in case they have a one to one relationship, maintain the latest service-id against the order.

Reciprocate this functionality at the Service level by implementing the below URL.

`GET /<service>/<tenantID>/<service-resource>/<client-request-type>/<client-request-id>/?**latest=true**`

So that the client can always access which is the active service data for a resource (order)

