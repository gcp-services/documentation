# Unexpected Large Invoice

Users who are new to public cloud's, such as students or independent developers, will sometimes run up a large amount of costs very quickly without fully understanding why. Other times, users might have their credentials stolen and services are brought up without the user's knowledge. While both of these cases aren't common, this guide serves to aid users in both preventative and reactive care for unexpectedly large invoices.

# Running Up a Bill

One of the most important things to understand about public clouds, including GCP, is the on-demand pricing model. Unlike a traditional VPS, public clouds are meant to scale to millions, and even billions of users around the world. While it's certainly possible to run a small operation on public clouds, special care must be taken to safe guard your usage, as the default mode of operation is "let the user scale for as much as they can allocate."

The most common occurrence of unexpectedly large invoice is users accidentally running up a bill due to mistakes in configuration or usage of GCP. This can manifest in any number of ways, such as running a CloudSQL database with a large amount of compute or storage, running multiple multi-core VM's, using a lot of bandwidth, or making many requests to a paid API, such as BigQuery or Google Maps. In all these cases, several steps can be taken to mitigate the chance of runaway billing.

## Budget Alerts

The primary mechanism for keeping watch over your billing in GCP is to [setup a budget alert](https://cloud.google.com/billing/docs/how-to/budgets). Budget alerts warn you when you have overrun a budget you define, with the most simple warning mechanism being an e-mail. Budget alerts **are not billing caps**, and will not stop you from overspending on GCP, as they are purely for informational purposes. Budget alerts also have a bit of a delay, so an alert may not fire exactly on the dollar you set, but at some point once you cross the budget threshold(s).

Budget alerts can be used to automate some action or task, such as [disabling billing should you go over a budget](https://cloud.google.com/billing/docs/how-to/notify#cap_disable_billing_to_stop_usage). It's important to note that **by disabling billing, resources on any project tied to the billing account will be deleted permanently, which includes a possibility of irreversible data loss.** Since this is generally not desireable, users may want to consider stopping services where it makes sense. Ultimately, the responsibility of what action to take is entirely up to the user.

# Stolen Credentials

Another common way accidental billing overages occur is the leaking of account credentials, which can include a user login or a service account. Special care should be taken to safe guard against unauthorized access of a user's Google identity to start up services on GCP.

## User Account

All user accounts should, at all times, enable MFA on their Google account. This can be done via the [security settings page](https://myaccount.google.com/security) on the Google account website. If possible, users should use a non-SMS based 2FA token, such as a physical hardware key or an authenticator mobile phone app such as Google Authenticator or Authy.

## Service Account

If possible, GCP service account keys should not be generated at all. In virtually all cases, a service account key does not even need to be generated, even if a user is developing and testing locally from a laptop or home/work computer.

### Remote Development

When executing services that need to access GCP remotely, users should use the Google Cloud CLI to generate [Application Default Credentials](https://cloud.google.com/sdk/gcloud/reference/auth/application-default/login). All Google Cloud client libraries will automatically use these local credentials to connect to Google Cloud, eliminating the need to maintain service account keys.

### Cloud Development

When executing services that access GCP from within GCP itself, GCP provides the equivalent of Application Default Credentials automatically via the platform that runs an application. For example, in GCE (VM's), the [default service account](https://cloud.google.com/compute/docs/access/service-accounts#default_service_account) provides an identity to any Google Cloud client libraries running on that VM, eliminating the need for service account keys. Similarly, Cloud Run offers a [service identity](https://cloud.google.com/run/docs/securing/service-identity) for managing what identity the Cloud Run service runs as, and what API's it has access to.

# Help! I Already Have a Huge Bill!

Google Cloud always offers [free billing and payment support to all users](https://cloud.google.com/billing/docs/how-to/resolve-issues), without the need to purchase a support plan. If you have already run up a large bill by accident, you are welcome to discuss the bill with Cloud Billing support. This is not a promise nor guarantee that Google Cloud will refund you in any way, however Cloud Billing can work with you to try to understand why you've received the invoice you did.

# I'm locked out of my account because it was suspended!

If you discovered your bill too late, and you're already suspended, use [this form](https://support.google.com/cloud/contact/cloud_platform_suspensions) to ask for help.

# Links

* [Setting up Budget Alerts](https://cloud.google.com/billing/docs/how-to/budgets)
* [Setting up automated actions with Budget Alerts](https://cloud.google.com/billing/docs/how-to/notify)
* [Manage Google account security settings](https://myaccount.google.com/security)
* [GCE Default Service Account](https://cloud.google.com/compute/docs/access/service-accounts#default_service_account)
* [Cloud Run service identity](https://cloud.google.com/run/docs/securing/service-identity)
* [Cloud Billing support](https://cloud.google.com/billing/docs/how-to/resolve-issues)