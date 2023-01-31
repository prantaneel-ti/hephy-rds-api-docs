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

### Create a hephy database and corresponding roles ###

* Endpoint: `/v1/organizations/ORGNAME/clusters/CLUSTERNAME/hephy/databases`
* HTTP Verb: `POST`

A successful request against this endpoint results in a JSON object representing the newly-created hephy database in a central RDS instance as well as a `201` response code. **The cluster slug is used as the db name and to create db roles.** Example:

#### Request ####

`POST https://apiserver/v1/organizations/ORGNAME/clusters/CLUSTERNAME/hephy/databases`

#### Payload ####

```json
{
  "hephy_instance_name":"hephy-database-central-instance-test-0",
  "hephy_instance_host":"hephy-database-central-instance-test-0.cqxxxxxxx.us-west-2.rds.amazonaws.com",
  "hephy_instance_port":"5432",
  "hephy_instance_user":"postgres",
  "hephy_instance_password":"Drwxxxxxxxx"
}
```

#### Response ####

```json
{
  "hephy_db_name" : "cluster-0123456789",
  "hephy_db_user" : "hephy-cluster-0123456789",
  "hephy_db_host" : "",
  "hephy_db_port" : "",
  "hephy_db_password" : "",
  "state" : "awaiting_provisioning"
}
```

#### Notes ####

* Actual provisioning of this resource is done in a background process, so it is not guaranteed to be ready when the request returns. The suggested mechanism for dtermining when the resource is ready to use is to poll the associated `GET` request for the `state` to change to `ready`. At this point, all values in the object should be populated as well.

* `hephy_db_user` is the role name of that role which has access limited to this hephy database.
