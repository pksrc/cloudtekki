---
title: "Publicly trusted TLS for VMware Eventing Platform"
date: 2020-06-11T00:00:00-00:00
lastmod: 2020-12-18T00:00:00-00:00
draft: false
categories: [VEBA]
tags:
- event-driven
- cloud-native
lightgallery: true
---

{{< admonition type=info title="Info" open=true >}}
This post was originally published in Medium - [pkblah.medium.com/publicly-trusted-tls-for-vmware-eventing-platform](https://medium.com/publicly-trusted-tls-for-vmware-eventing-platform-6c6f5d0a14fb)
{{< /admonition >}}

**V**[mware Event Broker Appliance](https://flings.vmware.com/vmware-event-broker-appliance) (VEBA) continues to gain momentum and as Enterprise Customers start adopting the Appliance, we continue to broach Enterprise Features such as gauranteeing High Availability or the ability to upload/bootstrap the appliance with Internal CA signed or Public TLS certificates. While I had previously covered in part how the default self-signed TLS cert that is bound to OpenFaaS gateway can be updated through our documentation below, In this short post, I wanted to provide an end to end overview of obtaining a public certificate and binding it to the Ingress Gateway.

> [VMware Event Broker Appliance - Certificates](https://vmweventbroker.io/kb/advanced-certificates)

## Let’s Encrypt

I’m going to be using  [**Let’s Encrypt**](https://letsencrypt.org/) which is a f**ree, automated, open, non-profit Certificate Authority** that provides digital certificates to improve internet security by lowering the barrier to entry for obtaining and binding certificates. Here’s a quick view of their rise in under 3 years —

{{<image src="https://miro.medium.com/max/1788/1*dY4UkRL-4gUIhsAneLHpvw.png" caption="https://letsencrypt.org/stats/">}}

## Obtaining the SSL certificate

Let’s Encrypt provides a couple of methods that I’m familiar with to generate a publicly trusted TLS certificate.

*   One mechanism assumes that you own the website and are able to standup an endpoint to prove ownership(`_http://example.com/.well-known/acme-challenge/testfile_`)
*   The other mechanism assumes that you own the domain and are able to add a TXT record to prove ownership

For VEBA, we are going to take the domain route. To get started, you’ll need the certbot cli installed. On a mac, i was able to install the certbot using homebrew.

```
brew install certbot
```

Once certbot is succesfully installed, you can generate the certificate using the command below.

```
sudo certbot certonly --manual -d path.domain.com --preferred-challenges dns
```

After providing the affirmation to a couple of questions, certbot will request you to update the DNS to create a TXT record with `_acme-challenge.path.domain.com` with a generated value from certbot. This

{{ <image src="https://miro.medium.com/max/3780/1*I5N5vxGLxximDSW3HzaJFA.png" caption="Updating TXT record for your domain" >}}

can be done from your domain manager as shown in the screenshot. For certbot to validate the DNS TXT record under the key `_acme-challenge.path.domain.com` you’ll provide the key “`_acme-challenge.path`” for the **Host** and populate the **TXT value** with the generated string from certbot.

{{<image src="https://miro.medium.com/max/1220/1*8PiIJxAkovWo04mxVGfcgA.png" caption="Let's Encrypt results" class="imageright">}}

Verify that the DNS record has propagated using tools like the one shown below.

* [DNS Checker - DNS Check Propagation Tool](https://dnschecker.org/#TXT/_acme-challenge.path.domain.com)

Once verified, hit enter on the prompt for certbot to complete the certificate generation.

The certs are now available under the path indicated and in the pem format which we can readily use with VMware Event Broker Appliance.

{{< admonition type=info title="Info" open=true >}}
Let’s Encrypt generates certificate that are short lived (3 months) when compared to other CAs but the generation and renewal is fairly easy and can be automated
{{< /admonition >}}


## Binding Public Certificate to VEBA

### Assumption

*   Access to VMware Event Broker Appliance terminal
*   Certificates from a trusted authority pre-downloaded onto the Appliance

{{<youtube 7oMCvxvL2ns>}}

I’ve documented the steps to update the certificate without disrupting VEBA here — [https://vmweventbroker.io/kb/advanced-certificates](https://vmweventbroker.io/kb/advanced-certificates) and also made it available as a short video as shown. To provide the end to end view of how easy it is to update VEBA with new certificates, the steps are provided below.

### **Steps**

Run the commands below to update the certificates

```
cd /folder/certs/location  
CERT_NAME=eventrouter-tls #DO NOT CHANGE THIS  
KEY_FILE=<private-key-file>.pem  
CERT_FILE=<public-cert-chain>.pem#recreate the tls secret  
kubectl --kubeconfig /root/.kube/config -n vmware delete secret ${CERT_NAME}  
kubectl --kubeconfig /root/.kube/config -n vmware create secret tls ${CERT_NAME} --key ${KEY_FILE} --cert ${CERT_FILE}#reapply the config to take the new certificate  
kubectl --kubeconfig /root/.kube/config apply -f /root/config/ingressroute-gateway.yaml
```

Once you have the certs in place, the last task is to ensure your Load Balancer or Networking rules are setup (for VMC customers, you can NAT traffic from Elastic IP to VEBA IP) to ensure the traffic flows through to VEBA. Once this is confirmed, update your domain’s record to have a A record pointing to the Load Balancer’s VIP or the Elastic IP.

You should now have a VEBA endpoint that is secured with a publicly trusted TLS certificate!

## Automation and Beyond

I wanted to dive deeper and automate parts of this and It is worth noting that automating cert generation and renewal with Let’s Encrypt has been covered by [Kelsey Hightower](https://medium.com/u/9e783a6f12f6?source=post_page-----6c6f5d0a14fb--------------------------------) in 2016. For customers or individuals looking to use Let’s Encrypt and if there is sufficient interest, I’ll look to provide an automated method of generating Let’s Encrypt certificates along with certificate regeneration specific to VEBA.

Hopefully, this provides a complete overview to obtaining a public TLS certificate and binding that to VEBA. While I’ve used Let’s Encrypt here, Enterprises may want to use their existing Certificate Authorities to obtain a valid certificate and follow the steps to update the certs for VEBA.

You may recall that the [VMware Event Broker Appliance team](https://vmweventbroker.io/#team-veba) works on their spare time to bring you the capabilities of event-driven automation and an extensibility platform to our VMware SDDC customers. We strive to meet market demand and build new capabilities either through features or documentation as appropriate to ensure progress. We encourage our customers to continue providing those feedbacks on where and how we can improve our product.

Happy Eventing!
