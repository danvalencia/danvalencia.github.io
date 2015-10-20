## Implementing SSO with hybris

This is the story of how I implemented Single Sign-On (SSO) functionality with hybris.  

### The Requirements
When the client requested us to implement SSO functionality with hybris there didn't provide much details besides the Identity Provider (IdP) of choice, which was [Okta](http://okta.com). The client had already implemented SSO for other applications (namely [Widen](http://widen.com), a cloud base DAM), and given that there was zero experience doing this with hybris we were somehow convinced that it was an easy feature to implement (mistake #1).

After digging around and asking some more questions, we concluded that we needed to implement the following features:
- Enable IdP initiated SSO functionality for all hybris cockpits.
- We must have feature toggling enabled for SSO, as we want to choose the environments that should have SSO. For example, local dev environments, CI environment should have SSO toggled off.
- User/group information lived in Active Directory and would be handled solely by the IT department.
- No direct access to LDAP would be provided, so we couldn't leverage the hybris LDAP extension to sync user/group information.

### SAML Primer
The first thing I had to do is familiarize myself with SAML (Security Assertion Markup Language), which is both a protocol and an XML based markup language.

The main concepts of SAML is that there's the concept of an Identity Provider (IdP), and a Service Provider (SP).

*IdP*s are the systems in charge of authenticating users. They typically sit in front of a directory service such as Active Directory or LDAP. *SP*s are the applications that are secured with the SSO solution. In this example, hybris is the SP and Okta is the IdP.

Authentication happens by doing a series of browser intermediated HTTP requests, with SAML payload called Assertions. The payload is encrypted using private/public key pair encryption on both ways. The following image depicts the flow nicely:

<img src="https://upload.wikimedia.org/wikipedia/en/thumb/0/04/Saml2-browser-sso-redirect-post.png/1280px-Saml2-browser-sso-redirect-post.png"></img>

There are a 2 types of authentication flows:
- IdP initiated authentication
- SP initiated authentication

In IdP initiated authentication the users log in to the system via an IdP webpage (typically a dashboard). Upon successful authentication the user is then redirected to the SP endpoint.

In SP initiated authentication, the user goes directly to the SP login page and enters her credentials there; she then gets redirected to the *IdP*s authentication endpoint and, if access is granted, she again gets redirected back to the SP.

As you can see there's an extra redirect in SP initiated flow; even with that extra step the user experience can be arguably better with SP initiated flows.

### Research

In order to even start to think about how to implement SSO in hybris I had to dig into how hybris does authentication. Now, the first thing I found out is that each hybris web extension implements its own authentication mechanism:

* All cockpits (i.e. not the NG Cockpits) implement authentication using Spring Security, and leveraging the same Spring Security context files, which are referenced from the *cockpit* extension under *classpath:cockpit/cockpit-spring-security.xml*.
* In turn, every cockpit typically overrides certain spring security configuration by specifying an additional configuration file, named *$COCKPIT_NAME-spring-security.xml*, in its own classpath.

* hybris storefronts also use Spring Security for authentication, although the exact implementation differs a bit from the cockpit implementation.

* HAC and HMC do not use Spring Security, so SSO needs to be implemented in a different way.

### Spring Security Primer


### Challenges


### Lessons Learned
