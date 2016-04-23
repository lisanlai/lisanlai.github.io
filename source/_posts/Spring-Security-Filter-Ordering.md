---
title: Spring Security Filter Ordering
date: 2016-04-22 11:00:46
categories:
- Development
tags:
- spring security
- Java
---

> **The order that filters are defined in the chain is very important. Irrespective of which filters you are actually using, the order should be as follows:**

1. ChannelProcessingFilter, because it might need to redirect to a different protocol

2. SecurityContextPersistenceFilter, so a SecurityContext can be set up in the SecurityContextHolder at the beginning of a web request, and any changes to the SecurityContext can be copied to the HttpSession when the web request ends (ready for use with the next web request)

3. ConcurrentSessionFilter, because it uses the SecurityContextHolder functionality and needs to update theSessionRegistry to reflect ongoing requests from the principal

4. Authentication processing mechanisms - UsernamePasswordAuthenticationFilter, CasAuthenticationFilter,BasicAuthenticationFilter etc - so that the SecurityContextHolder can be modified to contain a valid Authenticationrequest token
<!-- more -->

5. The SecurityContextHolderAwareRequestFilter, if you are using it to install a Spring Security awareHttpServletRequestWrapper into your servlet container

6. The JaasApiIntegrationFilter, if a JaasAuthenticationToken is in the SecurityContextHolder this will process theFilterChain as the Subject in the JaasAuthenticationToken

7. RememberMeAuthenticationFilter, so that if no earlier authentication processing mechanism updated theSecurityContextHolder, and the request presents a cookie that enables remember-me services to take place, a suitable remembered Authentication object will be put there

8. AnonymousAuthenticationFilter, so that if no earlier authentication processing mechanism updated theSecurityContextHolder, an anonymous Authentication object will be put there

9. ExceptionTranslationFilter, to catch any Spring Security exceptions so that either an HTTP error response can be returned or an appropriate AuthenticationEntryPoint can be launched

10. FilterSecurityInterceptor, to protect web URIs and raise exceptions when access is denied
