---
layout: layout-pane.njk
title: Prototyping
section: Get started
theme: Setup guides
order: 1
description: This guide explains how to create prototypes using the GOV.UK Design System and GOV.UK Prototype Kit
---

{% from "govuk/components/inset-text/macro.njk" import govukInsetText %}

The GOV.UK guide explains how to create prototypes using the GOV.UK Design System and GOV.UK Prototype Kit.

## Before you start

To make prototypes use the link below to navigate to the GOV.UK setup guide.

[GOV.UK Prototyping setup guide](https://design-system.service.gov.uk/get-started/prototyping/).

[Build a basic prototype](https://govuk-prototype-kit.herokuapp.com/docs/make-first-prototype/start) using the GDS Prototype Kit.

## Using the HM Land Registry components

Using the latest prototype kit as a base, install the HM Land Registry frontend styles and templates:

```sh
npm install @hmlr/frontend@v1.1.0
```

You can now import and use the HM Land Registry components in your prototype just as you can with the GOV.UK ones.

Example code can be found in the [components](/components/) section.

{{ govukInsetText({
  classes: "app-table--constrained",
  html: 'These instructions will install an older version (v1.1.0) of the HM Land Registry components, as a bug exists when using the latest version (v1.3.0) with the latest Gov prototype kit (v13).   <a href="/get-in-touch/">Contact us if you need help using the components</a>.'
}) }}


