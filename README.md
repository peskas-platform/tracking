Peskas: tracking domain
================

Peskas is an advanced intelligence platform for artisanal fisheries. One
of the key features of Peskas is the ability to integrate spatial data
from boat tracks with the catch data. From the data architecture point
of view, [Peskas is divided into “data
domains”](https://blog.peskas.org/posts/domain-based-architecture/),
each domain plays a bounded role and should function largely
independently from other domains. The spatial tracking data is one of
Peskas domains and *this repository contains all code for its internal
data pipeline*.

## Pipeline description

*Pipeline input(s)*: Tracking data obtained from the Pelagic Data
Systems API.

*Pipeline product(s)*: A BigQuery dataset with clean catch data. The
dataset includes tables with *(i)* the full clean dataset.

**Stages**:

1.  Input data is retrieved on a daily basis at midnight (UTM) by a
    Google Build Service from the Pelagic Data Systems restful API.
    Daily tracks are stored in a Google Storage bucket. In addition, the
    storage bucket is checked every time in case missing days exist.
2.  Changes in the storage bucket that match the daily update prefix are
    automatically detected and trigger a Google pub/sub notification.
3.  The pub/sub message triggers a call to an API processing function.
    This API reads the data from the storage and aggregates the data.
    This data is then deposited in a Google BigQuery table. The
    aggregation function runs serverless in Google CloudRun.

## Deployment steps

**Authentication**

  - [Create a service
    account](https://cloud.google.com/iam/docs/creating-managing-service-accounts)
    for this domain:
    [`deployment/create-service-account.sh`](deployment/create-service-account.sh)
  - [Create a
    key](https://cloud.google.com/iam/docs/creating-managing-service-account-keys)
    for the service account:
    [`deployment/create-service-account-key.sh`](deployment/create-service-account-key.sh)
  - [Upload the service account
    key](https://cloud.google.com/secret-manager/docs/creating-and-accessing-secrets#secretmanager-create-secret-cli)
    to Google Secret Manager to simplify reuse:
    [`deployment/create-secret.sh`](deployment/create-secret.sh)
  - Upload the Pelagic Data Systems secret information to Google Secret
    Manager. This must be done manually. The secret information should
    be in a yaml format and have two fields. The field “SECRET” which
    contains the X-API-SECRET that will be included in the header of the
    request to the Pelagic Data Systems API, and the field “API\_KEY”
    which contains the API KEY that’s part of the path of the request.

**Ingestion**

  - [Create the
    storage](https://cloud.google.com/storage/docs/creating-buckets)
    bucket:
    [`deployment/create-storage-bucket.sh`](deployment/create-storage-bucket.sh)
  - [Grant service account
    permissions](https://cloud.google.com/iam/docs/creating-managing-service-accounts)
    to admin storage objects:
    [`deployment/grant-service-account-storage-permissions.sh`](deployment/grant-service-account-storage-permissions.sh)

**Aggregation**

  - [Create topic](https://cloud.google.com/pubsub/docs/admin) for
    message passing between the storage bucket and the aggregation API:
    [`deployment/create-pubsub-aggregation-topic.sh`](deployment/create-pubsub-aggregation-topic.sh)
  - [Enable
    notifications](https://cloud.google.com/storage/docs/gsutil/commands/notification)
    for raw data updates to the aggregation topic:
    [`deployment/create-raw-data-notification.sh`](deployment/create-raw-data-notification.sh)
  - Deploy data aggregation API using a [local
    build](https://cloud.google.com/cloud-build/docs/build-debug-locally).
    This will create the url for the service:
    [`deployment/deploy-aggregation-api.sh`](deployment/deploy-aggregation-api.sh)
  - If its the first time running pubsub push service,
    [setup](https://cloud.google.com/pubsub/docs/push) pubsub
    permissions and create and invoker account for push notifications,
    skip otherwise.
  - Grant permissions to invoke aggregation api:
    [`deployment/grant-aggregation-api-permisions.sh`](deployment/grant-aggregation-api-permisions.sh)
  - Grant permissions to use BigQuery:
    [`deployment/grant-service-account-bigquery-permissions.sh`](deployment/grant-service-account-bigquery-permissions.sh)
  - [Create pubsub
    subscription](https://cloud.google.com/pubsub/docs/admin#pubsub_create_pull_subscription-gcloud)
    to data aggregation topic:
    [`deployment/create-pubsub-aggregation-subscription.sh`](deployment/create-pubsub-aggregation-subscription.sh)

**Continuous development**

  - [Connect repository to cloud
    build](https://console.cloud.google.com/cloud-build/triggers/connect)
    (needs to be done manually)
  - Rebuild ingestion when Github code is updated:
    [`deployment/create-cd-github-ingestion-trigger.sh`](deployment/create-cd-github-ingestion-trigger.sh)
  - Rebuild aggregation when Github code is updated:
    [`deployment/create-cd-github-aggregation-trigger.sh`](deployment/create-cd-github-aggregation-trigger.sh)

**Scheduling**

  - [Schedule](https://stackoverflow.com/questions/57681367/trigger-google-cloud-build-with-google-cloud-scheduler-periodically)
    ingestion daily:
    [`deployment/schedule-ingestion-build.sh`](deployment/schedule-ingestion-build.sh)

## Support & Contributing

For general questions about the Peskas Platform, contact [Alex
Tilley](mailto:a.tilley@cgiar.org). For questions about Peskas’ code and
technical infrastructure, contact [Fernando
Cagua](mailto:f.cagua@cgiar.org).

## Authors

**Fernando Cagua**

  - Website: <http://www.cagua.co/>
  - Github: <https://github.com/efcagua/>

**Alex Tilley**

  - Email: <a.tilley@cgiar.org>

## License

The code is available under the [MIT license](LICENSE).
