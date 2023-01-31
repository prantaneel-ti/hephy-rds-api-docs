# EYK API v2 (Harbour Master) #

HarbourMaster is an authenticated JSON REST API that one uses to interact with EYK clusters and their associated resources.

## Request Headers ##

The following headers are required for all requests against the API:

* `Content-Type`: `application/json`
* `Accept`: `application/json`
* `Authorization`: `YOUR_API_KEY`

### Get Available Hephy RDS instance ###

* Endpoint: `/v2/hephy/available`
* HTTP Verb: `GET`
* **Only Used by harbour master while creating a cluster using EYK CLI**

A successful request to this endpoint returns the name and credentials of the RDS instance that can be used for creating a hephy database for a new cluster.

#### Request ####

`GET https://apiserver/v2/hephy/available`

#### Response ####

```json
{
  "hephy_instance_name":"hephy-database-central-instance-test-0",
  "hephy_instance_host":"hephy-database-central-instance-test-0.cqxxxxxxx.us-west-2.rds.amazonaws.com",
  "hephy_instance_port":"5432",
  "hephy_instance_user":"postgres",
  "hephy_instance_password":"Drwxxxxxxxx"
}
```

**Notes**
* These master credentials to the RDS instance are only available to harbour master and are not to be passed down to the tenant clusters.
