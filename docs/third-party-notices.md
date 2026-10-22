---
layout: default
title: Third-Party Notices
nav_order: 20
description: "Dependency and adapter boundaries for LocalizeShip 1.0.0."
permalink: /docs/third-party-notices/
---

# Third-Party Notices
{: .fs-9 }

### Know what is required, referenced and not redistributed
{: .fs-6 .text-grey-dk-000 }

LocalizeShip requires the free Unity Localization package **1.5.13**. Addressables **1.25.0** is installed with that package. Their licensing remains governed by Unity's package terms.

## Optional provider adapters

LocalizeShip includes compile-isolated modules named for I2 Localization and Pixel Crushers Dialogue System. They are verified only for clean compilation and provider-absent behaviour. LocalizeShip does not redistribute either commercial product, and no compatibility claim is made against a licensed installation.

The provider capture used in this documentation deliberately shows both optional providers absent:

![Provider discovery with optional adapters absent]({{ site.baseurl }}/assets/images/15-existing-provider.png)

*The optional I2 Localization and Dialogue System adapters are shown absent; this is not licensed-install verification.*

## Your responsibility

Review the Unity Asset Store terms and any third-party provider licence before distributing a project that uses those products. Keep provider-specific code behind its compile boundary and test it against the exact versions in your project.

## Related pages

- [Installation]({{ site.baseurl }}/docs/installation/)
- [Localization Providers]({{ site.baseurl }}/docs/providers/)
- [Limitations]({{ site.baseurl }}/docs/limitations/)
