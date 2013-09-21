resource.cfg
============

    ###########################################################################
    #
    # RESOURCE.CFG - Sample Resource File for Icinga
    #
    # You can define $USERx$ macros in this file, which can in turn be used
    # in command definitions in your host config file(s).  $USERx$ macros are
    # useful for storing sensitive information such as usernames, passwords, 
    # snmp communities, etc.
    # They are also handy for specifying the path to plugins and  event
    # handlers - if you decide to move the plugins or event handlers to
    # a different directory in the future, you can just update one or two
    # $USERx$ macros, instead of modifying a lot of command definitions.
    #
    # The CGIs will not attempt to read the contents of resource file by default,
    # so you can set restrictive permissions (600 or 660) on them. This might
    # need a change when enabling full command line resolution for config.cgi
    #
    # Icinga supports up to 256 $USERx$ macros ($USER1$ through $USER256$)
    #
    # Resource files may also be used to store configuration directives for
    # external data sources like MySQL...
    #
    ###########################################################################

    # Sets $USER1$ to be the path to the plugins
    $USER1$=@PLUGINDIR@

    # Sets $USER2$ to be the path to event handlers
    #$USER2$=@PLUGINDIR@/eventhandlers

    # Store some usernames and passwords (hidden from the CGIs)
    #$USER3$=someuser
    #$USER4$=somepassword