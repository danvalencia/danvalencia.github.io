## Implementing SSO with hybris

This is the story of how I implemented Single Sign-On (SSO) functionality with [hybris](https://www.hybris.com/en/)  

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
There's 2 things you need to setup initially for enabling Spring Security for your web application:
1. First, you need to setup your context file via a context-param. For example, the *productcockpit* extension's spring configuration is setup in the following way:
``` xml  
  <context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>
      classpath:productcockpit/productcockpit-spring-configs.xml
    </param-value>
  </context-param>
```  
2. Then, you need to declare the *springSecurityFilter*, again in web.xml file, like so:  
``` xml
  <filter>
    <filter-name>springSecurityFilterChain</filter-name>
    <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
  </filter>
  <filter-mapping>
    <filter-name>springSecurityFilterChain</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>
```

After setting up your *web.xml* file you should be able to configure Spring Security directly in the spring configuration file.

#### The most simple Spring Security configuration possible
Spring makes simple things easy, and complex tasks possible; Spring Security is no exception. In its most basic form, this is how a Spring Security configuration looks like:

``` xml
  <security:http auto-config="true">
    <security:intercept-url pattern="/admin**" access="ROLE_USER" />
  </security:http>

  <security:authentication-manager>
    <security:authentication-provider>
       <security:user-service>
  	    <security:user name="dvalencia" password="Pssword" authorities="ROLE_USER" />
      </security:user-service>
    </security:authentication-provider>
  </security:authentication-manager>
```
With these few lines of xml you get a lot for free:
* You get a login page with logout functionality.
* You get a user service where you can define users and even assign roles.

A few additional concepts to note here are:
* `<security:http>` tag: here you can define the authenticated paths, as well as paths allowed for anonymous access, such as asset paths, or login pages. You may declare have multiple `<http>` configuration.
* `<security:authentication-manager>` tag: This is where the magic happens; inside this tag is where you define an *authentication-provider*, which is the main point of extension for Spring Security. In this example we're simply declaring default *authentication-manager* and *authentication-provider*.

### Spring Security SAML
Everything's all fine and dandy with the basic Spring Security sample. It get's much more interesting (and kind of confusing to be honest) when using the [Spring Security SAML extension][http://projects.springgreat Okta + Sp.io/spring-security-saml/]. The documentation is pretty good to be fair, but there's so many concepts to understand that it can be kind of daunting.

The good thing is that there's a [great Okta + Spring Security SAML sample project](http://developer.okta.com/docs/guides/spring_security_saml.html) which is exactly what I was looking for. So in order to implement the hybris SSO integration it was just a matter of configuring the cockpits to use the same approach as the sample app, or so I thought.

### hybris security and Spring Security SAML
As I mentioned in the beginning of this post, one of the key requirements was to have the ability to turn on/off SSO by using some sort of configuration. This would mean that I would need to be able to support 2 authentication mechanisms:
* hybris OOTB authentication
* SAML authentication

The `<security:http>` and `<authentication-manager>` configurations for hybris cockpits looks like this:
``` xml
<security:http  access-denied-page="/accessDenied.zul">
    <security:session-management session-authentication-strategy-ref="fixation" />
    <security:anonymous key="cockpitAnonymous" username="anonymousUser" granted-authority="ROLE_ANONYMOUS" />
    <security:intercept-url pattern="/login.zul" access="IS_AUTHENTICATED_ANONYMOUSLY" />
    <security:intercept-url pattern="/zkau/**" access="IS_AUTHENTICATED_ANONYMOUSLY" />
    <security:intercept-url pattern="/**" access="IS_AUTHENTICATED_REMEMBERED" />
    <security:remember-me services-ref="rememberMeServices" key="cockpit" />
    <security:logout logout-success-url="/index.zul" />

    <security:form-login always-use-default-target="false" login-page="/login.zul" authentication-failure-url="/login.zul?login_error=1" />
</security:http>

<security:authentication-manager alias="authenticationManager">
    <!-- Register authentication manager for SAML provider -->
    <security:authentication-provider ref="coreAuthenticationProvider"/>
</security:authentication-manager>

<bean id="coreAuthenticationProvider" class="de.hybris.platform.cockpit.security.CockpitAuthenticationProvider">
    <property name="preAuthenticationChecks" ref="corePreAuthenticationChecks" />
    <property name="userDetailsService" ref="coreUserDetailsService" />
</bean>
```

The SAML version of the above hybris configuration looks like this:  
``` xml
<security:http entry-point-ref="samlEntryPoint" use-expressions="false">
    <security:intercept-url pattern="/zkau/**" access="IS_AUTHENTICATED_ANONYMOUSLY" />
    <security:custom-filter before="FIRST" ref="metadataGeneratorFilter"/>
    <security:custom-filter after="BASIC_AUTH_FILTER" ref="samlFilter"/>
    <security:intercept-url pattern="/**" access="IS_AUTHENTICATED_FULLY"/>
    <security:logout success-handler-ref="successLogoutHandler" />
</security:http>

<security:authentication-manager alias="authenticationManager">
    <!-- Register authentication manager for SAML provider -->
    <security:authentication-provider ref="samlAuthenticationProvider"/>
</security:authentication-manager>

<!-- SAML Authentication Provider responsible for validating of received SAML messages -->
<bean id="samlAuthenticationProvider" class="org.springframework.security.saml.SAMLAuthenticationProvider" />
```
Contrasting these 2 configurations and after testing things out more than once I came to the following conclusions:
* I needed to have a way to turn on/off `<security:http>` configurations based on a property.
* I needed to be able to have 2 `authentication-provider`s: one for hybris auth and another one for SAML auth.
* Additionally our `authentication-manager` would need to
choose the correct `authentication-provider` based on the same property.

We were already using [Flip](https://github.com/tacitknowledge/flip) as a feature toggling framework, so it made sense to use this for switching SSO on and off.

After a lot more investigation I found that one could add a reference to a [RequestMatcher](http://docs.spring.io/spring-security/site/docs/3.1.x/apidocs/org/springframework/security/web/util/RequestMatcher.html) to the `<security:http>` configuration. I could then use the RequestMatcher to enable or disable each one of the `<security:http>` configs.
I created the following implementation of a RequestMatcher:  
``` java
public class SstSecurityRequestMatcher implements RequestMatcher {
    @Resource
    private FeatureService featureService;

    private boolean negateValue;

    @Override
    public boolean matches(HttpServletRequest httpServletRequest)
    {
        boolean featureEnabled = featureService.isFeatureEnabled(Features.COCKPIT_SSO);

        if (negateValue)
        {
            featureEnabled = !featureEnabled;
        }

        return featureEnabled;
    }

    public void setNegateValue(boolean negateValue)
    {
        this.negateValue = negateValue;
    }
}
```
Then, I created two `RequestMatcher` bean definitions:  
``` xml
<!-- This one is to enable the SAML http security -->
<bean id="samlSecurityRequestMatcher" class="com.sst.core.security.SstSecurityRequestMatcher" />

<!-- This other one is a negated version of the RequestMatcher for enabling the hybris OOTB security-->
<bean id="hybrisSecurityRequestMatcher" class="com.sst.core.security.SstSecurityRequestMatcher" >
    <property name="negateValue" value="true" />
</bean>
```

With these beans in place I proceeded to configure the `request-matcher-ref` for both of my `<http:security>` configurations:  
``` xml
<!-- hybris OOTB Security Configuration -->
<security:http  request-matcher-ref="hybrisSecurityRequestMatcher" access-denied-page="/accessDenied.zul">
    <security:session-management session-authentication-strategy-ref="fixation" />
    <security:anonymous key="cockpitAnonymous" username="anonymousUser" granted-authority="ROLE_ANONYMOUS" />
    <security:intercept-url pattern="/login.zul" access="IS_AUTHENTICATED_ANONYMOUSLY" />
    <security:intercept-url pattern="/zkau/**" access="IS_AUTHENTICATED_ANONYMOUSLY" />
    <security:intercept-url pattern="/**" access="IS_AUTHENTICATED_REMEMBERED" />
    <security:remember-me services-ref="rememberMeServices" key="cockpit" />
    <security:logout logout-success-url="/index.zul" />
    <security:form-login always-use-default-target="false" login-page="/login.zul" authentication-failure-url="/login.zul?login_error=1" />
</security:http>

<!--Secured pages with SAML as entry point -->
<security:http request-matcher-ref="samlSecurityRequestMatcher" entry-point-ref="samlEntryPoint" use-expressions="false">
    <security:intercept-url pattern="/zkau/**" access="IS_AUTHENTICATED_ANONYMOUSLY" />
    <security:custom-filter before="FIRST" ref="metadataGeneratorFilter"/>
    <security:custom-filter after="BASIC_AUTH_FILTER" ref="samlFilter"/>
    <security:intercept-url pattern="/**" access="IS_AUTHENTICATED_FULLY"/>
    <security:logout success-handler-ref="successLogoutHandler" />
</security:http>
```
With these `request-matcher-ref`s and using my feature toggle framework I was able to have both auth configurations in place, but only one of them active at a time.

The next step was to figure out a way to use the same feature toggle to configure the correct `AuthenticationProvider`. I solved this by using a Spring factory bean, which would contain a reference to both the hybris and SSO `AuthenticationProvider`s. I used the same feature toggle to return the right bean:  

``` java
public class SstAuthenticationProviderFactory
{
   @Resource
   private FeatureService featureService;

   @Resource
   private AuthenticationProvider coreAuthenticationProvider;

   @Resource
   private AuthenticationProvider samlAuthenticationProvider;

   public AuthenticationProvider createInstance()
   {
       if (featureService.isFeatureEnabled(Features.COCKPIT_SSO))
       {
           return samlAuthenticationProvider;
       } else
       {
           return coreAuthenticationProvider;
       }
   }
}
```

And I declared both `AuthenticationProvider`s and the `SstAuthenticationProviderFactory` factory beans in the following way:  
``` xml
<!-- This bean acts like a single bean, and with the factory-bean and factory-method attributes
     it will return the right instance based on our feature toggle-->
<bean id="authenticationProvider" factory-bean="authenticationProviderFactory" factory-method="createInstance"/>

<!-- We declare our factory bean here, it gets wired by Spring annotations -->
<bean id="authenticationProviderFactory" class="com.sst.core.security.SstAuthenticationProviderFactory"/>

<!-- SAML Authentication Provider responsible for validating of received SAML messages -->
<bean id="samlAuthenticationProvider" class="com.sst.core.security.SstSAMLAuthenticationProvider" />

<!-- hybris OOTB Authentication Provider-->
<bean id="coreAuthenticationProvider" class="de.hybris.platform.cockpit.security.CockpitAuthenticationProvider">
    <property name="preAuthenticationChecks" ref="corePreAuthenticationChecks" />
    <property name="userDetailsService" ref="coreUserDetailsService" />
</bean>
```

With those 2 changes in place, I was able to switch between normal authentication and SAML authentication.

### Challenges


### Lessons Learned
