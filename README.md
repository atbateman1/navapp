# NavApp Infrastructure:

This repository contains reverse-proxy and Kubernetes manifest configuration. This is intended as a minimum viable product (MVP) and does not take into consideration more robust availability and security configuration that I would employ in a production environment. The considerations I would make to migrate this to production are bulleted in each section.

## NGINX Configuration:

NGINX will proxy to the directions endpoint of Openrouteservice and to the CRUD application. Since this was the only requirement listed the other endpoints are not proxied. This is a security best practice to lower attack surfaces.

Generally my thought process would be to look at existing reverse-proxy configuration and iterate it with something like I put here. For instance, if we're already hosting an application with NGINX I would expect to add the server(s) to that configuration in an API gateway style solution. That is why I have a nginx.conf with what I would expect to be some base configuration tailored to an existing envrionment, where I include a new configuration file placed in conf.d.

Beyond MVP I would consider following changes:
* Utilize Ingress - https://www.nginx.com/products/nginx-ingress-controller/
* If this were the first instance of NGINX I would ensure that only the most secure ciphers are enabled in nginx.conf
* NGINX would need to be configured to redirect to the authentication platform

## Kubernetes Configuration:

This contains a pod manifest, a service definition, and a persistent volume for OSM files. NGINX hits the service IP and port defined in navapp_service.yml.

Beyond MVP I would consider the following changes:
* Add `type: loadbalancer` for scaling
* Implement autoscaling by adding `replicas` or `minReplicas` and `maxReplicas`


## Openrouteservice

The Kubernetes pod configuration in this repository refers to a public container registry. That registry holds the Openrouteservice container. This contains the API being accessed via NGINX.

The directions API takes coordinates and gives back directions. The testing example is below.

Beyond MVP I would consider the following changes:
* Build Openrouteservice locally and push it to a registry owned by the organization
* Store OSM files in an S3 bucket (or NAS if on prem)
* Employ a tool such as API Umbrella to handle authentication via the CRUD application https://api-umbrella.readthedocs.io/en/latest/

## Testing

Testing could be performed by hitting the proxy endpoint with curl

curl --include \
     --header "Content-Type: application/json; charset=utf-8" \
     --header "Accept: application/json, application/geo+json, application/gpx+xml, img/png; charset=utf-8" \
  'https://navapp.com/api/directions/driving-car?api_key=your-api-key&start=8.681495,49.41461&end=8.687872,49.420318'
  
The CRUD application can be tested by accessing https://navapp.com/crud
