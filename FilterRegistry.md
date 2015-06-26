# Introduction #

The FilterRegistry is refered as the default tapestry registry, which is inititated in the servlet filter. In a portlet it is only used to serve assets.


# Details #

Please do not add any services to the registry in the AppModule, which would not be used for asset handling.