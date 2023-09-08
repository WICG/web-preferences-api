# [Self-Review Questionnaire: Security and Privacy](https://w3ctag.github.io/security-questionnaire/)


> 01.  What information does this feature expose, and for what purposes?
> 02.  Do features in your specification expose the minimum amount of information
       >      necessary to implement the intended functionality?

This feature exposes no new information.

> 03.  Do the features in your specification expose personal information,
       >      personally-identifiable information (PII), or information derived from
       >      either?
 
No.

> 04.  How do the features in your specification deal with sensitive information?

This API does not interact with any sensitive information.

> 05.  Do the features in your specification introduce state
       >      that persists across browsing sessions?

Yes, the API is designed to persist user preference overrides across browsing sessions.

> 06.  Do the features in your specification expose information about the
       >      underlying platform to origins?
 
No. 

> 07.  Does this specification allow an origin to send data to the underlying
       >      platform?
 
No.

> 08.  Do features in this specification enable access to device sensors?

No. 

> 09.  Do features in this specification enable new script execution/loading
       >      mechanisms?
 
No.

> 10.  Do features in this specification allow an origin to access other devices?

No.

> 11.  Do features in this specification allow an origin some measure of control over
       >      a user agent's native UI?

Other than the ability to override a users color-scheme preference for the current origin which can impact:
- scrollbars (this isn't a new capability though),
- the rendered favicon
- form controls rendering

This API provides no control over native UI. Also all 3 of the above are already author controllable.

> 12.  What temporary identifiers do the features in this specification create or
       >      expose to the web?
 
No temporary identifiers are created or exposed.

> 13.  How does this specification distinguish between behavior in first-party and
       >      third-party contexts?
 
TODO: Answer this question. Based on #8.
 
> 14.  How do the features in this specification work in the context of a browser’s
       >      Private Browsing or Incognito mode?
 
This API works the same in private browsing mode as it does in normal browsing mode.

> 15.  Does this specification have both "Security Considerations" and "Privacy
       >      Considerations" sections?

Yes (currently combined into one section)

> 16.  Do features in your specification enable origins to downgrade default
       >      security protections?

No. 

> 17.  What happens when a document that uses your feature is kept alive in BFCache
       >      (instead of getting destroyed) after navigation, and potentially gets reused
       >      on future navigations back to the document?
 
BFCache has no impact on this API.

> 18.  What happens when a document that uses your feature gets disconnected?

The override will persist until the user clears their preference or the user agent clears the preference.

Therefor when the document reloads the override will still be in place.

> 19.  What should this questionnaire have asked?

Questions above covered things well.
