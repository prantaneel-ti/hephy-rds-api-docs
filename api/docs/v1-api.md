# EYK API v1 (Changes in Harbour Master) #

HarbourMaster is an authenticated JSON REST API that one uses to interact with EYK clusters and their associated resources.

## Request Headers ##

The following headers are required for all requests against the API:

* `Content-Type`: `application/json`
* `Accept`: `application/json`
* `Authorization`: `YOUR_API_KEY`

### Create a hephy database and corresponding roles ###

* Endpoint: `/v1/organizations/ORGNAME/clusters/CLUSTERNAME/hephy/databases`
* HTTP Verb: `POST`

A successful request against this endpoint results in a JSON object representing the newly-created hephy database in a central RDS instance as well as a `201` response code. **The cluster slug is used as the db name and to create db roles.** Example:

#### Request ####

`POST https://apiserver/v1/organizations/ORGNAME/clusters/CLUSTERNAME/hephy/databases`

#### Payload ####

```json
{}
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

* > Queries the Harbour master database to find name and credentials of the RDS instance that can be used for creating a hephy database for a new cluster.

* > Stores the mapping of the cluster and the RDS instance hosting the hephy database in the Harbour Master Database and updates the number of hephy databases that the current instance is serving.

### Get the mapping of cluster and its hephy instance ###

* Endpoint: `/v1/organizations/ORGNAME/clusters/CLUSTERNAME/hephy/databases`
* HTTP Verb: `GET`

A successful request to this endpoint returns the credentials for the hephy database along with its instance details in JSON format as well as a 200 response code.

#### Request ####

`GET https://apiserver/v1/organizations/ORGNAME/clusters/CLUSTERNAME/hephy/databases`

#### Response ####

```json
{
  "hephy_instance_name": "hephy-instance-0000001",
  "hephy_db_name": "hephy-cluster-0123456789",
  "hephy_db_user": "hephy-cluster-0123456789",
  "hephy_db_host" : "",
  "hephy_db_port" : "",
  "hephy_db_password" : "",
  "state" : "ready"
}
```
#### Notes ####

* As the provisioning process happens out-of-band, the only fields in the return object that are guaranteed to be non-empty are `hephy_db_name` and `state`. When the database is in a `ready` state, all fields should have non-empty values.

### Delete the hephy database ###

* Endpoint: `/v1/organizations/ORGNAME/clusters/CLUSTERNAME/hephy/databases`
* HTTP Verb: `DELETE`

A successful request to this endpoint returns the credentials for the hephy database along with its instance details in JSON format as well as a 200 response code.

#### Request ####

`DELETE https://apiserver/v1/organizations/ORGNAME/clusters/CLUSTERNAME/hephy/databases`

#### Response ####

```json
{
  "hephy_db_name" : "hephy-cluster-0123456789",
  "hephy_instance_name" : "hephy-instance-0000001",
  "hephy_db_user": "hephy-cluster-0123456789",
  "hephy_db_host" : "",
  "hephy_db_port" : "",
  "hephy_db_password" : "",
  "state" : "awaiting_decommissioning"
}
```

#### Notes ####

* > It uses the `GET https://apiserver/v1/organizations/ORGNAME/clusters/CLUSTERNAME/hephy/databases` to get the mapping of the cluster and the RDS instance which serves the hephy db for that cluster.

* The delete request does the actual decommissioning process in a background job. When that process is complete, subsequent requests for hephy db info will show the state as `decommissioned`.
 

### Create a RDS instance to host hephy databases ###

* Endpoint: `/v1/hephy/instances`
* HTTP Verb: `POST`

A successful request against this endpoint results in a JSON object representing the newly-created (but not provisioned) database as well as a `201` response code. In order for the request to be successful, you ***must*** provide a `hephy_instance_name`, name and a user in your payload. There are also a few other attributes that you can tweak (see NOTES below). Example:

#### Request ####

`POST https://apiserver/v1/hephy/instances`

#### Payload ####

```json
{
  "hephy_instance_name": "hephy-instance-0000001"
  "name" : "hephy-instance-00000002",
  "user" : "eyk-central",
  "engine" : "postgres",
  "engine_version" : "10",
  "storage_db" : 100,
  "instance_class" : "db.t3.medium"

}
```
#### Notes ####
* > Queries the Harbour Mater database for number of hephy databases that the current instance is serving. If it is greater than a threshold `T`, a new RDS instance is created using the HM API.
#### Response ####

```json
{
  "instance_created": "true"
  "name" : "hephy-instance-00000002",
  "host" : "",
  "port" : "",
  "user" : "eyk-central",
  "password" : "",
  "database_url" : "",
  "engine" : "postgres",
  "engine_version" : "10",
  "storage_db" : "100",
  "state" : "awaiting_provisioning"
}
```

#### Notes ####

* The `name` and `user` payload fields are required and therefore make up the minimal payload for the request. All other values are optional and have defaults.

* The supported `engine` (`engine_version`) combinations are `postgres` (`9.5`, `9.6`, `10`, `11`) and `mysql` (`5.5`, `5.6`, `5.7`, `8.0`). These fields both *must* be strings if provided, and the default combination is `postgres` (`10`).

* If specified, `storage_gb` *must* be an integer that is greater-than-or-equal-to `20`. The default is `100`.

* If specified, `instance_class` *must* be a string that represents a supported instance size for your database instance(s). Currently, the only (and the default) supported instance size is `db.t3.medium`, but we'll be expanding this shortly.

* Actual provisioning of this resource is done in a background process, so it is not guaranteed to be ready when the request returns. The suggested mechanism for determining when the resource is ready to use is to poll the associated `GET` request for the `state` to change to `ready`. At this point, all values in the object should be populated as well.

* Stores the created instance as the currently available instance.


