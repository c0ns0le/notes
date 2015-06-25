##### LTPA
* In web application, when WAS authenticates a user (form-based authentication), it set two imporant cookies:
  * JSESSIONID: the value of the cookie is a string constituted of: cache ID, session ID, separator `:`, clone ID, and partition ID. JSESSION ID will include a partition ID instead of a clone ID when memory-to-memory replication in peer-to-peer mode is selected. Typically, the partition ID is a long numeric number.  
   * Reference: `WebSphere Application Server V7: Session Management` redbookt, page 15.
   * [How to find out the WAS server handling request from JSESSIONID]http://wpcertification.blogspot.com/2009/09/how-to-find-out-was-server-handling.html)
  * LtpaToken2: base64 encoded string of an encrypted concatenated string of expiration time, user identiy and signature.
    * [Reference1: Understanding LTPA](http://www-01.ibm.com/support/knowledgecenter/SS9H2Y_5.0.0/com.ibm.dp.xs.doc/understandingltpa.htm)
    * [Reference2: LTPA versions and token formats](http://www-01.ibm.com/support/knowledgecenter/SS9H2Y_5.0.0/com.ibm.dp.xs.doc/understandingltpa02.htm%23ltpaversions)
    * [Decode LTPA token](http://wrschneider.blogspot.com/2011/10/quick-and-dirty-sso-with-ltpa.html)
* WAS has an authentication cache in which is stores authentication credentials and other security stuff.  When an LTAP token is sent to WAS, it checks if it has seen this token before in the cache and if it has then it authenticates the request without validating the token. If it does not exist in the cache, then it validates the token (by decrypting and validating signature) and if it authenticates the token it adds it to the cache.
 * [Reference: WAS v8.5.5 Authentication cache settings](http://www-01.ibm.com/support/knowledgecenter/SSAW57_8.5.5/com.ibm.websphere.nd.doc/ae/usec_sec_domains_cache.html?cp=SSAW57_8.5.5)
* The authentication cache stores the following:
  * User credentials with the LTPA token as key. (the LTPA token as key is my guess based on test where I change on character of the string of an already authenticated LTPA token in the cookie sent from browser and the WAS does not accept the request and directs to login page.  If it was not taking the token as key then it would not have detected this since it will only validate token content for the first time only)
  * Username and password: The password is hashed in one-way hash that cannot be reversed and both the username and hashed password are used as key to the cache.  Whenever a username and password is used, WAS uses the combination of username and hased password as key into the cahce.  If an entry is found, then the user is considered authenticated and his credentials are retrieved from the entry.  This is to avoid accessing user registry every time username and password authentication is used.
  * My understanding: it seems that authentication cache entries can have two keys: one is the LTPA token and the other is the username and password or that these are split in two seperate caches.

* an LTPA token can be used as long as it is not expired.  LTPA token expiry is a seperate concept from http session timeout.  The same token can be used with multiple different sessions and WAS accepts http requests with the LTPA token.  In general, when the http session is expired, the web application will potentially direct the user to login page and upon a new login an new token will be generated.  However, in principle if the same token is used with the new session, it is still accepted.  I have conducted the below two tests:
  * 


### Resources
* [WAS v8.5.5 Tuning security configurations](http://www-01.ibm.com/support/knowledgecenter/SSAW57_8.5.5/com.ibm.websphere.nd.doc/ae/tsec_tune.html?cp=SSAW57_8.5.5%2F1-12-2-9-0&lang=en)