# AWS Technical Essentials

Based on lectures provided by AWS

## On-Premise vs Cloud

[_Additional material_](https://www.hpe.com/us/en/what-is/on-premises-vs-cloud.html)

**On-Premises computing** is a computing model when companies use their own power to establish an IT infrastructure. It is good when complete transparency on usage of resources is required or when you build an infrastructure unimplementable with means of any cloud provider or both. On the other hand having an IT infrastructure and a team to support it costs a lot and requires a lot of effort.

**Cloud computing** is a computing model when companies use services provided by some provider (e.g., AWS, GCP or Azure). Unless you really need to control the resulting infrastructure on all the possible levels you are good to go with a cloud solution. In any other case establishing your infrastructure on a cloud provider can be efficient (from the financial point of view).

## [Global Infrastructure](https://aws.amazon.com/about-aws/global-infrastructure/regions_az/)

In order to reach high availability AWS is built redundant.

**Data centers** are connected with each other into so called **Availability Zones (or AZs)** which in turn are clustered into a location-based **Region**.

Aspects to consider upon a region's selection:

- Compliance (due to some GDPR stuff or some other regulations, e.g.);
- Latency (geographical closeness);
- Pricing (some regions are cheaper than others);
- Service availability (services do not get deployed to all regions at the same time).
