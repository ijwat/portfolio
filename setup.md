
### Technical Writing Project Part 2

This part focuses on the Setup of NetApp's FabricPool.


How to set up FabricPool for use with existing ONTAP system

FabricPool is a feature in NetApp's ONTAP storage operating system that allows you to tier less frequently accessed data(cold) to lower-cost object storages while keeping frequently accessed data(hot) to higher performing storages.

Configuring Cloud Tiering:
--------------------------

-   First, you need to confirm your cloud tiering license/third-party certificate. FabricPool requires a capacity-based license when attaching third-party object storage providers.

-   These certificates should be installed into ONTAP before attaching them for use with local tiers.

Considerations when Cloud Tiering:
----------------------------------

-   NOTE - Beginning with ONTAP 9.4, CA Certificates are not required. Regardless, using signed certificates from a third-party authority remains the best practice

-   Be aware of the different versions you are using because some might require specific permissions

-   FDQN - FabricPool stipulates that CA Certificates must have a fully qualified domain name as the cloud tier server with which they are connected

-   In releases prior to StorageGRID 11.3, the CA certificates use a common name that is not based on the FQDN. This might cause errors that prohibit StorageGRID from being attached to ONTAP local tiers.

-   Potential Errors - "Unable to add a cloud tier. Cannot verify the certificated provided by the object store server. The certificates might not be installed on the cluster."

-   In order to avoid these potential errors and successfully attach StorageGRID 11.2 or earlier releases as a cloud tier, you have to replace the certificates in the grid with certificates that use an accurate FQDN.

Installation:
-------------

-   Installation requires CA certificates. To install, first retrieve the CA Certificates and then install the certificates into ONTAP.

### How to Retrieve CA Certificates:

1.  Open the StorageGRID admin console > Select Configuration > Load Balancer Endpoints > Select your endpoint and click edit endpoint > copy the certificate PEM:

2.  ![](https://lh7-us.googleusercontent.com/RBKhhHQS5ncsa3c3I-d3dIPn7RfQw8TKebna2FaBmYqTbCPEleCpgEYg00hyaQnsa4WwQ46Zkdrl5KCsX0GpHXl8P_Y51itzti9JyA_QUFK9wimjDP7iotBNkPDm8_jZJ_qQyJYQwutSDHOhNNnf8X8)

3.  Retrieving the certificate when using a third-party load balancer:

4.  ![](https://lh7-us.googleusercontent.com/1iVkr21NepOCDmYCFPdk9uYNinAafp9kv30JA-HvgfOK8fYO9SZWa204dQPBnfvfzQxP2cMHMKa-MMhydItnk8HFHED-Pwl55e0OcI4QVh-aecDB4_YPZclvSs6HHsR0xRDtFzOAsbIUu0wKe4L0nto)

5.  ![](https://lh7-us.googleusercontent.com/EP3ou5FRReQ3zMzvyjhXKyVNS7vU3gwZU2xtw9gcnDNNKhQH6I3FNZm6zOAXrFQTduAQoa4pQjSM-cSF26Uy_yZYj3txDfeke8c6Rja7DtzQOtZf4-A16YGu8OKpja7qmI8Uz84Z57IydqRC5iEQRXA)

### How to Install Certificates to ONTAP:

1.  To install the root certificates to ONTAP, run the following command:

      ![](https://lh7-us.googleusercontent.com/c2iPWIFOcpxL4hzgbD_w6WXa98Sffb6RoIi_Bhd1aYXqECwJcHE1Y6Y9waFuumZNirSp7ImY6f7vDUcCrGnvrMKC9C-nOoAof3BjqmWgsxvJrnsmBonXmV1lOqMbHphnWLPMkGISiU4dIOJJ0JDR3hA)
