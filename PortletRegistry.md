# Introduction #

The PortletRegistry is the registry used by the portlet itself. It is initiated in the Portlet Class `org.apache.tapestry5.portlet.ApplicationPortlet`.

Add new services or configure services in the `PortletModule.class`. Use the `AppModule.class` only for configuring symbols, which should be shared between the PortletRegistry and the FilterRegistry.


# Details #