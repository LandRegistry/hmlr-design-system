# Deploying the production environment

## Overview

The Design System is a static site which is generated by
[Metalsmith](http://www.metalsmith.io/).

When the main branch changes, [Github Actions][github-actions] will build the static site by
running `npm run build`. As long as that succeeds, the contents of the deploy
directory will then by uploaded to our application on
[GOV.UK Platform as a Service][paas] (PaaS). GOV.UK PaaS uses the open source Cloud Foundry project and exposes parts of their API.

We use nginx as a simple web server.

## Github Actions

There are two workflows used to test and deploy the Design System website.

### Testing

The test workflow is configured in [.github/workflows/test.yaml](/.github/workflows/test.yaml).

This workflow is set to run on every branch except main, where it is instead run as a dependency on the deployment workflow.

The workflow will ensure the Design System website can be built successfully, then run the linter and the tests.

### Deployment

The deployment workflow is configured in [.github/workflows/deploy.yaml](/.github/workflows/deploy.yaml).

This workflow only runs on the main branch, as a deployment to the [production environment][gh-env]. It uses a [concurrency group][gh-concurrency] to prevent multiple workflows from trying to deploy at the same time.

Before deploying, the workflow will:

- run the test workflow, passing the built site from that workflow as an artefact to be deployed
- check that the version of the site being deployed matches the current main branch, to prevent an old job being re-run and an old version of the site being deployed

We define healthchecks in the manifest file to check that the `/__canary__` path returns a 200 response, which indicates that the deployed instance is running correctly.

## Hosting on GOV.UK Platform as a Service (PaaS)

We deploy to the `govuk-design-system-origin` app.

This app exists within the `govuk-design-system` organisation and the
`design-system` space, and is deployed by the
`design-system-deploy+design-system@digital.cabinet-office.gov.uk` user, the
credentials for which can be found in BitWarden.

We use an [application manifest](/deploy/manifest.yml) to configure the app name
and the type of buildpack that we want to use.

See [PaaS documentation](https://docs.cloud.service.gov.uk/) for how to sign in and deploy apps.

### nginx

We use the [nginx buildpack][nginx-bp]. Nginx is configured in
[nginx.conf](../../deploy/nginx.conf).

## Content Delivery Network (CDN)

We use a CDN to cache the Design System assets and to terminate the SSL for the
service domain. We don't cache HTML files.

This uses the [`cdn-route` service][cdn-route] provided by PaaS, which in turn
provides a CloudFront distribution which we CNAME the service domain to.

Caching respects the expires headers that we send from nginx.

If you suspect issues with caching on the service domain, you can bypass the CDN
by visiting https://govuk-design-system-origin.cloudapps.digital/.

Authentication is enabled on this domain to prevent indexing and minimise
confusion. The username is `origin` and the password is `badidea`.

## DNS

`design-system.service.gov.uk` is a subdomain of the `service.gov.uk` zone and
is defined in [`alphagov/govuk-dns-config`][govuk-dns-config] (private repo) and
deployed by the GOV.UK 2nd Line team using [`alphagov/govuk-dns`][govuk-dns].
Support queries relating to DNS are best sent to `hostmaster@digit...`.

Most other services are delegated as their own zones so that the service team
have control over their own DNS, but we have opted to set up a subdomain instead
because:

- if delegated design-system.service.gov.uk would become the domain apex, which
  [cannot then also be a CNAME][cname].
- the Design System is maintained by GDS, and so we would opt to use govuk-dns
  to configure our DNS regardless.
- as a team of mostly Frontend developers we want to keep the amount of
  infrastructure we need to maintain to a minimum.

## History

### Use of the staticfile buildpack

We originally used the staticfile buildpack, but moved away because we found it
too restrictive. For example, it was difficult to configure caching because the
only place you could include nginx configuration was within a `location` block.

### Proxying on GOV.UK

We explored hosting the Design System on gov.uk/design-system by configuring
the Design System as a backend and route on GOV.UK's router.

This was abandoned because GOV.UK consistently does not use trailing slashes and
is [configured at the caching layer to strip them out][varnish-strip], whilst
the Design System consistently uses trailing slashes and nginx by default will
automatically redirect to add them (because that's how static sites and
directories with index files work). This caused an infinite redirect.

We considered changing the nginx configuration to match the behaviour of GOV.UK
but decided against it because:

- we would not be able to match this behaviour on other services we use, such as
  Netlify.
- it would break relative URLs.

We considered modifying the caching layer to allow the Design System to keep its
trailing-slashed URLs, but decided against it because:

- it introduced a 'special case' and additional complexity to GOV.UK's
  configuration.
- other parts of GOV.UK's infrastructure (such as their mirroring) make
  assumptions based on the lack of a trailing slash and relative URLs.

[github-actions]: https://github.com/LandRegistry/hmlr-design-system/actions
[paas]: https://www.cloud.service.gov.uk/
[nginx-bp]: https://github.com/cloudfoundry/nginx-buildpack
[govuk-dns-config]: https://github.com/alphagov/govuk-dns-config/blob/master/service.gov.uk.yaml
[govuk-dns]: https://github.com/alphagov/govuk-dns
[cname]: https://serverfault.com/questions/613829/why-cant-a-cname-record-be-used-at-the-apex-aka-root-of-a-domain
[cdn-route]: https://docs.cloud.service.gov.uk/#set-up-a-custom-domain-using-the-cdn-route-service
[varnish-strip]: https://github.com/alphagov/govuk-puppet/blob/22e8fd6ab532febd4a5df30381d3fc215f6e0153/modules/varnish/templates/default.vcl.erb#L31-L37
[gh-env]: https://docs.github.com/en/actions/deployment/targeting-different-environments/using-environments-for-deployment
[gh-concurrency]: https://docs.github.com/en/actions/using-jobs/using-concurrency
