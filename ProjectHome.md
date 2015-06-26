<h1>Tapestry 5 Portlet</h1>
<h2>Update (2010-05-05)</h2>
Fixed a problem, where all urls returned by an XMLHttpRequest were encoded with wrong Window state.

<h2>Update (2010-03-11)</h2>
Fixed a problem, where page Activation Context was not encoded into event/action links.

<h2>Preamble</h2>
The code is based on the JIRA commitments by Trun Le Xuan and Kristina B. Taylor:
<a href='https://issues.apache.org/jira/browse/TAP5-64'><a href='https://issues.apache.org/jira/browse/TAP5-64'>https://issues.apache.org/jira/browse/TAP5-64</a></a>
<br />
Up to now it is only working in Liferay Portal, because some features are based on the liferay api (explanation above).

<h2>Features</h2>
Origin Features:<br />
  * Render Requests
  * Action Requests

The following features have been added to the origin commitments:<br />
  * XHR Requests (AJAX)
  * Resource Serving (Portlet 2.0 Download Functionality)

<h2>Architecture</h2>

The class-diagram should give you a quick impression: <a href='http://tapestry5portlet.googlecode.com/files/uml_architecture.PNG'>Link</a>

<h2>Usage</h2>
<h3>Basic Configuration</h3>
<br />
The project is based on maven and eclipse wtp, so just import it.
<p>
Make sure the additional libraries are in your classpath:<br />
<ul><li>Portlet 2.0 API<br>
</li><li>Liferay Kernel (portal-kernel.jar)<br>
</p></li></ul>

Portlet Development is almost the same as developing a default tapestry application, but there are some things you have to be aware of:<br />

<p>There are <b>two</b> IoC registries for each portlet.</p>

<p>The first registry (refered as "PortletRegistry") is initiated in the <code>"ApplicationPortlet.class"</code>. This class is the first one reached in the portlet lifecycle, it delegates the different requests (action, render, resource) to their corresponding handlers.</p>

<p>The second one (refered as "FilterRegistry") is created in the Tapestry Filter, which is only used for assets, so make sure the Tapestry filter maps to <code>"/assets/*"</code>, otherwise your portlet would be accessible from outside the portal.</p>

![http://tapestry5portlet.googlecode.com/files/registries.png](http://tapestry5portlet.googlecode.com/files/registries.png)

<p>
Each registry has its own custom module for configuration:<br>
<br>
<ul><li>"Portlet" Registry - <code>PortletModule.class</code> located in <code>org.apache.tapestry5.portlet.services</code>
<blockquote>Here you should do your custom configurations e.g. add new services, configure existing ones</blockquote></li></ul>

<ul><li>"Filter" Registry - by default <code>AppModule.class</code>
<blockquote>You shouldn't add any custom services here, because as I already mentioned it is only used for serving Assets<br>
<br />
To avoid redundancy configuration I also included the <code>AppModule.class</code> in my PortletRegistry to contribute ApplicationDefaults. If you don't do so you would have for example specify APPLICATION_VERSION twice, otherwise assets are to be searched on the wrong path, because a random version number is generated, which is only known either to the FilterRegistry or the PortletRegistry.<br>
<br />If your class is not named <code>"AppModule"</code>, you will have to edit the <code>PortletUtilities.class</code> in <code>org.apache.tapestry5.portlet</code>. Just replace <code>appInitializer.addModules(AppModule.class);</code> by <code>appInitializer.addModules(YourModule.class);</code>.<br>
</p></blockquote></li></ul>

<h3>Resource Serving / Download Functionality</h3>

StreamResponses can only be returned as a result of an ActionEvent.
The id of an resource serving ActionLink must end with "resource" e.g.:
`<t:actionlink t:id="download`<b>resource</b>`">Download</t:actionlink>`
<br />
<br />
It is <b>not</b> possible to serve resources from  page activation (`onActivate()`) (see `redownloadresource` and the pagelink in the sources - Index.tml)

<h3>Portlet Services / Access PortletRequest</h3>

Inject `PortletRequestGlobals` to access the underlying `PortletRequest, RenderRequest, ActionRequest, ResourceRequest` (for example to check user roles etc.)

<h3>Application Catalog</h3>

To be able to use the application catalog, you have to define it manually:
```
configuration.add(SymbolConstants.APPLICATION_CATALOG, "context:WEB-INF/app.properties");
```

<h2>Customization / Port to your portlet-container of choice</h2>

As I mentioned before the portlet functionality only works for Liferay.

In order to make it work with other portal implementations the following parts must be customized:

<h3>Recognize XHR Request</h3>

Tapestry recognizes XHR Requests by looking into the http request headers. If the header attribute `X-Requested-With` with the value `XMLHttpRequest` is present, the request is XHR.
<br />
The portlet api does not provide direct access to the underlying http request. The Portlet specification states:
```
A portlet can access portal/portlet-container specific properties and, if available, the
headers of the HTTP client request through the following methods of the methods of the
PortletRequest interface:
• getProperty
• getProperties
• getPropertyNames

...

Depending on the underlying web-server/servlet-container and the portal/portletcontainer
implementation, client request HTTP headers may not be always available.
```

Liferay does not put the http request headers into the PortletRequest properties, so I had to rely on the liferay api, which provides access to the `HttpServletRequest`.

<br />class: `PortletRequestImpl` package: `org.apache.tapestry5.internal.portlet.services` method: `isXHR()`
```
LiferayPortletRequest liferayRequest = (LiferayPortletRequest) _request;

if (XML_HTTP_REQUEST.equals(liferayRequest.getHttpServletRequest().getHeader((REQUESTED_WITH_HEADER)))){
    	   _logger.info("REQUEST IS XHR");
    	   return true;
    	} else {
    	   return false;
    	}
```

So go find out, if your portlet-container puts the request headers into the PortletRequest properties. If it does you are lucky and you dont have to rely on your portlet-container api.

<h3>XHR Links</h3>

Currently XHR Requests and XHR Responses are handled via render urls with special liferay specific parameters (`LiferayWindowsState.EXCLUSIVE`):

see `ComponentEventLinkEncoderImplPortlet.class` in `org.apache.tapestry5.internal.portlet.services`

```
	portletURL = renderResponse.createRenderURL();
        portletURL.setPortletMode(portletRequest.getPortletMode());
 	portletURL.setWindowState(LiferayWindowState.EXCLUSIVE);
```

in your portal implementation you would rather use resource links for XHR:

```
	portletURL = renderResponse.createResourceURL();
```

<h2>TODO</h2>

The FilterRegistry loads all default Tapestry Services (`TapestryModule.class`), though only services for handling assets are needed. There should be a way to write a filter, which only loads the necessary services.

Share the portlet registry among portlets (put `PortletUtilities.class` in the portlet-container shared dir ?).